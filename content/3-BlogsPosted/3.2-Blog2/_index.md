---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# IS DEEP LEARNING ALWAYS BETTER THAN TRADITIONAL MACHINE LEARNING? LESSONS ON OPTIMIZING PERFORMANCE AND COST ON AWS SAGEMAKER

When embarking on AI/ML projects (like time-series forecasting), many people assume that *using complex Deep Learning models (like LSTM or Transformers) will inherently yield better results than traditional Machine Learning*. However, practical deployment in a Cloud environment like AWS proves otherwise: using a sledgehammer to crack a nut is rarely the best approach.

Below are my practical lessons on balancing model complexity, performance, and infrastructure costs on AWS SageMaker.

### 1. The Showdown on Tabular Data
In my E-commerce sales forecasting project, I experimented with both approaches:
*   **Deep Learning (LSTM):** Despite high expectations, training on CPUs was sluggish, required complex `sequence_length` configurations, and yielded poor results with a Mean Absolute Percentage Error (MAPE) of **~32.79%**.
*   **Traditional Machine Learning (XGBoost):** Ran exceptionally smoothly on basic CPU instances, converged quickly, and achieved a highly accurate MAPE of just **~9.92%**.

The root causes were **Feature Engineering** and **Data Sensitivity**. XGBoost effectively utilized the Lag features and Rolling means I engineered, and it was immune to the disparate scales of the raw data (like competition distance or store IDs). Conversely, LSTM is extremely sensitive to scale and will learn "garbage" if the data is not meticulously normalized.

### 2. The Economics of AWS SageMaker
On AWS SageMaker, the cost of a Training Job is calculated as: **Instance Price × Convergence Time**.
*   **XGBoost:** Only requires cheap CPUs and converges after a few dozen epochs.
*   **LSTM:** Often demands expensive GPUs and hundreds of epochs to converge.

Furthermore, for Inference deployment, a lightweight model like XGBoost can run flawlessly 24/7 on a low-cost `ml.t2.medium` instance. Stubbornly deploying a heavy Deep Learning model can cause Cloud costs to skyrocket without delivering proportional business value.

### 3. Professional Model Lifecycle Management
Instead of tracking metrics manually, I leveraged **SageMaker Experiments** to run multiple configurations (XGBoost vs. LSTM) in parallel, automatically generating comparison tables for metrics (RMSE, MAPE, Training Time). Once XGBoost was crowned the champion, the best artifact was pushed to the **SageMaker Model Registry** for professional version control before hitting Production.

### 4. Common MLOps Anti-patterns
Through this project, I identified several classic pitfalls to avoid:
1.  **Hype-driven Development:** Forcing Deep Learning onto a project with only a few thousand rows of CSV data. Deep Learning truly shines with Unstructured Data (CV, NLP) or massive Big Data.
2.  **Neglecting Feature Engineering:** Relying entirely on neural network architectures while forgetting that data cleaning and transformation is king.
3.  **Ignoring Inference Costs:** Designing a state-of-the-art 2GB model that requires renting a massive, expensive Endpoint running 24/7.

### Conclusion
The true power of AI/ML lies not just in complex algorithms, but in data compatibility and infrastructure optimization. The right question for a Cloud AI/ML Engineer isn't *"Which algorithm is the most complex?"*, but rather: **"Which model delivers the best balance between accuracy, training time, and deployment cost?"**.

[*...\[The post link on AWS Study Group\]...*](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2229015587863401/)