---
title: "Automating with SageMaker Pipeline"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

## The turning point: a quota increase actually got approved

After moving to a personal AWS account (Part 2), the SageMaker Training Job quota was still 0. A quota increase request was submitted through the Service Quotas API for the `ml.m5.large` instance type, and **it was approved**, raising the quota from 0 to 1. This was the turning point that allowed the project to move from a local substitute (`simple_orchestration.py`) to actually running a Pipeline on AWS.

## Resolving SDK dependency conflicts

Installing the SDK without pinning a version (`pip install sagemaker`) pulled in the latest release, causing `ModuleNotFoundError`/`ImportError` errors when using `sagemaker.workflow` — due to internal dependency conflicts between `sagemaker` and `sagemaker-core`. After confirming the issue was the SDK's stability (by temporarily falling back to plain boto3 and reproducing an equivalent limitation independently), the project pinned `sagemaker==2.257.5` — a version verified to work reliably with this specific pipeline.

{{% notice note %}}
This is a pragmatic choice for this project, not a judgment that SDK v2 is better than v3 overall — AWS is officially moving development toward v3 (see the v3 experiment in section 5.4.2.6 below).
{{% /notice %}}

## Building and debugging the real Pipeline — 3 real bugs

`pipeline_definition.py` defines a Pipeline with one Training step (XGBoost), using the SDK's `Estimator` + `TrainingStep` + `Pipeline` classes.

| # | Error | Root cause | Fix |
|---|---|---|---|
| 1 | `ClientError: 404 HeadObject` when the Training Job ran | The Training step was missing `HyperParameters` pointing to `sagemaker_program`/`sagemaker_submit_directory` — the container didn't know where the training code was in S3 | Automatically package the training code (`train_xgboost_sm.py` + `feature_utils.py`) into `sourcedir.tar.gz`, upload it before every run, and pass the correct S3 URI as `source_dir` on the Estimator |
| 2 | `"You can't override the metric definitions for Amazon SageMaker algorithms"` when creating the Training Job | `sagemaker-xgboost:1.7-1` is classified by AWS as a **built-in algorithm image**; in script mode the container doesn't auto-log metrics, but AWS also doesn't allow custom `MetricDefinitions` to compensate | Dropped the Condition step entirely — model quality checking became an independent post-check outside the Pipeline (`build_real_features.py`, section 5.5.1), matching the common MLOps pattern of separating CI (train) from CD (controlled deploy) |
| 3 | The Pipeline generated clutter — every `create_pipeline()` call created a new object with a timestamp in the name | The original code appended `datetime.now()` to the pipeline name | Switched to a **static Pipeline**: fixed name `Rossmann-Sales-Pipeline`, using `pipeline.upsert()`; the timestamp only appears in `execution_display_name`. Cleaned up 9 clutter pipelines generated during debugging |

## Result — the Pipeline ran successfully, for real, on AWS

```json
{
    "StepName": "Rossmann-Model-Training",
    "StepStatus": "Succeeded",
    "Metadata": {
        "TrainingJob": {
            "Arn": ".../training-job/pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc"
        }
    }
}
```
![SageMaker Pipeline Training Job Details](/images/5-Workshop/5.4-Model-training-and-pipeline/pipeline-training-job.png)

The Training Job completed and produced a real model artifact on S3:
```
s3://quanvan-ml-forecasting-2026/ml-forecasting/models/pipeline-artifacts/pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc/output/model.tar.gz
```

Confirmed the Pipeline is fully static via `list-pipelines`:
```
-----------------------------
|       ListPipelines       |
+---------------------------+
|  Rossmann-Sales-Pipeline  |
+---------------------------+
```

This is the exact model deployed and validated in Part 5.5.

{{% notice note %}}
**Why "Evaluation metrics" on the SageMaker Console is always empty:** this panel only reads from `FinalMetricDataList`, which is empty for every Training Job in this Pipeline (confirmed via CLI) — this is expected given bug #2 above, **not a sign of a poor-quality model**. Actual quality is confirmed through the CloudWatch training logs (`print(f"Val RMSE: ...")`) and the independent Quality Gate (section 5.5.1).
{{% /notice %}}

## Making the Pipeline fully self-contained

The first working version of `pipeline_definition.py` pointed `source_dir` to a fixed S3 location that was, in fact, the artifact of a separate, earlier, one-off manual Training Job run — predating this Pipeline entirely. This made the Pipeline depend on an S3 object outside its own management.

