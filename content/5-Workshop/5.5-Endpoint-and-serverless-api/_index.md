---
title: "Endpoint & Serverless API"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
{{% notice note %}}
**Important architectural note:** this section covers **two independent systems**, serving two different purposes:
- **5.5.1 (Endpoint Deployment):** the model runs for real on a **SageMaker Endpoint** (cloud), called through API Gateway + Lambda — the production system.
- **5.5.2 (Serverless API & UI):** the REST API layer that exposes that endpoint publicly, plus a fully **local** demo UI that loads `xgboost_model.pkl` directly into memory and predicts on its own — it does **not** call the SageMaker Endpoint or API Gateway. Meant for quick demos without needing AWS credentials.
{{% /notice %}}

- **[5.5.1 — Endpoint Deployment](5.5.1-endpoint-deployment/)**: packaging the model, deploying to a SageMaker Endpoint, the three real bugs fixed along the way, and validating accuracy against real historical data.
- **[5.5.2 — Serverless API & Demo UI](5.5.2-serverless-api-ui/)**: exposing the endpoint through Lambda + API Gateway, end-to-end testing, and a local demo dashboard for presentations.
