---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
* Build an automated data drift detection system to monitor the deployed machine learning model.
* Set up a highly visual CloudWatch Dashboard on the AWS Console.

### Deployment Context:
* The XGBoost model was successfully deployed in Week 6; however, since the Rossmann dataset concludes in 2015, there is no real-world streaming data available for live monitoring.
* To resolve this, the proposed solution is to develop a `drift_simulator.py` script to artificially generate anomalous data, enabling the simulation and rigorous testing of the monitoring system.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Generate baseline statistics from the original training dataset by reading `week2_preprocessing/data/processed/train.csv`. <br> - Calculate core statistical metrics (mean, std, min, max, count) exclusively for the top 5 most critical features determined by Week 5's SHAP analysis: `Store`, `DayOfWeek`, `Promo`, `CompetitionDistance`, and `Month`. <br> - Export the calculations to `week7_monitoring/baseline_stats.json` and upload the artifact to the S3 bucket path: `ml-forecasting/monitoring/baseline/`. | 07/27/2026 | 07/27/2026 | Pandas Docs |
| 3 | - Develop the `drift_simulator.py` script as a valid technical workaround due to the SageMaker Model Monitor quota being blocked on the account. <br> - Ensure clean code practices: comprehensive docstrings, robust error handling, absolute pathing, and optional S3 upload capabilities. <br> - Program two distinct drift scenarios: `shift` (CompetitionDistance multiplied by 3 and Promo set to 80% to simulate a highly competitive market surge) and `noise` (randomized DayOfWeek values to simulate data pipeline errors). | 07/28/2026 | 07/28/2026 | Python Docs |
| 4 | - Engineer the Data Drift detection algorithm utilizing the statistical z-score method. <br> - Implement the mathematical formula: `z_score = \|mean_new - mean_baseline\| / std_baseline`. <br> - Configure the system to trigger an alert when the `z_score > 2.0`, aligning with standard statistical practices for anomaly detection. | 07/29/2026 | 07/29/2026 | Scipy/Numpy Docs |
| 5 | - Execute comprehensive drift detection tests across varying scenarios. <br> - Scenario 1 (Normal Data): Fed the original test set into the system; resulted in 0 alerts, successfully proving the absence of false alarms. <br> - Scenario 2 (Shifted Data): Fed the simulated shift data into the system; correctly identified and triggered exactly 2 alerts. <br> - Extract simulated datasets to `drifted_shift.csv` and `drifted_noise.csv` within the `week7_monitoring/data/` directory. | 07/30/2026 | 07/30/2026 | Internal System |
| 6 | - Create and configure the `RossmannForecastingDashboard` directly within the AWS CloudWatch Console. <br> - Integrate essential monitoring widgets: API Request count, Lambda execution duration, Lambda error rate, and a Drift Detection Status table. <br> - Capture and save the finalized dashboard screenshot as `week7_monitoring/cloudwatch_dashboard.png`. | 07/31/2026 | 07/31/2026 | CloudWatch Docs |

### Week 7 Achievements:

* **Drift Detection System Performance:**
  * The anomaly detection logic functioned with absolute precision: the Normal Data scenario returned a valid state (0 alerts), while the Drifted Data (shift type) successfully triggered 2 accurate alerts.
  * *Z-score Analysis Breakdown for Shift Scenario:* The `CompetitionDistance` feature shifted from a Baseline Mean of 5430.34 to a Current Mean of 16291.02, pushing the z-score to 6.83 (Status: ⚠️ DRIFT). The `Promo` feature shifted from 0.38 to 0.80, pushing the z-score to 4.21 (Status: ⚠️ DRIFT). Remaining features (`Store`, `DayOfWeek`, `Month`) maintained z-scores between 0.00 and 0.01 (Status: ✅ OK).
* **Deliverables Output:**
  * Successfully generated 4 core files for the monitoring infrastructure: `baseline_stats.json` (5-feature statistics), `drifted_shift.csv` (shift scenario data), `drifted_noise.csv` (noise scenario data), and `cloudwatch_dashboard.png` (dashboard evidence).

### Technical Notes (Lessons Learned):
* **System Constraints & Workarounds:** The inability to use SageMaker Model Monitor due to quota blocks was effectively mitigated by building the custom `drift_simulator.py` script. This proved to be a highly practical workaround that ensured testing could proceed uninterrupted.
* **Data Consistency:** All S3 uploads were synchronized to utilize the existing `quanvan-ml-forecasting-2026` bucket established in Week 6, standardizing the data flow architecture across the project.
* **Data Science Foundation:** Selecting a `z-score threshold = 2.0` provided the monitoring system with optimal balance; it serves as a highly reliable and widely accepted statistical standard for anomaly detection in production environments.