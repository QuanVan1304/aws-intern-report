---
title: "Yêu cầu Tiền đề & Hạ tầng"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Phần 2 — Yêu cầu Tiền đề & Cấu hình AWS Credentials

## 2.1. Yêu cầu về Service Quota trên AWS

Để chạy được các công việc huấn luyện (Training Jobs) thực tế trên AWS SageMaker, tài khoản AWS triển khai dự án bắt buộc phải có đủ hạn mức (Quota). Rất nhiều tài khoản mặc định hoặc tài khoản tổ chức có thể bị giới hạn **quota SageMaker Training Jobs = 0** cho toàn bộ các loại instance. Điều này sẽ chặn hoàn toàn khả năng chạy Training Job thật (kể cả khi chạy thông qua SageMaker Pipeline).

> **Bài học rút ra:** Khi gặp lỗi `ResourceLimitExceeded`, việc đầu tiên cần làm là kiểm tra Service Quota, không phải thay đổi instance type. Nếu quota = 0 cho toàn bộ region trên tài khoản hiện có, cần gửi yêu cầu tăng quota cho AWS Support (ví dụ: xin tăng lên 1 cho `ml.m5.large`) hoặc chuyển sang môi trường không bị giới hạn để không làm gián đoạn tiến độ triển khai dự án.

## 2.2. Kiểm tra Service Quota trước khi bắt đầu

```python
import boto3

quotas = boto3.client('service-quotas', region_name='ap-southeast-1')
response = quotas.list_service_quotas(ServiceCode='sagemaker', MaxResults=100)

for q in response['Quotas']:
    if q['Value'] > 0:
        print(f"{q['Value']:>6.0f}  {q['QuotaName']}")
```

Ở giai đoạn đầu, quota Endpoint (`ml.t2.medium`) đã đủ dùng, trong khi quota Training Job vẫn bằng 0 cho toàn bộ loại instance — đây là lý do model dùng để deploy lần đầu được train **local** trước, sau đó mới chuyển sang train thật trên SageMaker khi quota được duyệt.

## 2.3. Tạo IAM User và Role

- Tạo IAM User `admin-user` để bootstrap hạ tầng ban đầu (tạo bucket, role...).
- Tạo riêng **IAM Role cho SageMaker Execution**: `SageMaker-ExecutionRole-QuanVan`.
- Trong thực tế triển khai, Role này được gắn các **managed policy phạm vi rộng** (`AmazonSageMakerFullAccess`, `AmazonS3FullAccess`, `CloudWatchFullAccess`) do áp lực thời gian khi debug nhiều lỗi deploy liên tiếp.

### Chính sách Phân quyền IAM khuyến nghị (least-privilege) cho triển khai chuẩn

