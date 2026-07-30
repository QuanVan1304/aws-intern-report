---
title: "Data Preprocessing"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## 3.1. Data Preparation (Self-contained setup)

1. Directly download the raw Rossmann Store Sales dataset (`train.csv`, `store.csv`) from Kaggle.
2. Place them into the correct local project directory structure.
3. Run `preprocessing.py` to generate the processed `train.csv`, `val.csv`, `test.csv`, and `scaler.pkl`.
4. Upload all data (raw + processed) to the `YOUR_BUCKET_NAME` bucket.

### Raw Data File Structure

```text
data/raw/
├── train.csv    — 1,017,209 rows × 9 columns (transaction history)
└── store.csv    — 1,115 rows × 10 columns (store information)
```
The original dataset from Kaggle is stored intact in the `raw/` directory:

![Raw data on S3](/images/5-Workshop/5.3-Data-preprocessing/5.png)

## 3.2. Execute Preprocessing Script

```bash
python week2_preprocessing/preprocessing.py
```

## 3.3. EDA — Key findings guiding feature engineering

| Finding | Implication |
|---|---|
| `Sales` distribution is right-skewed | Consider log-transform if training in that direction (the final project trained directly on raw `Sales` — see note in Part 5.3 regarding related errors) |
| 172,817 records have `Open = 0` or `Sales = 0` | Completely remove before training, as these are not useful forecasting signals |
| `Promo` increases average sales by ~37% | Becomes 1 of the 2 most important features according to SHAP analysis (Part 4.3) |
| December sales are exceptionally high | Strong seasonality — reasonable since the data only goes up to 2015-07-31, so this effect is observed in previous years within the train set |

## 3.4. 4-Step Process for Preprocessing & Feature Engineering

### Step 1 — Data Merging
Combine metadata from `store.csv` into `train.csv` based on the primary key `Store`.

### Step 2 — Data Cleaning
Remove 172,817 records where the store is closed (`Open = 0`) or sales are 0.

### Step 3 — Create 22 Engineered Features (Feature Engineering)

| Group | Features |
|---|---|
| **Calendar Features** | `Year`, `Month`, `Day`, `DayOfWeek`, `WeekOfYear`, `IsWeekend` |
| **Business-known (known in advance, not data leakage)** | `Promo`, `StateHoliday`, `SchoolHoliday` — these are scheduled business decisions, entirely different from `Sales` (unknown outcome) |
| **Store Features (static)** | `StoreType`, `Assortment`, `CompetitionDistance`, `Promo2` |
| **Time Lag Features** | `sales_lag_7`, `sales_lag_14`, `sales_lag_30` |
| **Rolling Mean/Standard Deviation** | `rolling_mean_7/14/30`, `rolling_std_7/14/30` |

> **Important note — clearly distinguish the 2 types of features when building input for inference:** `Promo`/`StateHoliday`/`SchoolHoliday` must use the exact values of **the day being predicted itself** (known in advance). Conversely, `sales_lag_*` and `rolling_*` must only be calculated from data **prior to** the predicted day, to avoid seeing the future (data leakage). Confusing these two types caused a 23.01% deviation in the first validation run (see Part 5.4 for details).

### Step 4 — Chronological Split

| Split | Time Period | Number of Rows |
|---|---|---|
| Train | 2013-01-01 → 2015-05-31 | 785,727 |
| Validation | 2015-06-01 → 2015-06-30 | 28,423 |
| Test | 2015-07-01 → 2015-07-31 | 30,188 |

The resulting files from the preprocessing script (`train.csv`, `val.csv`, `test.csv`, `scaler.pkl`) are pushed to the `processed/` directory, getting ready for the training phase:

![Processed data on S3](/images/5-Workshop/5.3-Data-preprocessing/3.png)

> A chronological split (not random) is mandatory for time series problems to prevent data leakage from the future into the past when creating lag/rolling features.

## 3.5. Upload to S3

```powershell
aws s3 cp week2_preprocessing/data/processed/ s3://quanvan-ml-forecasting-2026/ml-forecasting/data/processed/ --recursive
aws s3 cp week2_preprocessing/data/raw/train.csv s3://quanvan-ml-forecasting-2026/ml-forecasting/data/raw/train.csv
aws s3 cp week2_preprocessing/data/raw/store.csv s3://quanvan-ml-forecasting-2026/ml-forecasting/data/raw/store.csv
```
After executing the above commands, resources are organized in a hierarchical structure on Amazon S3. The project directory includes separate partitions for data, models, and source code:

![Project root directory](/images/5-Workshop/5.3-Data-preprocessing/1.png)

Inside the data branch, files are separated into two states: before and after processing:

![Data branch directory](/images/5-Workshop/5.3-Data-preprocessing/2.png)

> **Version consistency:** the model, scaler, and preprocessing code must always match in version — discrepancies here are a common source of hard-to-debug errors during serving later (see the XGBoost version mismatch error in Part 5.3).