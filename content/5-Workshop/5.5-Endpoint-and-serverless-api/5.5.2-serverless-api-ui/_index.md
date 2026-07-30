---
title: "Serverless API & Demo UI"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

## Building the REST API (Lambda + API Gateway)

A SageMaker Endpoint can only be invoked using AWS credentials/SDK (`sagemaker-runtime`) — it is not a public REST endpoint. To let external clients call it with a plain HTTP request, a Lambda + API Gateway layer was added.

### AWS Lambda
- Dedicated role: `Lambda-InvokeSageMaker-QuanVan`, granting only permission to invoke the SageMaker Endpoint plus basic CloudWatch Logs write access.
- Lambda code managed locally in `lambda_function.py` — receives the request from API Gateway, calls `sagemaker-runtime.invoke_endpoint()`, returns JSON.
- `deploy_lambda.py` (Infrastructure as Code) fully automates the Lambda deployment: checks whether the function already exists, updates the code if so, creates it fresh if not.

### API Gateway
- New REST API, Resource `/forecast`, Method `POST`, **Lambda Proxy integration** pointing to `rossmann-forecast-api`.
- Deployed to stage: `prod`.
- Invoke URL: `https://81nxjqyb91.execute-api.ap-southeast-1.amazonaws.com/prod/forecast`
*(Configuration diagram of the POST method on API Gateway, directly routing client requests to the Lambda function for processing)*
![API Gateway Lambda Integration](/images/5-Workshop//5.5-Endpoint-and-serverless-apit/apigateway-lambda-integration.png)
### End-to-end testing (3 layers: Lambda → boto3 → real REST API)

```text
PS ...\week6_deployment> python deploy_lambda.py
Function 'rossmann-forecast-api' da ton tai — update code...
Deploy hoan tat!

PS ...\week6_deployment> python invoke_test.py
✅ KET QUA TU ENDPOINT:
{ "predicted_sales": [ 5267.64 ] }

PS ...\week6_deployment> curl.exe -X POST https://81nxjqyb91.execute-api.ap-southeast-1.amazonaws.com/prod/forecast -H "Content-Type: application/json" -d '{...}'
{"predicted_sales": [5267.64]}
```

The result from `curl` (through the public REST API) matches the result from invoking the Endpoint directly with boto3 — confirming the full API Gateway → Lambda → SageMaker Endpoint chain works correctly.

{{% notice tip %}}
Recommended test order: local `model.predict()` → invoke the SageMaker Endpoint with boto3 → invoke Lambda directly → call through API Gateway. This isolates failures quickly, since each layer adds exactly one new point of possible failure.
{{% /notice %}}

---
## Overall Serving Architecture
![Serving Architecture](/images/5-Workshop/5.5-Endpoint-and-serverless-api/serving-architecture.png)
{{% notice tip %}}
The entire serving flow is independent of the training flow (section 5.4.2) — the SageMaker Endpoint only reads the model artifact already available on S3 and does not automatically re-trigger when the Training Pipeline finishes running (in accordance with the CI/CD separation principle mentioned in 5.4.2).
{{% /notice %}}
## Demo UI Dashboard (runs locally, no AWS dependency)

Besides the cloud REST API, the project includes a web UI for quick demos without needing AWS credentials configured — suited for presentations.

### Architecture

```
Browser (index.html + app.js + style.css)
      │  fetch POST /api/forecast
      ▼
server.py (plain Python http.server, port 8000)
   — Loads xgboost_model.pkl directly into RAM at startup
   — Reuses build_features_for_store() from week6_deployment/build_real_features.py
   — Predicts in-process, does NOT call the SageMaker Endpoint
```

### Running it

```powershell
python demo_ui/server.py
```
Visit: **`http://localhost:8000`**

### Key features

- **Select a Store + forecast date**, toggle `Promo`/`SchoolHoliday` → automatically re-calls `/api/forecast` on change.
- **What-If Simulation:** automatically computes an additional prediction with `Promo` flipped, showing the % difference — a visual illustration of the promotion's impact (matching the SHAP insight in section 5.4.1: `Promo` is one of the two most important features).
- **14-day trend chart** (Chart.js): connects real historical data with the predicted point.
- **Table of all 22 computed features** with their formulas (e.g. `rolling_mean_7` → `Mean(Sales[t-7 : t-1])`), making the model's inputs transparent rather than a black box.

### Backend mechanism (`server.py`)

```python
from build_real_features import load_full_history, build_features_for_store

MODEL = pickle.load(open(MODEL_PATH, "rb"))
DF_HISTORY = load_full_history()

class ForecastRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_POST(self):
        if self.path == '/api/forecast':
            # 1. Receive store_id, target_date, Promo/SchoolHoliday overrides from the UI
            # 2. Call build_features_for_store() — reusing logic already validated in 5.5.1
            # 3. Apply UI overrides to the Promo/SchoolHoliday features if changed
            # 4. Predict twice: once with current input, once with Promo flipped for What-If
            # 5. Fetch the last 14 days of history + the target date's actual value for the chart
            # 6. Return JSON to the frontend
```

This flow is fully consistent with the logic already validated in section 5.5.1 (`build_features_for_store`) — ensuring the demo UI doesn't use different feature-computation logic than the production system.

{{% notice note %}}
The UI header currently reads "23 Inputs" for the feature table, but there are actually 22 features — a small display mismatch that doesn't affect the underlying calculation.
{{% /notice %}}
