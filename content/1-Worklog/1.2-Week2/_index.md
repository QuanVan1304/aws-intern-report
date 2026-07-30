---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
* Data preprocessing — Exploratory Data Analysis (EDA), data cleaning, data splitting, scaling, and uploading to S3.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Download the Rossmann Store Sales dataset from Kaggle, including `train.csv` (~38MB) and `store.csv`, saving them to `week2_preprocessing/data/`. <br> - Upload raw data to `s3://aws-internship-hkq-2026/ml-forecasting/data/raw/`. <br> - Upload `train.csv` using `boto3` and `store.csv` via AWS CLI. | 06/22/2026 | 06/22/2026 | Kaggle, Boto3 Docs |
| 3 | - Write and execute the EDA notebook (`eda.ipynb`). <br> - Analyze data shape, data types, and missing values. <br> - Analyze the distribution of Sales and the variance of Sales across time, day of the week, and month. <br> - Evaluate the impact of Promo and StateHoliday on sales. | 06/23/2026 | 06/23/2026 | Pandas/Seaborn Docs |
| 4 | - Write the `preprocessing.py` script (Part 1). <br> - Merge `train.csv` and `store.csv` based on the `Store` ID. <br> - Remove records of closed stores (`Open=0`) and records with zero sales (`Sales=0`). <br> - Fill missing values for the Competition and Promo2 columns. <br> - Encode categorical features (`StateHoliday`, `StoreType`, `Assortment`). | 06/24/2026 | 06/24/2026 | Pandas Docs |
| 5 | - Write the `preprocessing.py` script (Part 2). <br> - Generate calendar features: `Year`, `Month`, `Day`, `WeekOfYear`, `IsWeekend`. <br> - Perform a chronological time-series data split. <br> - Apply log-transform to the target variable and fit the StandardScaler (strictly on the train set only). | 06/25/2026 | 06/25/2026 | Scikit-Learn Docs |
| 6 | - Upload processed data to `s3://aws-internship-hkq-2026/ml-forecasting/data/processed/`. <br> - Successfully push all 4 files: `train.csv` (103MB), `val.csv` (3.7MB), `test.csv` (3.9MB), and `scaler.pkl` (538B). | 06/26/2026 | 06/26/2026 | Boto3 Docs |

### Week 2 Achievements:

* **Key EDA Findings:**
  * Sales data is right-skewed; thus, applying a log-transform to the target variable is necessary during training.
  * 172,817 records are associated with closed stores and were removed for training.
  * Clear yearly seasonality requires the integration of calendar features.
  * Sales peak on Mondays/Sundays and hit their lowest on Saturdays, making DayOfWeek a crucial feature.
  * December exhibits exceptionally high sales, making Month a vital feature.
  * Promo increases sales by ~37%, identifying it as the most important feature.
* **Data Splitting Details:**
  * Train Set: 785,727 rows (2013-01-01 → 2015-05-31).
  * Val Set: 28,423 rows (2015-06-01 → 2015-06-30).
  * Test Set: 30,188 rows (2015-07-01 → 2015-07-31).
* **S3 Storage Artifacts:**
  * Raw data: `train.csv` (38MB), `store.csv` (45KB).
  * Processed data: `train.csv` (103MB), `val.csv` (3.7MB), `test.csv` (3.9MB), `scaler.pkl` (538B).

### Technical Notes (Lessons Learned):
* AWS CLI encountered a multipart upload error for large files — this was circumvented by using `boto3.upload_file()` to push `train.csv` to S3.
* The `fillna(inplace=True)` syntax is deprecated in newer pandas versions — the team plans to refactor this code in Week 8.
* The original dataset ends on 2015-07-31, not 2015-12-31 as initially anticipated — consequently, the split boundaries were appropriately adjusted.
* The Scaler was rigorously fitted only on the training set before transforming the validation and test sets — effectively avoiding data leakage.