Đối với người triển khai lại dự án hoặc dùng cho môi trường production, khuyến nghị dùng policy tối thiểu thay vì FullAccess:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ProjectBucketOnly",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    },
    {
      "Sid": "SageMakerModelAndEndpoint",
      "Effect": "Allow",
      "Action": [
        "sagemaker:CreateModel", "sagemaker:DescribeModel", "sagemaker:DeleteModel",
        "sagemaker:CreateEndpointConfig", "sagemaker:DeleteEndpointConfig",
        "sagemaker:CreateEndpoint", "sagemaker:UpdateEndpoint",
        "sagemaker:DescribeEndpoint", "sagemaker:InvokeEndpoint", "sagemaker:DeleteEndpoint",
        "sagemaker:CreateTrainingJob", "sagemaker:DescribeTrainingJob",
        "sagemaker:CreatePipeline", "sagemaker:UpdatePipeline",
        "sagemaker:StartPipelineExecution", "sagemaker:DescribePipelineExecution"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ECRPullBuiltInContainer",
      "Effect": "Allow",
      "Action": ["ecr:GetAuthorizationToken", "ecr:BatchGetImage",
                 "ecr:GetDownloadUrlForLayer", "ecr:BatchCheckLayerAvailability"],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchLogsForSageMaker",
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents", "logs:GetLogEvents"],
      "Resource": "arn:aws:logs:*:*:log-group:/aws/sagemaker/*"
    },
    {
      "Sid": "LambdaApiGateway",
      "Effect": "Allow",
      "Action": ["lambda:*", "apigateway:*"],
      "Resource": "*"
    }
  ]
}
```

> **Ghi chú:** Việc dùng FullAccess trong quá trình phát triển thực tế giúp tránh mất thời gian debug lỗi permission khi đang gấp rút xử lý các lỗi deploy khác (xem Phần 5.3) — đánh đổi hợp lý cho giai đoạn phát triển nhanh, nhưng nên siết lại thành least-privilege trước khi đưa vào môi trường thật lâu dài.

## 2.4. Tạo S3 Bucket

Bucket riêng: `quanvan-ml-forecasting-2026` (region `ap-southeast-1`).

Cấu trúc thư mục:
```
quanvan-ml-forecasting-2026/ml-forecasting/
├── data/
│   ├── raw/                     ← train.csv, store.csv tải từ Kaggle
│   └── processed/                ← train.csv, val.csv, test.csv, scaler.pkl
├── pipeline-code/                 ← sourcedir.tar.gz do Pipeline tự đóng gói (Phần 4.4)
└── models/
    ├── artifacts/                 ← model deploy đơn lẻ (Phần 5)
    └── pipeline-artifacts/         ← model sinh ra từ SageMaker Pipeline
```

## 2.5. Khởi tạo Môi trường Local

```powershell
# 1. Clone dự án về máy cục bộ
git clone https://github.com/YOUR-USERNAME/aws-internship-ML-forecasting.git
cd aws-internship-ML-forecasting

# 2. Khởi tạo và kích hoạt môi trường ảo Python
python -m venv venv
.\venv\Scripts\activate

# 3. Cài đặt toàn bộ thư viện phụ thuộc
pip install -r requirements.txt

# 4. Kiểm tra cấu hình kết nối AWS
python verify_setup.py
```

## 2.6. Cấu hình `config.py`

```python
# config.py — Cấu hình thông tin tài khoản và tài nguyên AWS
BUCKET_NAME = "quanvan-ml-forecasting-2026"
REGION = "ap-southeast-1"
PREFIX = "ml-forecasting"

S3_RAW_DATA = f"s3://{BUCKET_NAME}/{PREFIX}/data/raw/"
S3_PROCESSED_DATA = f"s3://{BUCKET_NAME}/{PREFIX}/data/processed/"
S3_MODEL_ARTIFACTS = f"s3://{BUCKET_NAME}/{PREFIX}/models/artifacts/"
SAGEMAKER_ROLE_ARN = "arn:aws:iam::897355252080:role/SageMaker-ExecutionRole-QuanVan"
```

## 2.7. Công cụ và Thư viện

| Công cụ | Phiên bản/Ghi chú |
|---|---|
| Python | Venv riêng biệt cho mỗi mục đích (môi trường chính, và `sagemaker-v3-env` riêng cho thử nghiệm SDK v3) |
| `boto3` | Dùng trực tiếp (thay vì SageMaker SDK) cho các thao tác deploy/đóng gói ở Tuần 6, để tự kiểm soát toàn bộ quá trình và tránh vấn đề tương thích SDK |
| SageMaker Python SDK | Ghim `sagemaker==2.257.5` — dùng riêng để xây dựng Pipeline (Phần 4.4), phiên bản đã kiểm chứng ổn định sau khi gặp xung đột phụ thuộc với bản mới hơn (`sagemaker-core`) |
| AWS CLI v2 | Cấu hình profile cho account `897355252080` |
| `xgboost` | Phải khớp version với container built-in dùng lúc serving (`1.7.6` local ↔ container `sagemaker-xgboost:1.7-1`) — đây là nguồn lỗi thực tế gặp phải, xem Phần 5.3 |
| `curl` / `curl.exe` | Test REST API sau khi có Endpoint |