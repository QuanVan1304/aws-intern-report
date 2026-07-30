---
title: "Week 3 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
* Develop and evaluate a robust XGBoost baseline model to establish performance benchmarks.
* Architect the deep learning skeleton for the LSTM neural network in preparation for the upcoming training phase.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Reload the thoroughly processed dataset from Week 2. <br> - Perform Advanced Feature Engineering: Extract lag features over 7, 14, and 30-day intervals to capture historical trends. <br> - Compute rolling features (statistical mean and standard deviation) across 7, 14, and 30-day time windows. | 06/29/2026 | 06/29/2026 | Pandas Docs |
| 3 | - Develop the primary training script: `week3_xgboost/train_xgboost.py`. <br> - Address the introduction of missing values (NaNs) caused by lag features by rigorously applying the `dropna()` method. <br> - Configure the XGBoost model parameters, implementing an early stopping mechanism (`early_stopping_rounds=50`) to mitigate the risk of overfitting. | 06/30/2026 | 06/30/2026 | XGBoost Docs |
| 4 | - Execute the model training phase and comprehensively evaluate the XGBoost model's performance on both the validation and test datasets. <br> - Extract and analyze key error metrics. <br> - Package and securely save the trained model artifact into the `week3_xgboost/models/` directory for future benchmarking. | 07/01/2026 | 07/01/2026 | Scikit-Learn Docs |
| 5 | - Initialize the `week4_lstm/model.py` script to define the neural network's architecture. <br> - Build the `LSTMForecaster` class, which integrates: 2 hidden LSTM layers, a Dropout layer for regularization, and a fully connected Linear layer for output generation. <br> - Successfully validate the architecture via a forward pass test. | 07/02/2026 | 07/02/2026 | PyTorch Docs |
| 6 | - Develop the `week4_lstm/dataset.py` script to standardize the time-series data pipeline for PyTorch. <br> - Construct the `TimeSeriesDataset` class to programmatically generate sliding window sequences isolated by individual Stores. <br> - Successfully test the PyTorch DataLoader to ensure optimal batch data provisioning. | 07/03/2026 | 07/03/2026 | PyTorch Docs |

### Week 3 Achievements:

* **XGBoost Baseline Performance:**
  * Validation Set Metrics: RMSE recorded at 941.21, with a MAPE of 9.92%.
  * Test Set Metrics: RMSE recorded at 925.28, with a MAPE of 9.92%.
  * *Professional Assessment:* These evaluation metrics significantly surpassed the initial expectation threshold (~1,200 RMSE). This XGBoost model provides a highly reliable baseline to compare against the complex LSTM deep learning model in Week 4.
* **LSTM Skeleton Implementation Outcomes:**
  * The `model.py` script successfully processed an Input tensor with dimensions (32, 30, 10) and accurately projected an Output tensor with dimensions (32,).
  * The `dataset.py` system successfully synthesized 752,277 sequences; the resulting tensors were perfectly structured with input X having a shape of (32, 30, 14) and target y having a shape of (32,).

### Technical Notes (Lessons Learned):
* **Training Optimization:** The implementation of `early_stopping_rounds=50` allowed the XGBoost algorithm to automatically halt at epoch 499 out of 500. This indicated that the model maximized its learning capacity without crossing over into overfitting territory.
* **Data Cleansing Logic:** The generation of Lag and Rolling features inherently injected NaN values into the earliest time periods. The strategic use of `dropna()` cleanly sanitized the dataset without compromising the continuity of the time-series data.
* **Mathematical Target Transformations:** Because the target variable `Sales` was mathematically normalized using the natural logarithm `log1p(Sales)` during Week 2 preprocessing, all subsequent model predictions mathematically require the application of the inverse function `expm1()` to convert the predictions back to the original sales currency scale for accurate error calculation.
* **Future Work:** Currently, the LSTM skeleton has only achieved structural validation (input/output). The full neural network training pipeline will be orchestrated in the `train_lstm.py` script during Week 4.