---
title: "Endpoint & Serverless API"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Endpoint & Serverless API

{{% notice note %}}
**Lưu ý quan trọng về kiến trúc:** Mục này gồm **2 hệ thống độc lập**, phục vụ 2 mục đích khác nhau:
- **5.5.1 (Deploy Endpoint):** Model chạy thật trên **SageMaker Endpoint** (cloud), gọi qua API Gateway + Lambda — hệ thống production.
- **5.5.2 (Serverless API & UI):** lớp REST API expose endpoint ra ngoài, cùng với một demo UI chạy **hoàn toàn local**, load trực tiếp `xgboost_model.pkl` vào bộ nhớ và tự predict — **không gọi** SageMaker Endpoint/API Gateway. Mục đích demo nhanh, không cần AWS credentials.
{{% /notice %}}

- **[5.5.1 — Deploy Endpoint](1-endpoint-deployment/)**: đóng gói model, deploy lên SageMaker Endpoint, 3 lỗi thực tế đã sửa, và validate độ chính xác bằng dữ liệu lịch sử thật.
- **[5.5.2 — Serverless API & Demo UI](2-serverless-api-ui/)**: expose endpoint qua Lambda + API Gateway, kiểm thử toàn trình, và demo dashboard chạy local cho trình bày/thuyết trình.
