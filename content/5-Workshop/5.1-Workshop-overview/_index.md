---
title: "Workshop Overview"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

> **Project:** E-commerce Sales Forecasting System on AWS SageMaker
> **Program:** AWS First Cloud AI Journey
> **Organized by:** Amazon Web Services Viet Nam Company Limited
> **Documentation:** Detailed step-by-step deployment guide, including real logs and encountered/resolved errors during development.

---

## 1.1. Project Context

**Rossmann Store Sales** is a famous retail sales forecasting dataset on Kaggle, serving as the foundation for the forecasting problem in this project. The dataset includes:

- **1,017,209** daily sales transaction records.
- **1,115** stores across Germany.
- History from **2013-01-01** to **2015-07-31** (approximately 942 days).
- **Original Features:** `Store`, `DayOfWeek`, `Date`, `Sales`, `Customers`, `Open`, `Promo`, `StateHoliday`, `SchoolHoliday` (from `train.csv`), along with store metadata (`StoreType`, `Assortment`, `CompetitionDistance`, `Promo2`...) from `store.csv`.
## 1.2. Business Problem

| Challenge | Without Forecasting | With Machine Learning Forecasting |
|-----------|----------------|--------------------------|
| **Inventory** | Localized overstock/shortages | Optimize daily inventory levels |
| **Human Resources** | Over/under-staffed shift assignments | Right people, right peak times |
| **Marketing** | Ineffective promotions | Select optimal campaign timing (Promo increases sales by ~37% — see Part 4) |
| **Finance** | Unexpected revenue fluctuations | Proactive revenue forecasting |

## 1.3. System Architecture Diagram (4 Tiers)

```text
┌─────────────────────────────────────────────────────────────┐
│                       DATA TIER                             │
│  Kaggle Rossmann CSV → Amazon S3 (raw + processed)          │
│  Bucket: quanvan-ml-forecasting-2026 (ap-southeast-1)       │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────────┐
│                  MACHINE LEARNING TIER                        │
│  Preprocessing & Feature Engineering (22 features)            │
│  Train XGBoost (baseline) compared with PyTorch LSTM          │
│  Automation via SageMaker Pipeline (Real Training Step)       │
│  Results: RMSE 925.28, MAPE 9.92% (XGBoost, primary model)    │
└─────────────────────────┬─────────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────────┐
│                  SERVING TIER                                 │
│  SageMaker Endpoint (ml.t2.medium)                            │
│  → AWS Lambda (rossmann-forecast-api)                         │
│  → API Gateway REST API (POST /forecast, stage prod)          │
│  + Demo UI Dashboard (run locally, independent of AWS)        │
└─────────────────────────┬─────────────────────────────────────┘
                          │
┌─────────────────────────▼──────────────────────────────────────────┐  
│                     MONITORING TIER                                │
│  Data Drift Detection (Z-Score, simulated via drift_simulator.py)  │
│  CloudWatch Dashboard: RossmannForecastingDashboard                │
└────────────────────────────────────────────────────────────────────┘
```

## 1.4. Implementation Roadmap (Timeline)

| Phase | Content | Status |
|---|---|---|
| Week 1–5 | Environment setup, EDA, preprocessing, train XGBoost/LSTM baseline, SHAP, Model Registry (JSON workaround) | ✅ Completed |
| Week 6 | Deploy SageMaker Endpoint + REST API (Lambda + API Gateway) | ✅ Completed |
| Week 7 | Data Drift Detection + CloudWatch Dashboard | ✅ Completed |
| Week 8 | Automation via real SageMaker Pipeline on AWS | ✅ Completed |

> **Important infrastructure note:** From Week 6, the Deployment/Pipeline phase was migrated to a **personal AWS account** (`897355252080`, region `ap-southeast-1`) because the original account used in Weeks 1–5 (`119505195050`) was restricted with a SageMaker Training Jobs quota = 0, which could not be expanded in time for the schedule. All data and models were downloaded/retrained independently on the new account.

## 1.5. Actual Achieved Results

- **Core Model:** XGBoost Regressor, trained with `xgboost==1.7.6` (matching the serving container version — see reason in Part 5).
- **Test RMSE:** 925.28
- **Test MAPE:** 9.92%
- Comparison: PyTorch LSTM (2-layer) achieved an RMSE of 3,044.43 / MAPE of 32.79% — significantly worse, not selected as the primary model (details in Part 4).
- **Accuracy when validated against real historical data:** deviations ranging from **4.75%–5.14%** across independent validation runs (each model retraining yields slightly different results due to random seeds) — all well within the 15% quality-gate threshold set for the project.
- **Endpoint Cost:** ~$0.065/hour (`ml.t2.medium`, region `ap-southeast-1`), cleaned up immediately after every test/demo (see Part 6).
- **SageMaker Pipeline:** successfully executed on AWS after receiving quota increase approval (see Part 4.4), not just a local simulation.

## 1.6. Major Technical Challenges Overcome

This project was not simply a "smoothly follow the guide" exercise — most of the real learning value lies in debugging actual infrastructure errors. In total, the following were encountered and resolved:

- 3 errors during Endpoint deployment (region mismatch, XGBoost version mismatch, `inference.py` logic error) — Part 5
- 3 errors while building the real SageMaker Pipeline (missing HyperParameters, MetricDefinitions limits, pipeline generating junk output) — Part 4
- 1 initial quota limitation, resolved by switching accounts + simultaneously requesting a quota increase in parallel — Part 2 & Part 4
- Independent testing of SDK v3, identifying 2 additional technical limitations (metric definitions at the AWS Service tier, and line-ending issues on Windows) — recorded as directions for future development