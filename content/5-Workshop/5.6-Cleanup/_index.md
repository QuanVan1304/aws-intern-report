---
title: "Clean up"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Cleaning Up Resources After the Demo

#### Why cleanup can't wait

The **SageMaker Endpoint** (`rossmann-forecasting-endpoint`, `ml.t2.medium` instance) is billed **continuously, by the hour**, from the moment it's deployed, regardless of whether it ever receives a request. It's by far the most expensive resource in this architecture if left running.

#### The `cleanup.py` script

Deletes resources in the correct dependency order: Endpoint → Endpoint Config → Model.

Final confirmed run, after redeploying and re-validating the model from the SageMaker Pipeline:
```text
Ban chac chan muon xoa Endpoint/Model? (yes/no): yes
=== STARTING RESOURCE CLEANUP ===
✅ Deleted Endpoint: rossmann-forecasting-endpoint
✅ Deleted Endpoint Config: rossmann-config-1785134679
✅ Deleted Model: rossmann-pipeline-xgboost-1785134679
✅ Deleted Model: rossmann-pipeline-xgboost-1785125081
=== CLEANUP COMPLETE ===
```

{{% notice warning %}}
**A real gap was found and fixed here:** an earlier version of `cleanup.py` filtered models with `NameContains="rossmann-xgboost"`, which does **not** match names like `rossmann-pipeline-xgboost-*` (the substring isn't contiguous). This left one orphaned Model behind after a cleanup run, only caught later by manually cross-checking with `aws sagemaker list-models`. The filter was broadened afterward — the run above shows it correctly catching both the current and a previously-orphaned Model in the same pass.
{{% /notice %}}

#### What was **kept** vs. removed by project end

| Resource | Kept? | Reason |
|---|---|---|
| `model.tar.gz` on S3 | ✅ Kept | Small size, negligible storage cost — kept for fast redeployment later |
| API Gateway + Lambda (`rossmann-forecast-api`) | ✅ Kept | No hourly cost while idle, billed only per invocation; managed via IaC (`deploy_lambda.py`) so it can be recreated anytime |
| SageMaker Endpoint | ❌ Deleted | Billed continuously by the hour — the main source of cost |
| Extra experimental Pipeline (`Rossmann-Sales-Pipeline-V3`, from an SDK v3 trial) | ❌ Deleted | Not the official solution (see SDK v2 pinning decision); kept would just be clutter |
| The official `Rossmann-Sales-Pipeline` | ✅ Kept | Pipelines themselves don't incur hourly cost — kept as the final architecture artifact, re-runnable at any time |

Final state, confirmed via AWS CLI at project end: `list-endpoints`, `list-endpoint-configs`, and `list-models` all return empty; `list-pipelines` returns exactly one entry.

#### Cost-management practices applied in this project

- Set up an **AWS Budget alert**, since AWS billing data is delayed — relying only on the billing dashboard isn't enough for early detection.
- Ran cleanup immediately after each validation pass, instead of leaving it for later, to avoid forgetting to delete resources between test runs.
- Cross-checked cleanup results against `list-models` / `list-endpoint-configs` / `list-endpoints` via CLI rather than trusting the script's own success message alone — this is exactly how the filter gap above was caught.

{{% notice warning %}}
This was a hard rule throughout the project: **every Endpoint created for a demo or test must be deleted immediately afterward** — never left running overnight without a specific reason.
{{% /notice %}}
