---
title: "Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

The **E-commerce Sales Forecasting on AWS SageMaker** project was successfully implemented and completed by the team (including Huỳnh Kim Quý, Văn Thái Quân, Nguyễn Ngọc Sáng) within **08 weeks**. 

On this page, the team presents a detailed overview of the worklog, the core achievements, and how the team flexibly handled resource limits (quotas) on the AWS Cloud.

---

## 1. 8-Week Progress Summary

**Week 1:** [AWS environment setup (Account initialization, S3 Bucket creation, and IAM Role configuration)](1.1-week1/)

**Week 2:** [Exploratory Data Analysis (EDA) and Data Preprocessing](1.2-week2/)

**Week 3:** [Training the baseline model (XGBoost Baseline) and building the LSTM skeleton](1.3-week3/)

**Week 4:** [Training the PyTorch LSTM model and evaluating performance](1.4-week4/)

**Week 5:** [Setting up the Model Registry and analyzing model interpretability with SHAP](1.5-week5/)

**Week 6:** [Model Deployment to SageMaker Endpoint and building Serverless REST API](1.6-week6/)

**Week 7:** [Configuring Monitoring, simulating Drift Detection, and creating CloudWatch Dashboard](1.7-week7/)

**Week 8:** [Automating workflow with SageMaker Pipeline (IaC) and code refactoring](1.8-week8/)

---

## 2. Environment Management & Workarounds

During the internship, the team flexibly utilized **2 AWS accounts** to ensure the project schedule was not interrupted by service quotas:

*   **Team Account (Weeks 1–5):** `119505195050` – Used for storing raw/processed data in the S3 Bucket `aws-internship-hkq-2026`.
*   **Personal Account (Weeks 6–8):** `897355252080` – Used to deploy the SageMaker Endpoint and Pipeline because the team account's quota was blocked (S3 Bucket: `quanvan-ml-forecasting-2026`).

**Technical barriers and how the team overcame them:**
*   **SageMaker Training Jobs/Pipelines (Quota = 0):** Shifted to local training, logged metrics via boto3, and built an automated local orchestration script.
*   **SageMaker Model Registry (Quota = 0):** Designed a workaround by saving metadata as JSON directly to S3.
*   **SDK & CLI Errors:** Instead of using the broken SageMaker SDK 3.x, the team directly utilized `boto3.client()` for deep system control.

---

## 3. Summary of Key Achievements

*   **Model Performance:** XGBoost achieved a Test RMSE of **925.28** and a Test MAPE of **9.92%**, outperforming LSTM (MAPE 32.79%) to become the Production model.
*   **Serverless API System:** Successfully deployed a public REST API with a response time of ~1.1s. When validated with real historical data, the forecast deviation was only **5.14%** (well within the safe threshold of 15%).
*   **Proactive Monitoring:** The Drift Detection script worked perfectly: it accurately generated 2 alerts when the data drifted and 0 false alarms with the original data.
*   **Integrated Data Flow:** Successfully built an End-to-End architecture from Data Preprocessing ➔ Training ➔ Deployment ➔ Serverless API, with a total execution time of only **~587 seconds**.