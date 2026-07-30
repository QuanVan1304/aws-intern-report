---
title: "Prerequisites & Infrastructure"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## 2.1. AWS Service Quota Requirements

To execute real training jobs on AWS SageMaker, the deployment AWS account must have sufficient quotas. Many default or organizational accounts might be restricted with a **SageMaker Training Jobs quota = 0** for all instance types. This will completely block the ability to run real Training Jobs (even via SageMaker Pipelines).

> **Lesson learned:** When encountering a `ResourceLimitExceeded` error, the first step is to check the Service Quotas, not change the instance type. If the quota is 0 for the entire region on the current account, you must submit a quota increase request to AWS Support (e.g., request an increase to 1 for `ml.m5.large`) or switch to an unrestricted environment to avoid losing progress.

## 2.2. Check Service Quotas before starting

```python
import boto3

quotas = boto3.client('service-quotas', region_name='ap-southeast-1')
response = quotas.list_service_quotas(ServiceCode='sagemaker', MaxResults=100)

for q in response['Quotas']:
    if q['Value'] > 0:
        print(f"{q['Value']:>6.0f}  {q['QuotaName']}")
```

In the early stages, the Endpoint quota (`ml.t2.medium`) was sufficient, while the Training Job quota remained 0 for all instance types — this is the reason the initial model used for deployment was trained **locally** first, then later moved to real training on SageMaker when the quota was approved.

## 2.3. Create IAM User and Role

- Create IAM User `admin-user` to bootstrap initial infrastructure (create bucket, role...).
- Create a separate **IAM Role for SageMaker Execution**: `SageMaker-ExecutionRole-QuanVan`.
- In actual deployment, this Role was attached with **broad-scope managed policies** (`AmazonSageMakerFullAccess`, `AmazonS3FullAccess`, `CloudWatchFullAccess`) due to time pressure when debugging multiple consecutive deployment errors.

### Recommended IAM Permissions Policy (least-privilege) for standard deployment

For those re-deploying the project or for production environments, it is recommended to use minimal policies instead of FullAccess:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ProjectBucketOnly",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    },
    {
      "Sid": "SageMakerModelAndEndpoint",
      "Effect": "Allow",
      "Action": [
        "sagemaker:CreateModel", "sagemaker:DescribeModel", "sagemaker:DeleteModel",
        "sagemaker:CreateEndpointConfig", "sagemaker:DeleteEndpointConfig",
        "sagemaker:CreateEndpoint", "sagemaker:UpdateEndpoint",
        "sagemaker:DescribeEndpoint", "sagemaker:InvokeEndpoint", "sagemaker:DeleteEndpoint",
        "sagemaker:CreateTrainingJob", "sagemaker:DescribeTrainingJob",
        "sagemaker:CreatePipeline", "sagemaker:UpdatePipeline",
        "sagemaker:StartPipelineExecution", "sagemaker:DescribePipelineExecution"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ECRPullBuiltInContainer",
      "Effect": "Allow",
      "Action": ["ecr:GetAuthorizationToken", "ecr:BatchGetImage",
                 "ecr:GetDownloadUrlForLayer", "ecr:BatchCheckLayerAvailability"],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchLogsForSageMaker",
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents", "logs:GetLogEvents"],
      "Resource": "arn:aws:logs:*:*:log-group:/aws/sagemaker/*"
    },
    {
      "Sid": "LambdaApiGateway",
      "Effect": "Allow",
      "Action": ["lambda:*", "apigateway:*"],
      "Resource": "*"
    }
  ]
}
```

> **Note:** Using FullAccess during actual development helps avoid wasting time debugging permission errors while rushing to resolve other deployment issues (see Part 5.3) — a reasonable trade-off for rapid development phases, but it should be tightened back to least-privilege before long-term production deployment.

## 2.4. Create S3 Bucket

Dedicated bucket: `quanvan-ml-forecasting-2026` (region `ap-southeast-1`).

Directory structure:
```text
quanvan-ml-forecasting-2026/ml-forecasting/
├── data/
│   ├── raw/                     ← train.csv, store.csv downloaded from Kaggle
│   └── processed/                ← train.csv, val.csv, test.csv, scaler.pkl
├── pipeline-code/                 ← sourcedir.tar.gz packaged automatically by Pipeline (Part 4.4)
└── models/
    ├── artifacts/                 ← single deployed model (Part 5)
    └── pipeline-artifacts/         ← model generated from SageMaker Pipeline
```

## 2.5. Initialize Local Environment

```powershell
# 1. Clone project to local machine
git clone [https://github.com/YOUR-USERNAME/aws-internship-ML-forecasting.git](https://github.com/YOUR-USERNAME/aws-internship-ML-forecasting.git)
cd aws-internship-ML-forecasting

# 2. Initialize and activate Python virtual environment
python -m venv venv
.\venv\Scripts\activate

# 3. Install all dependencies
pip install -r requirements.txt

# 4. Check AWS connection configuration
python verify_setup.py
```

## 2.6. Configure `config.py`

```python
# config.py — Configure AWS account information and resources
BUCKET_NAME = "quanvan-ml-forecasting-2026"
REGION = "ap-southeast-1"
PREFIX = "ml-forecasting"

S3_RAW_DATA = f"s3://{BUCKET_NAME}/{PREFIX}/data/raw/"
S3_PROCESSED_DATA = f"s3://{BUCKET_NAME}/{PREFIX}/data/processed/"
S3_MODEL_ARTIFACTS = f"s3://{BUCKET_NAME}/{PREFIX}/models/artifacts/"
SAGEMAKER_ROLE_ARN = "arn:aws:iam::897355252080:role/SageMaker-ExecutionRole-QuanVan"
```

## 2.7. Tools and Libraries

| Tool | Version/Notes |
|---|---|
| Python | Separate Venv for each purpose (main environment, and separate `sagemaker-v3-env` for SDK v3 testing) |
| `boto3` | Used directly (instead of SageMaker SDK) for deploy/packaging operations in Week 6, to maintain full control over the process and avoid SDK compatibility issues |
| SageMaker Python SDK | Pinned `sagemaker==2.257.5` — used specifically for building Pipeline (Part 4.4), a proven stable version after encountering dependency conflicts with newer versions (`sagemaker-core`) |
| AWS CLI v2 | Configured profile for account `897355252080` |
| `xgboost` | Must match version with built-in container used during serving (local `1.7.6` ↔ container `sagemaker-xgboost:1.7-1`) — this was an actual source of error encountered, see Part 5.3 |
| `curl` / `curl.exe` | Test REST API after Endpoint is available |