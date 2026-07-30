---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# FROM LOCAL NOTEBOOK TO PRACTICAL MLOPS: MASTERING AWS SAGEMAKER PIPELINES

For anyone stepping into Machine Learning, the local Jupyter Notebook is the ultimate comfort zone for data processing and model training. However, real-world enterprise requirements present a much harsher challenge: *How can we fully automate this workflow, seamlessly scale it, and reliably serve the model for production applications?*

To solve this, I decided to migrate my sales forecasting project (utilizing the Rossmann Store Sales dataset) onto the AWS cloud. Below is a comprehensive look at the system architecture and the invaluable practical lessons learned along the way.

### 1. Decoupling the System with a 3-Tier MLOps Architecture

Instead of a monolithic approach, the system is strictly decoupled into three independent tiers:

*   **Tier 1 - Data Lake & Baseline Modeling:** Amazon S3 serves as the centralized data repository. During the evaluation phase, the traditional XGBoost algorithm (MAPE of 9.92%) overwhelmingly outperformed the complex PyTorch LSTM Deep Learning model (MAPE of 32.79%). This efficiency earned XGBoost its place as the core model.
*   **Tier 2 - Automated Training (CI with SageMaker Pipelines):** Manual training was replaced by the `Rossmann-Sales-Pipeline`. Any code changes automatically trigger the pipeline: it provisions computing instances, fetches data from S3, executes the XGBoost training script, and neatly packages the Model Artifact back into S3.
*   **Tier 3 - Real-time Model Serving (CD & Serving):** To expose the SageMaker Endpoint to external applications, I implemented a Serverless REST API layer using **Amazon API Gateway** and **AWS Lambda**. The Lambda function translates HTTP requests into `invoke_endpoint` commands and returns the predicted results. With this architecture, the final error rate on production data (Quality Gate) dropped to an impressive **4.75%**.

### 3 "Painful" but Invaluable Cloud Deployment Lessons

Deploying a system to the Cloud is rarely smooth sailing. Here are three major pitfalls I overcame:

*   **The Service Quotas Barrier:** The pipeline initially froze because my AWS account’s default SageMaker Training Jobs quota was 0. Instead of blindly debugging code, checking the Quota Dashboard and submitting a limit increase request to AWS Support (for the `ml.m5.large` instance) was the crucial fix.
*   **Dependency Hell:** A `ModuleNotFoundError` once broke the entire pipeline due to SDK version mismatches. The lesson? Never install libraries loosely. Strictly pinning the version with `sagemaker==2.257.5` was the ultimate lifesaver for the project.
*   **Cost Optimization:** Unlike AWS Lambda, a SageMaker Endpoint incurs hourly charges continuously as long as it is running. Therefore, writing a `cleanup.py` script to automatically tear down the Endpoint after every test run is a mandatory skill to prevent budget leaks.

### Conclusion

Transitioning from a local notebook to a full-fledged MLOps pipeline requires a massive paradigm shift: from merely writing functional code to designing scalable infrastructure, optimizing costs, and handling system failures. The reward is a robust, automated engine fully capable of meeting stringent enterprise standards.

[*...\[The post link on AWS Study Group\]...*](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2227791551319138/)