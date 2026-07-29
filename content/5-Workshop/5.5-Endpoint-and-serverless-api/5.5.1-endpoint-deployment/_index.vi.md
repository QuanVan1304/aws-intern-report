---
title: "Deploy Endpoint"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

# Deploy lên SageMaker Endpoint

## Đóng gói Model theo chuẩn SageMaker

`deploy_endpoint.py` dùng **boto3 thuần** (không dùng SageMaker Python SDK) để kiểm soát toàn bộ quá trình đóng gói, tránh vấn đề tương thích SDK.

Script lấy model artifact trực tiếp từ Training Job đã hoàn tất (`describe_training_job` → `ModelArtifacts.S3ModelArtifacts`), tải về, đóng gói lại theo đúng cấu trúc container built-in yêu cầu:

```
model.tar.gz
├── xgboost_model.pkl
└── code/
    └── inference.py
```

Vì dùng boto3 thuần, biến môi trường `SAGEMAKER_PROGRAM` và `SAGEMAKER_SUBMIT_DIRECTORY` phải khai báo **thủ công** trong `create_model()`:

```python
sm.create_model(
    ModelName=model_name,
    PrimaryContainer={
        "Image": XGBOOST_IMAGE,
        "ModelDataUrl": model_url,
        "Environment": {
            "SAGEMAKER_PROGRAM": "inference.py",
            "SAGEMAKER_SUBMIT_DIRECTORY": "/opt/ml/model/code",
        },
    },
    ExecutionRoleArn=ROLE_ARN,
)
```

{{% notice note %}}
Script này được tái sử dụng cho 2 nguồn model: model train local (giai đoạn đầu, khi quota Training Job vẫn = 0) và sau đó, model sinh ra từ SageMaker Training Job thật chạy trong Pipeline (mục 5.4.2). Logic đóng gói giống hệt nhau, chỉ khác tên Training Job nguồn.
{{% /notice %}}

## Deploy Endpoint bằng Boto3

Script tự động kiểm tra endpoint đã tồn tại chưa — `update_endpoint()` nếu có, `create_endpoint()` nếu chưa, cho phép chạy lại nhiều lần không lỗi trùng tên.

- **Endpoint:** `rossmann-forecasting-endpoint`
- **Instance type:** `ml.t2.medium`

**Log thực tế** (deploy model sinh ra từ SageMaker Pipeline, job `pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc`):

```text
=== DEPLOY MODEL TU SAGEMAKER PIPELINE ===
1. Lay thong tin model tu Training Job: pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc...
   Model artifact tren S3: s3://.../pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc/output/model.tar.gz
2. Tai model.tar.gz cua Pipeline ve may...
3. Giai nen va dong goi lai kem inference.py...
4. Uploading len S3...
5. Tao SageMaker Model (tu model cua Pipeline)...
6. Tao Endpoint Config...
7. Tao moi Endpoint 'rossmann-forecasting-endpoint'...
8. Deploying... (cho 5-10 phut)
   -> Status: Creating
   -> Status: InService

✅ ENDPOINT DA SAN SANG DE SU DUNG!
```

## Ba lỗi thực tế đã gặp và cách xử lý

| # | Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|---|
| 1 | `ValidationException` khi `create_model` | Container image URI hardcode account ID region `us-east-1`, trong khi endpoint chạy ở `ap-southeast-1` — mỗi region có account ID riêng cho container built-in | Tra cứu đúng registry path qua tài liệu AWS chính thức (Docker Registry Paths and Example Code), dùng đúng account ID theo region (`121021644041`) |
| 2 | `ModelError` (HTTP 500) khi `invoke_endpoint` | Container built-in `sagemaker-xgboost:1.7-1` không tương thích ngược khi `pickle.load()` model train bằng XGBoost phiên bản mới hơn (`3.2.0`) | Downgrade `xgboost` xuống `1.7.6` khớp container, train lại model, đóng gói và deploy lại |
| 3 | Kết quả trả về bất thường (`Infinity`) | `inference.py` gọi `np.expm1()` lên kết quả dự đoán, nhưng model được train trực tiếp trên `Sales` gốc (không qua `log1p()`), gây tràn số | Bỏ dòng `np.expm1()` trong `predict_fn`, trả thẳng `model.predict(X)` |

{{% notice tip %}}
**Quy trình gỡ lỗi thực tế:** đọc trực tiếp CloudWatch Logs (`aws logs get-log-events`) để lấy traceback thật, sau đó cô lập vấn đề bằng cách test `model.predict()` trực tiếp trên máy local trước khi kết luận lỗi nằm ở tầng hạ tầng hay logic mã nguồn — cách này xác định nhanh vấn đề #3 nằm ở logic code, không phải hạ tầng.
{{% /notice %}}

## Validate Model bằng Dữ liệu Lịch sử Thật (Quality Gate)

Sau khi endpoint hết lỗi 500, viết `build_real_features.py` để validate sâu hơn về độ chính xác — thay vì chỉ kiểm tra endpoint có phản hồi hay không (smoke test), bước này xác nhận **model dự đoán đúng**.

**Cách làm:**
1. Gộp `train.csv` + `val.csv` + `test.csv` thành một dataframe đầy đủ lịch sử, cho phép tra cứu bất kỳ ngày nào.
2. Tính 22 features từ dữ liệu lịch sử thật của một cửa hàng — chỉ lấy `sales_lag_*`/`rolling_*` từ các ngày **trước** ngày dự đoán để tránh data leakage; riêng `Promo`/`StateHoliday`/`SchoolHoliday` lấy đúng giá trị **của chính ngày dự đoán** (đã biết trước, không phải leakage).
3. So sánh kết quả dự đoán với doanh số thực tế, tính % sai lệch.

**Debug qua 2 lần thử:**

| Lần | Cách làm | Sai lệch |
|---|---|---|
| 1 | Suy luận `Promo`/`StateHoliday` từ ngày gần nhất có sẵn (sai cách tiếp cận) | 23.01% |
| 2 | Lấy đúng giá trị thật của **chính ngày dự đoán** | ~5% |

**Kết quả validate chính thức** (model từ SageMaker Pipeline) — Store 1, ngày 2015-06-15:
```text
Doanh so THAT ngay 2015-06-15: 5518.00
Doanh so DU DOAN:           5780.13
Sai lech:                   4.75%
✅ PASS — Sai lech 4.75% trong nguong cho phep (15.0%)
```

Các lần chạy validate khác nhau (mỗi lần model được train lại cho kết quả hơi khác do random seed) cho sai lệch trong khoảng **4.75%–5.14%** — đều nằm sâu trong ngưỡng quality-gate.

{{% notice note %}}
**Phân biệt quan trọng:** `Promo`/`StateHoliday` của **chính ngày cần dự đoán** là dữ liệu kinh doanh đã biết trước (lịch khuyến mãi lên kế hoạch từ trước), không phải rò rỉ dữ liệu. Nhầm lẫn điểm này gây ra sai lệch 23.01% ở lần thử đầu.
{{% /notice %}}

Cơ chế **exit code theo ngưỡng** (`MAPE_THRESHOLD = 15%`) được thêm vào để file có thể dùng như quality gate tự động trong pipeline/orchestration (mục 5.4.2).
