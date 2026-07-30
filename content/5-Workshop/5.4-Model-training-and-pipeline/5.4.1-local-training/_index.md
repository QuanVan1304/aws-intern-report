---
title: "Local Training: XGBoost vs. LSTM"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

## Model comparison results

| Model | Test RMSE | Test MAPE | Training time | Decision |
|---|---|---|---|---|
| **XGBoost Regressor (v1.7.6)** | **925.28** | **9.92%** | **~45 seconds** (local) | ✅ Chosen as production model |
| PyTorch LSTM (2-layer, hidden_size=128) | 3,044.43 | 32.79% | ~8 minutes (20 epochs, CPU) | ❌ Experimental only |

**Why the LSTM underperformed:**
1. Features were not properly normalized — LSTM is sensitive to input scale (`Store` 1–1115 vs `CompetitionDistance` in the thousands).
2. `sequence_length=7` was too short to learn longer-term patterns.
3. The LSTM used only raw features, without the lag/rolling features that XGBoost had.
4. Trained on CPU, limiting the number of epochs that could realistically be run.

{{% notice note %}}
For tabular time-series data like this, XGBoost combined with lag/rolling features clearly outperforms a plain LSTM — both in accuracy and in training speed.
{{% /notice %}}

## Running the XGBoost training

```bash
python week3_xgboost/train_xgboost.py
```

Configuration: `n_estimators=500`, `max_depth=6`, `learning_rate=0.05`, `subsample=0.8`, `colsample_bytree=0.8`, `early_stopping_rounds=20`.

## Feature importance analysis (SHAP)

```bash
python week5_registry/shap_analysis.py
```

| Rank | Feature | Impact | Description |
|---|---|---|---|
| 1 | `rolling_mean_14` | Very high | Average sales over the last 14 days |
| 2 | `Promo` | Very high | Promotion (increases sales by ~37%) |
| 3 | `rolling_mean_30` | High | 30-day sales trend |
| 4 | `DayOfWeek` | Medium | Weekly consumption cycle |
| 5 | `Day` | Medium | Day of the month |
