---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---
# Building an E-commerce Sales Forecasting System with AWS SageMaker

#### Overview

**Sales Forecasting** is a core challenge in the retail industry, helping businesses optimize inventory levels, staffing, and marketing campaigns.

In this lab, you will learn how to build an end-to-end Machine Learning Operations (MLOps) workflow on AWS: from uploading raw data to Amazon S3, training an XGBoost forecasting model, to automating and exposing the model for external consumption.

You will set up two main subsystems to manage the AI model's lifecycle:
+ **SageMaker Pipeline (CI)** - Create an automated pipeline to preprocess data and train the model on ephemeral compute instances, ensuring a consistent and self-contained Continuous Integration (CI) process.
+ **Serverless REST API (Serving)** - Create an intermediary layer using AWS Lambda and Amazon API Gateway to wrap the SageMaker Endpoint. This allows external applications (like a Demo UI) to communicate and receive predictions via standard HTTP requests without direct AWS authentication.

#### Content

1. [Workshop overview](5.1-workshop-overview/)
2. [Prerequisites & Infrastructure](5.2-prerequisites/)
3. [Data Preprocessing](5.3-data-preprocessing/)
4. [Model Training & Pipeline Automation](5.4-model-training-and-pipeline/)
5. [Serverless API & UI Deployment](5.5-endpoint-and-serverless-api/)
6. [Clean up](5.6-cleanup/)