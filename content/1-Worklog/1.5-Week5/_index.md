---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---
### Week 5 Objectives:
* Establish Model Lifecycle Management (Model Registry) and construct the inference skeleton to prepare for production deployment.
* Perform Model Explainability analysis utilizing the SHAP library to demystify the XGBoost prediction logic.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Initialize the analysis script: `week5_registry/shap_analysis.py`. <br> - Declare the specialized SHAP TreeExplainer architecture tailored for the XGBoost model. <br> - Perform a random extraction (sample) of 1,000 rows from the training set to optimize SHAP value computation time. | 07/13/2026 | 07/13/2026 | SHAP Docs |
| 3 | - Visualize feature contributions by generating 2 primary plots: a Feature Importance bar chart and a Summary dot plot. <br> - Upload these analytical plots to the S3 cloud storage under the `outputs/evaluation/` directory. | 07/14/2026 | 07/14/2026 | SHAP, Boto3 |
| 4 | - Develop the `week5_registry/model_registry.py` script. <br> - Engineer a robust workaround due to the official SageMaker Model Registry service facing a strict quota block (limit = 0). <br> - Build a mechanism to store model metadata in JSON format locally before cloud synchronization. | 07/15/2026 | 07/15/2026 | Boto3 Docs |
| 5 | - Update and register the metadata for both models (XGBoost and LSTM) into the custom registry. <br> - Embed all performance metrics and the explicit approval status (`Status: Approved`) within the JSON files. <br> - Upload the registry JSON files to the designated S3 bucket. | 07/16/2026 | 07/16/2026 | Boto3 Docs |
| 6 | - Construct the fundamental inference framework via the `week5_registry/inference.py` skeleton. <br> - Program the 4 core functions required by SageMaker Endpoints: `model_fn` (loads the model into memory during container startup); `input_fn` (parses the JSON request into a DataFrame); `predict_fn` (executes inference and applies the inverse log transform); and `output_fn` (formats the prediction into a JSON response). | 07/17/2026 | 07/17/2026 | SageMaker Docs |

### Week 5 Achievements:

* **Model Explainability (SHAP Feature Importance):**
  * Rank 1: The highest predictive contribution belongs to `rolling_mean_14` (⭐⭐⭐⭐⭐).
  * Rank 2: The `Promo` variable (⭐⭐⭐⭐⭐) proved that promotional campaigns heavily dictate sales volume.
  * Ranks 3-5: Sequentially claimed by `rolling_mean_30` (⭐⭐⭐⭐), `DayOfWeek` (⭐⭐⭐), and `Day` (⭐⭐⭐).
  * Variables exerting the least influence (⭐) included `Promo2` and `Assortment`.
* **Model Registry Accomplishments:**
  * Created `XGBoost-Baseline.json` (RMSE: 925.28, MAPE: 9.92%, Status: Approved ✅).
  * Created `LSTM-Forecaster.json` (RMSE: 3044.43, MAPE: 32.79%, Status: Approved ✅).
* **S3 Storage Structure Update:** All SHAP evaluation plots and registry JSON files were systematically organized and securely stored in the `aws-internship-hkq-2026` bucket.

### Technical Notes & Workarounds (Lessons Learned):
* **Service Limitation Handled:** The official SageMaker Model Registry was inaccessible due to an account quota block. The team skillfully bypassed this by architecting a custom JSON-based metadata registry directly on S3, fulfilling the exact same lifecycle function.
* **Data Context Logic:** The SHAP extraction process mandated the use of training data instead of testing data. This decision was mathematically necessary because the test set only spans a single month, which does not provide enough historical depth to accurately compute lag features.
* **Strategic Deployment Decision:** Following a comprehensive evaluation, the XGBoost model was officially locked in as the primary production model and scheduled for deployment in Week 6.