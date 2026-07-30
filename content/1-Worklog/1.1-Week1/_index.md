---
title: "Week 1 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:
* Setup the AWS environment and initialize the repository.
* Successfully connect to AWS services.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Create an IAM Role via the AWS Management Console. <br> &emsp; + Setup Trusted entity: AWS Service → SageMaker Execution. <br> &emsp; + Attach policies: `AmazonSageMakerFullAccess`, `AmazonS3FullAccess`, `CloudWatchFullAccess`. <br> - Create an S3 bucket via the AWS Management Console to store resources. | 15/06/2026 | 15/06/2026 | AWS Documentation |
| 3 | - Install AWS CLI v2. <br> - Configure SSO authentication (using the `aws login` command). <br> - Create a Python 3.11 virtual environment (venv) to prepare the programming environment. | 16/06/2026 | 16/06/2026 | AWS Documentation |
| 4 | - Initialize a private GitHub repository named `ml-forecasting-aws`. <br> - Build the folder structure by week: from `week1_setup/` to `week8_pipeline/`. <br> - Configure the `.gitignore` file (ensuring the exclusion of `.env`, `*.csv`, `__pycache__/`, etc.). <br> - Create a `requirements.txt` file with all necessary dependencies. | 17/06/2026 | 17/06/2026 | GitHub Docs |
| 5 | - Write the `config.py` script. <br> &emsp; + Use the `python-dotenv` library to load secure credentials from the `.env` file. <br> &emsp; + Define S3 paths, ARNs, and common constants used throughout the project. | 18/06/2026 | 18/06/2026 | Boto3 Docs |
| 6 | - Write the `verify_setup.py` script to automatically verify the environment. <br> - Program 5 core verification checks: <br> &emsp; + Check 1: AWS credentials (STS). <br> &emsp; + Check 2: S3 bucket accessibility. <br> &emsp; + Check 3: IAM Role existence. <br> &emsp; + Check 4: SageMaker API reachability. <br> &emsp; + Check 5: Region correctness. | 19/06/2026 | 19/06/2026 | Boto3 Docs |

### Week 1 Achievements:

* **IAM Role:** The role `SageMaker-ExecutionRole-hkq` was created successfully.
* **S3 Bucket:** The bucket `aws-internship-hkq-2026` was created and confirmed to be accessible.
* **GitHub Repo:** The `ml-forecasting-aws` repository was initialized successfully.
* **Python Environment:** The Python 3.11 virtual environment (venv) was activated.
* **Configuration Management:** Completed the `config.py` script with full definitions for S3 paths, ARNs, and constants.
* **Environment Verification:** The `verify_setup.py` script ran successfully, passing all 5/5 tests. Detailed output information:
  * *AWS Account:* 119505195050.
  * *User/Role:* arn:aws:iam::119505195050:root.
  * *S3 Bucket:* s3://aws-internship-hkq-2026 — accessible.
  * *IAM Role:* SageMaker-ExecutionRole-hkq — exists.
  * *SageMaker API:* OK — region ap-southeast-1.

### Technical Notes (Lessons Learned):
* **SSO Authentication:** Because the company account uses SSO, the login session expires periodically. The solution is to rerun the `aws login` command whenever the session expires.
* **SageMaker SDK Changes:** Discovered that the `sagemaker` SDK 3.x library has completely changed its core structure. To resolve this, the team decided to directly use `boto3.client("sagemaker")` instead of `sagemaker.Session()` to avoid unexpected errors.
* **Python Version Compatibility:** Noted an error where Python 3.13 is incompatible with the `numpy` and `sagemaker` libraries. This issue was thoroughly resolved by strictly forcing the use of a Python 3.11 venv environment.