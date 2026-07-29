---
title: "Data Preprocessing"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Part 3 — Data Preprocessing & Uploading Data to Amazon S3

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

## 3.2. Execute Preprocessing Script[cite: 4]

```bash
python week2_preprocessing/preprocessing.py
```

## 3.3. EDA — Key findings guiding feature engineering[cite: 4]

| Finding[cite: 4] | Implication[cite: 4] |
|---|---|
| `Sales` distribution is right-skewed[cite: 4] | Consider log-transform if training in that direction (the final project trained directly on raw `Sales` — see note in Part 5.3 regarding related errors)[cite: 4] |
| 172,817 records have `Open = 0` or `Sales = 0`[cite: 4] | Completely remove before training, as these are not useful forecasting signals[cite: 4] |
| `Promo` increases average sales by ~37%[cite: 4] | Becomes 1 of the 2 most important features according to SHAP analysis (Part 4.3)[cite: 4] |
| December sales are exceptionally high[cite: 4] | Strong seasonality — reasonable since the data only goes up to 2015-07-31, so this effect is observed in previous years within the train set[cite: 4] |

## 3.4. 4-Step Process for Preprocessing & Feature Engineering[cite: 4]

### Step 1 — Data Merging[cite: 4]
Combine metadata from `store.csv` into `train.csv` based on the primary key `Store`.[cite: 4]

### Step 2 — Data Cleaning[cite: 4]
Remove 172,817 records where the store is closed (`Open = 0`) or sales are 0.[cite: 4]

### Step 3 — Create 22 Engineered Features (Feature Engineering)[cite: 4]

| Group[cite: 4] | Features[cite: 4] |
|---|---|
| **Calendar Features**[cite: 4] | `Year`, `Month`, `Day`, `DayOfWeek`, `WeekOfYear`, `IsWeekend`[cite: 4] |
| **Business-known (known in advance, not data leakage)**[cite: 4] | `Promo`, `StateHoliday`, `SchoolHoliday` — these are scheduled business decisions, entirely different from `Sales` (unknown outcome)[cite: 4] |
| **Store Features (static)**[cite: 4] | `StoreType`, `Assortment`, `CompetitionDistance`, `Promo2`[cite: 4] |
| **Time Lag Features**[cite: 4] | `sales_lag_7`, `sales_lag_14`, `sales_lag_30`[cite: 4] |
| **Rolling Mean/Standard Deviation**[cite: 4] | `rolling_mean_7/14/30`, `rolling_std_7/14/30`[cite: 4] |

> **Important note — clearly distinguish the 2 types of features when building input for inference:** `Promo`/`StateHoliday`/`SchoolHoliday` must use the exact values of **the day being predicted itself** (known in advance).[cite: 4] Conversely, `sales_lag_*` and `rolling_*` must only be calculated from data **prior to** the predicted day, to avoid seeing the future (data leakage).[cite: 4] Confusing these two types caused a 23.01% deviation in the first validation run (see Part 5.4 for details).[cite: 4]

### Step 4 — Chronological Split[cite: 4]

| Split[cite: 4] | Time Period[cite: 4] | Number of Rows[cite: 4] |
|---|---|---|
| Train[cite: 4] | 2013-01-01 → 2015-05-31[cite: 4] | 785,727[cite: 4] |
| Validation[cite: 4] | 2015-06-01 → 2015-06-30[cite: 4] | 28,423[cite: 4] |
| Test[cite: 4] | 2015-07-01 → 2015-07-31[cite: 4] | 30,188[cite: 4] |

> A chronological split (not random) is mandatory for time series problems to prevent data leakage from the future into the past when creating lag/rolling features.[cite: 4]

## 3.5. Upload to S3[cite: 4]

```powershell
aws s3 cp week2_preprocessing/data/processed/ s3://quanvan-ml-forecasting-2026/ml-forecasting/data/processed/ --recursive
aws s3 cp week2_preprocessing/data/raw/train.csv s3://quanvan-ml-forecasting-2026/ml-forecasting/data/raw/train.csv
aws s3 cp week2_preprocessing/data/raw/store.csv s3://quanvan-ml-forecasting-2026/ml-forecasting/data/raw/store.csv
```

> **Version consistency:** the model, scaler, and preprocessing code must always match in version — discrepancies here are a common source of hard-to-debug errors during serving later (see the XGBoost version mismatch error in Part 5.3).[cite: 4]