**Fixed:** `pipeline_definition.py` now has a `package_and_upload_source()` function — it automatically packages `train_xgboost_sm.py` + `feature_utils.py` into `sourcedir.tar.gz` and uploads it to S3 **on every run**, before creating the Estimator:

```python
def package_and_upload_source():
    for fname in SOURCE_FILES:
        fpath = os.path.join(SCRIPT_DIR, fname)
        if not os.path.exists(fpath):
            raise FileNotFoundError(f"Missing {fname}...")
    with tarfile.open(LOCAL_TARBALL, "w:gz") as tar:
        for fname in SOURCE_FILES:
            tar.add(os.path.join(SCRIPT_DIR, fname), arcname=fname)
    s3 = boto3.client("s3", region_name=REGION)
    s3.upload_file(LOCAL_TARBALL, BUCKET, SOURCE_S3_KEY)
    return f"s3://{BUCKET}/{SOURCE_S3_KEY}"
```
![Sourcedir Packaged on S3](/images/5-Workshop/5.4-Model-training-and-pipeline/s3-sourcedir.png)

The Pipeline is now fully self-contained — it no longer depends on any unrelated, older artifact.

## SDK v3 experiment (technical appendix, not the official solution)

Experimented with migrating to SageMaker Python SDK v3 in a separate virtual environment (`sagemaker-v3-env`), without affecting the officially running v2 Pipeline.

**Motivation:** hoping v3's `ModelTrainer.with_metric_definitions()` would solve the "empty Evaluation metrics" issue without changing the container.

| Issue | Result |
|---|---|
| Migrating `Estimator` → `ModelTrainer`, `Pipeline` → `sagemaker.mlops.workflow.pipeline.Pipeline` | Worked — successfully configured and started a Training Job |
| Custom `MetricDefinition` via `.with_metric_definitions()` | **Still rejected by AWS** with the exact same error as v2 — confirming this is a limitation at the **AWS Service backend** for built-in algorithm containers, independent of SDK version |
| Automatic code packaging via `SourceCode(source_dir=...)` on Windows | Hit `sm_train.sh: line 1: $'\r': command not found` — the wrapper script auto-generated by the SDK had CRLF line-ending issues, even though the source code itself (`train_xgboost_sm.py`, `feature_utils.py`) was confirmed clean (LF only) by direct inspection. A known limitation of SDK v3 on Windows |

**Conclusion:** Two findings confirmed: (1) the `MetricDefinitions` restriction is an AWS Service-level limit, independent of SDK version; (2) SDK v3 is not yet fully stable on Windows. Decision: **keep SDK v2 as the official solution** — verified stable, `Succeeded` end-to-end. Migrating to v3 is noted as a future direction once the SDK matures further on Windows.

## An earlier fallback, superseded

Before the quota was approved, `simple_orchestration.py` was built as a working substitute — a plain Python script chaining `subprocess` calls through Preprocessing → Training (local) → Deploy → Test, with an automated quality gate (15% MAPE threshold). It served its purpose at the time and is kept in the repository as a record of that phase, but the SageMaker Pipeline described above is the system actually used for the project's final results.

## Overall training lifecycle architecture
![Training Pipeline Architecture](/images/5-Workshop/5.4-Model-training-and-pipeline/training-pipeline-architecture.png)

```text
[Data Preprocessing] → S3 (train.csv, val.csv)
        │
        ▼
pipeline_definition.py (SDK v2, pinned to 2.257.5)
   package_and_upload_source() → auto-packaged sourcedir.tar.gz
   pipeline.upsert("Rossmann-Sales-Pipeline") + start(run-YYYYMMDD-HHMMSS)
        │
        ▼
   [Pipeline: Rossmann-Sales-Pipeline — static, single object]
   — TrainingStep: ml.m5.large (Completed: ~12 minutes)
   — Status: Succeeded
        │
        ▼
   Real Model Artifact stored on S3 (pipeline-artifacts/)
```

{{% notice tip %}}
The Pipeline only handles the Train (CI) part. Deploying the Endpoint is a separate, controlled step (CD) — it is not triggered automatically even when the Training Job succeeds, to avoid deploying a model that hasn't passed the Quality Gate (section 5.5.1).
{{% /notice %}}
