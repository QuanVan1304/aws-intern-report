---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng Hệ thống Dự báo Doanh số E-commerce với AWS SageMaker

#### Tổng quan

**Dự báo doanh số (Sales Forecasting)** là một bài toán cốt lõi trong ngành bán lẻ, giúp doanh nghiệp tối ưu hóa lượng hàng tồn kho, nhân sự và các chiến dịch marketing.

Trong bài lab này, chúng ta sẽ học cách xây dựng một luồng công việc Machine Learning (MLOps) hoàn chỉnh trên AWS: từ bước đưa dữ liệu thô lên Amazon S3, huấn luyện mô hình dự báo XGBoost, cho đến việc tự động hóa và phơi bày (expose) mô hình ra bên ngoài.

Chúng ta sẽ thiết lập 2 phân hệ chính để quản lý vòng đời của mô hình AI:
+ **SageMaker Pipeline (CI)** - Tạo một luồng tự động hóa để tiền xử lý dữ liệu và huấn luyện mô hình trên các máy chủ ảo tạm thời, đảm bảo quá trình CI (Continuous Integration) diễn ra nhất quán và độc lập.
+ **Serverless REST API (Serving)** - Tạo một lớp trung gian sử dụng AWS Lambda và Amazon API Gateway để bọc SageMaker Endpoint. Điều này cho phép các ứng dụng bên ngoài (như Demo UI) giao tiếp và nhận kết quả dự báo qua HTTP request chuẩn mà không cần xác thực AWS trực tiếp.

#### Nội dung

1. [Tổng quan về workshop](5.1-workshop-overview/)
2. [Yêu cầu Tiền đề & Hạ tầng](5.2-prerequisites/)
3. [Tiền xử lý Dữ liệu](5.3-data-preprocessing/)
4. [Huấn luyện & Tự động hóa Pipeline](5.4-model-training-and-pipeline/)
5. [Triển khai Serverless API & UI](5.5-endpoint-and-serverless-api/)
6. [Dọn dẹp tài nguyên](5.6-cleanup/)