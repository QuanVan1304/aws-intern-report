---
title: "Deploy Endpoint"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

## Packaging the model for SageMaker

`deploy_endpoint.py` uses **plain boto3** (not the SageMaker Python SDK) to keep full control over the packaging process and avoid SDK compatibility issues.

The script fetches the model artifact directly from a completed Training Job (`describe_training_job` → `ModelArtifacts.S3ModelArtifacts`), downloads it, and repackages it into the structure the built-in container requires:

```
model.tar.gz
├── xgboost_model.pkl
└── code/
    └── inference.py
```

Since plain boto3 is used, the `SAGEMAKER_PROGRAM` and `SAGEMAKER_SUBMIT_DIRECTORY` environment variables must be set **manually** in `create_model()`:

```python
sm.create_model(
    ModelName=model_name,
    PrimaryContainer={
        "Image": XGBOOST_IMAGE,
        "ModelDataUrl": model_url,
        "Environment": {
            "SAGEMAKER_PROGRAM": "inference.py",
            "SAGEMAKER_SUBMIT_DIRECTORY": "/opt/ml/model/code",
        },
    },
    ExecutionRoleArn=ROLE_ARN,
)
```

{{% notice note %}}
This script is reused for two different model sources: a locally-trained model (early on, when the Training Job quota was still 0) and, later, a model produced by a real SageMaker Training Job running inside the Pipeline (section 5.4.2). The packaging logic is identical either way — only the source Training Job name changes.
{{% /notice %}}

## Deploying the Endpoint with boto3

The script automatically checks whether the endpoint already exists — `update_endpoint()` if it does, `create_endpoint()` if not — so it can be re-run safely without a "name already exists" error.

- **Endpoint:** `rossmann-forecasting-endpoint`
- **Instance type:** `ml.t2.medium`

**Actual log** (deploying the model produced by the SageMaker Pipeline, job `pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc`):

```text
=== DEPLOY MODEL TU SAGEMAKER PIPELINE ===
1. Lay thong tin model tu Training Job: pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc...
   Model artifact tren S3: s3://.../pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc/output/model.tar.gz
2. Tai model.tar.gz cua Pipeline ve may...
3. Giai nen va dong goi lai kem inference.py...
4. Uploading len S3...
5. Tao SageMaker Model (tu model cua Pipeline)...
6. Tao Endpoint Config...
7. Tao moi Endpoint 'rossmann-forecasting-endpoint'...
8. Deploying... (cho 5-10 phut)
   -> Status: Creating
   -> Status: InService

✅ ENDPOINT DA SAN SANG DE SU DUNG!
```
*(Confirmation on the AWS Console: The Endpoint has been successfully created and transitioned to the InService state, ready to serve forecasting requests)*
![Endpoint In Service](/images/5-Workshop/5.5-Endpoint-and-serverless-api/endpoint-in-service.png)
## Three real bugs encountered and how they were fixed

| # | Error | Root cause | Fix |
|---|---|---|---|
| 1 | `ValidationException` on `create_model` | Container image URI hardcoded to the `us-east-1` account ID, while the endpoint ran in `ap-southeast-1` — each region has its own account ID for built-in container images | Looked up the correct registry path in the official AWS documentation (Docker Registry Paths and Example Code), used the correct account ID for the region (`121021644041`) |
| 2 | `ModelError` (HTTP 500) on `invoke_endpoint` | The built-in `sagemaker-xgboost:1.7-1` container was not backward-compatible when `pickle.load()`-ing a model trained with a newer XGBoost version (`3.2.0`) | Downgraded `xgboost` to `1.7.6` to match the container, retrained the model, repackaged and redeployed |
| 3 | Predictions returned `Infinity` | `inference.py` called `np.expm1()` on the output, but the model was trained directly on raw `Sales` values (no `log1p()`), causing an overflow | Removed the `np.expm1()` call in `predict_fn`, returning `model.predict(X)` directly |

{{% notice tip %}}
**Actual debugging workflow:** read the raw traceback directly from CloudWatch Logs (`aws logs get-log-events`), then isolate the problem by testing `model.predict()` locally before concluding whether the bug was in infrastructure or in code logic — this quickly identified bug #3 as a code-logic issue, not an infrastructure one.
*(The CloudWatch log group interface tracing all Endpoint activities, a powerful tool for debugging Inference source code)*
![CloudWatch Logs for Endpoint](/images/5-Workshop/5.5-Endpoint-and-serverless-api/endpoint-service.png)
{{% /notice %}}

## Validating the model with real historical data (Quality Gate)

Once the endpoint was free of 500 errors, `build_real_features.py` was written for a deeper accuracy check — instead of just confirming the endpoint responds (a smoke test), this step confirms the **model predicts correctly**.

**Approach:**
1. Merge `train.csv` + `val.csv` + `test.csv` into a single dataframe with full history, allowing lookups for any date.
2. Compute 22 features from a store's real history — only using `sales_lag_*`/`rolling_*` from days **before** the prediction date to avoid data leakage; `Promo`/`StateHoliday`/`SchoolHoliday` use the actual values **of the prediction date itself** (known in advance, not leakage).
3. Compare the prediction against actual sales, computing the percentage error.

**Debugged across two attempts:**

| Attempt | Approach | Error |
|---|---|---|
| 1 | Inferred `Promo`/`StateHoliday` from the nearest available date (wrong approach) | 23.01% |
| 2 | Used the actual values for **the prediction date itself** | ~5% |

**Official validation result** (model from the SageMaker Pipeline) — Store 1, 2015-06-15:

![Real Validation Results](/images/5-Workshop/5.5-Endpoint-and-serverless-api/real-feature-validation.png)

Different validation runs (each retraining gives a slightly different result due to randomness) produced errors in the **4.75%–5.14%** range — all comfortably within the quality-gate threshold.

{{% notice note %}}
Important distinction: `Promo`/`StateHoliday` **for the prediction date itself** are known business inputs (promotion calendars are planned in advance), not data leakage. Mixing this up caused the 23.01% error in the first attempt.
{{% /notice %}}

An **exit-code mechanism** (`MAPE_THRESHOLD = 15%`) was added so this script can act as an automated quality gate inside a pipeline/orchestration flow (section 5.4.2).
