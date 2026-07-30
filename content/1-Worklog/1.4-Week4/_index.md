---
title: "Week 4 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---
### Week 4 Objectives:
* Train the PyTorch LSTM deep learning model.
* Execute a comprehensive evaluation and direct performance comparison against the XGBoost baseline to determine the final model for deployment.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Develop the core training script in `week4_lstm/train_lstm.py`. <br> - Program a custom PyTorch training loop to maintain granular control over forward and backward propagation passes. <br> - Initialize the DataLoader configured with `sequence_length=7`. | 07/06/2026 | 07/06/2026 | PyTorch Docs |
| 3 | - Configure the Adam optimizer for neural network weight adjustments. <br> - Integrate the `ReduceLROnPlateau` scheduler to dynamically decay the learning rate when loss plateaus. <br> - Implement the logic to automatically save the best model artifact based on validation loss improvements. | 07/07/2026 | 07/07/2026 | PyTorch Docs |
| 4 | - Initiate the LSTM neural network training phase, executing exactly 20 epochs heavily constrained on CPU hardware. <br> - Monitor, collect, and meticulously log evaluation metrics (Train Loss, Val Loss, Val RMSE, Val MAPE) throughout the training cycles. | 07/08/2026 | 07/08/2026 | PyTorch Docs |
| 5 | - Evaluate the trained LSTM model exclusively on the unseen test set. <br> - Apply an inverse transform mechanism to mathematically revert predicted values back to original Sales currency scales, ensuring accurate metric comparisons. <br> - Conduct a direct analytical comparison of RMSE and MAPE between the LSTM and XGBoost baseline. | 07/09/2026 | 07/09/2026 | Scikit-Learn Docs |
| 6 | - Save the optimal model locally at `week4_lstm/models/lstm_best.pt`. <br> - Manually upload the trained model artifact to the designated S3 cloud storage path: `s3://aws-internship-hkq-2026/ml-forecasting/models/artifacts/lstm_best.pt`. | 07/10/2026 | 07/10/2026 | Boto3 Docs |

### Week 4 Achievements:

* **LSTM Training Log Analysis:** The 20-epoch training process demonstrated steady, albeit slow, convergence. By Epoch 20, Train Loss dropped to 0.9734, Val Loss to 0.9866, Val RMSE was 3236.20, and Val MAPE rested at 35.01%.
* **Test Set Performance Breakdown (Model Comparison):**
  * LSTM Model: Test RMSE recorded at 3044.43; Test MAPE at 32.79%.
  * XGBoost Model: Test RMSE achieved **925.28**; Test MAPE achieved an impressive **9.92%**.
  * *Technical Decision:* The XGBoost model definitively outperformed the LSTM by a factor of three. Consequently, XGBoost was officially selected as the primary production model destined for the deployment pipeline.

### In-depth Root Cause Analysis of LSTM Underperformance:
1. **Unnormalized Features:** The LSTM architecture is inherently hyper-sensitive to the scale of input features. The raw utilization of columns like `Store` (ranging 1–1115) and `CompetitionDistance` (spanning from hundreds to tens of thousands) introduced massive scale variance, heavily disrupting the neural network's learning capability.
2. **Severely Short Sequence Length:** Constraining the `sequence_length` to 7 effectively prevented the LSTM from observing and learning critical long-term temporal patterns embedded in the data.
3. **Absence of Engineered Lag Features:** Unlike XGBoost, which heavily leveraged 7, 14, and 30-day lag features, the LSTM model was restricted strictly to raw features, causing a massive deficit in historical trend recognition.
4. **Hardware Limitations:** Executing the training phase solely on a CPU drastically restricted computational throughput, making it mathematically unfeasible to run the higher number of epochs required for deep convergence.

### Technical Notes & Workarounds (Lessons Learned):
* The `sequence_length` was forcefully capped at 7 because the validation and test datasets only contained exactly one month of chronological data.
* **Service Disruption Handled:** An attempt to utilize an automated SageMaker Training Job failed due to a strict AWS account quota block (quota=0). The successful workaround involved shifting the entire training workload to a local environment and executing a manual upload of the resulting artifact to S3.