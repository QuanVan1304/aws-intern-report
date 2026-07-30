---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
* Fully automate the model training and deployment lifecycle using AWS SageMaker Pipelines according to industry Best Practices.
* Successfully provision, manage, and completely decommission cloud computing resources by the end of the internship term.

### Deployment Context:
* The original goal was to automate the workflow on AWS, but the personal account was initially restricted by a SageMaker Training Job quota of 0, resulting in `ResourceLimitExceeded` errors.
* A quota increase request was submitted via the Service Quotas API/Console for the `ml.m5.large` instance. AWS approved the request, raising the quota from 0 to 1, which was the turning point that allowed the project to transition from local orchestration to running a real Pipeline infrastructure on AWS.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Resolve severe `ModuleNotFoundError`/`ImportError` issues related to `sagemaker.workflow`. <br> - Root cause analysis revealed that an unpinned `pip install sagemaker` pulled the latest release, causing internal dependency conflicts between `sagemaker` and the newly separated `sagemaker-core`. <br> - Implemented Dependency Pinning by hardcoding `sagemaker==2.257.5` in `requirements.txt` to guarantee system stability. | 07/27/2026 | 07/27/2026 | SageMaker SDK Docs |
| 3 | - Build and debug 3 core issues within the real SageMaker Pipeline. <br> - Bug 1: Fixed `ClientError: 404 HeadObject` by adding required `HyperParameters` pointing to the exact S3 path of `sourcedir.tar.gz`. <br> - Bug 2: Addressed the AWS limitation preventing `MetricDefinitions` overrides by removing the Condition step entirely, shifting model evaluation to an independent post-check (separating CI/CD). <br> - Bug 3: Eliminated pipeline clutter by configuring a **Static Pipeline** (`Rossmann-Sales-Pipeline`) using `pipeline.upsert()`, removing timestamps from the pipeline name. | 07/28/2026 | 07/28/2026 | Boto3/AWS Docs |
| 4 | - Upgrade `pipeline_definition.py` to automatically package source code (`train_xgboost_sm.py` & `feature_utils.py`) into `sourcedir.tar.gz` and upload it to S3 on every run, making the Pipeline completely self-contained. <br> - Successfully executed the Pipeline, generating Training Job ID `pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc` and a real Model Artifact on S3. <br> - Deployed the `rossmann-forecasting-endpoint` and executed the Quality Gate (`build_real_features.py`) using Store 1 historical data (2015-06-15), achieving a 4.75% MAPE (PASS). | 07/29/2026 | 07/29/2026 | AWS MLOps Architecture |
| 5 | - Set up an isolated virtual environment (`sagemaker-v3-env`) to experiment with SageMaker Python SDK v3. <br> - Successfully initialized `ModelTrainer` and Pipeline, but confirmed the `MetricDefinitions` override was still rejected by AWS, proving it is a backend service limitation, not an SDK issue. <br> - Identified a CRLF line-ending bug (`sm_train.sh: line 1: $'\r': command not found`) due to SDK v3's auto-generated wrapper script incompatibility on Windows. | 07/30/2026 | 07/30/2026 | SageMaker v3 Docs |
| 6 | - Perform comprehensive End-of-Term Housekeeping to ensure stringent cost optimization. <br> - Untracked the `sagemaker-v3-env/` directory from Git and added it to `.gitignore`. <br> - Deleted the experimental `Rossmann-Sales-Pipeline-V3` and conducted manual AWS CLI cross-verification to destroy orphan Models that bypassed the automated script due to naming convention changes. <br> - Deleted the final `InService` Endpoint, confirming zero remaining billable resources. | 07/31/2026 | 07/31/2026 | AWS CLI Docs |

### Week 8 Achievements:
* **Pipeline & MLOps Infrastructure:**
  * Flawless CI/CD Architecture: The Pipeline (CI) successfully automated the training lifecycle and produced a production-ready model artifact, while the deployment phase (CD) remained strictly controlled by an independent Quality Gate.
  * The deployed model generated from the Pipeline demonstrated exceptional accuracy against real-world historical data, yielding an error rate (MAPE) of just 4.75%, well within the project's 15.0% tolerance threshold.
* **Infrastructure Governance & Cleanup:**
  * All redundant resources and hourly-billed compute instances (Endpoints) were completely eradicated. Only the static `Rossmann-Sales-Pipeline` object was retained on AWS as architectural evidence, capable of being restarted at any time.

---

### Technical Notes & Lessons Learned:

This section synthesizes the most valuable practical engineering insights gained while building the automated infrastructure and troubleshooting AWS cloud limits:

1. **Strict Dependency Management:** 
   Executing a loose `pip install sagemaker` command allowed the environment to fetch the newest, potentially unstable releases, resulting in severe internal dependency conflicts. Pinning the exact, verified version (`sagemaker==2.257.5`) proved to be a highly pragmatic strategy, ensuring absolute stability for the production system over blindly chasing the newest tech updates.

2. **Core MLOps Mindset: Decoupling Definition from Execution:**
   A common anti-pattern is appending `datetime.now()` to a Pipeline's name, which pollutes the cloud environment with new objects upon every execution. Adopting standard practices by decoupling the "Static Pipeline Definition" (fixed name, using `upsert()`) from the "Dynamic Execution" (Execution IDs containing timestamps) kept the AWS environment exceptionally clean and manageable.

3. **Delineating AWS Service Limits vs. SDK Tools:** 
   When encountering the `"You can't override the metric definitions..."` error, cross-testing both SDK v2 and v3 clarified the technical boundary: the limitation was hardcoded into the built-in algorithm container by the AWS backend service, entirely independent of the SDK version. If an error persists across different tooling, the cloud service layer itself must be investigated.

4. **Strict Separation of CI (Training) and CD (Controlled Deployment):**
   A SageMaker Pipeline's responsibility should be restricted to training automation (CI). Deploying the model to a live Endpoint must remain a deliberately separated stage (CD) guarded by a Quality Gate. This architectural decision completely eliminates the catastrophic risk of automatically deploying an unverified, poor-quality model to a production environment.

5. **Ensuring Pipeline Self-Containment:**
   Early iterations of the Pipeline depended on an old `sourcedir.tar.gz` artifact generated by a previous manual run, creating a high risk of failure if that file was accidentally deleted. Upgrading the script to automatically compress and upload the current source code to S3 immediately before execution severed all external dependencies, transforming the Pipeline into a 100% self-contained and resilient entity.

6. **Automation Risks & The Importance of CLI Auditing:**
   Because the model naming conventions shifted across development phases (from `rossmann-xgboost-*` to `rossmann-pipeline-xgboost-*`), the automated `cleanup.py` script's filters missed several orphan resources. The critical takeaway is to never blindly trust automated cleanup scripts; routine cross-verification via the AWS CLI is mandatory to protect project budgets.

7. **Strict Environment Isolation:**
   All experimental technology trials (such as testing SDK v3) must be executed within isolated virtual environments (`sagemaker-v3-env`). Crucially, these scratchpad environments must be excluded from version control (via `.gitignore`) and thoroughly wiped from AWS immediately upon completion to prevent contamination of the official production architecture during project handover.