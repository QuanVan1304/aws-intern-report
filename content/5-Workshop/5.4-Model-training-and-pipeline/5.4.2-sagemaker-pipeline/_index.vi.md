---
title: "Tự động hóa bằng SageMaker Pipeline"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# Tự động hóa vòng đời huấn luyện bằng SageMaker Pipeline

## Bước ngoặt: yêu cầu tăng quota được duyệt

Sau khi chuyển sang tài khoản riêng (Phần 2), quota Training Job vẫn = 0. Một yêu cầu tăng quota được gửi qua Service Quotas API cho instance `ml.m5.large`, và **được AWS phê duyệt**, nâng quota từ 0 lên 1. Đây là bước ngoặt cho phép chuyển từ giải pháp thay thế local (`simple_orchestration.py`) sang chạy Pipeline thật trên AWS.

## Giải quyết xung đột phụ thuộc SDK

Cài đặt SDK không ghim phiên bản (`pip install sagemaker`) khiến môi trường tự động tải bản mới nhất, gây lỗi `ModuleNotFoundError`/`ImportError` khi dùng `sagemaker.workflow` — do xung đột phụ thuộc nội bộ giữa `sagemaker` và `sagemaker-core`. Sau khi xác nhận vấn đề nằm ở tính ổn định của SDK (bằng cách tạm chuyển sang boto3 thuần và tái hiện được giới hạn tương tự độc lập), dự án ghim `sagemaker==2.257.5` — phiên bản đã kiểm chứng hoạt động ổn định với pipeline cụ thể này.

{{% notice note %}}
Đây là lựa chọn thực dụng cho dự án, không phải đánh giá SDK v2 tốt hơn v3 về tổng thể — AWS đang định hướng phát triển chính thức sang v3 (xem thử nghiệm v3 ở mục 5.4.2.6 bên dưới).
{{% /notice %}}

## Xây dựng và debug Pipeline thật — 3 lỗi thực tế

`pipeline_definition.py` định nghĩa Pipeline với 1 bước Training (XGBoost), dùng `Estimator` + `TrainingStep` + `Pipeline` của SDK.

| # | Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|---|
| 1 | `ClientError: 404 HeadObject` khi Training Job chạy | Bước Training thiếu `HyperParameters` trỏ đúng `sagemaker_program`/`sagemaker_submit_directory` — container không biết code training nằm ở đâu trên S3 | Tự động đóng gói code training (`train_xgboost_sm.py` + `feature_utils.py`) thành `sourcedir.tar.gz`, upload trước mỗi lần chạy, truyền đúng S3 URI vào `source_dir` của Estimator |
| 2 | `"You can't override the metric definitions for Amazon SageMaker algorithms"` khi tạo Training Job | `sagemaker-xgboost:1.7-1` được AWS phân loại là **built-in algorithm image**; ở script mode, container không tự log metric, nhưng AWS cũng không cho phép tự khai báo `MetricDefinitions` để bù lại | Bỏ hẳn bước Condition khỏi Pipeline — kiểm tra chất lượng model chuyển thành bước hậu kiểm độc lập ngoài Pipeline (`build_real_features.py`, mục 5.5.1), đúng mô hình tách biệt CI (train)/CD (deploy có kiểm soát) |
| 3 | Pipeline sinh rác — mỗi lần `create_pipeline()` tạo 1 object mới có timestamp trong tên | Code ban đầu tự nối `datetime.now()` vào tên pipeline | Chuyển sang **Pipeline Tĩnh**: tên cố định `Rossmann-Sales-Pipeline`, dùng `pipeline.upsert()`; timestamp chỉ còn ở `execution_display_name`. Dọn dẹp 9 pipeline rác đã sinh ra trong lúc debug |

## Kết quả — Pipeline chạy thành công thật trên AWS

```json
{
    "StepName": "Rossmann-Model-Training",
    "StepStatus": "Succeeded",
    "Metadata": {
        "TrainingJob": {
            "Arn": ".../training-job/pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc"
        }
    }
}
```

Training Job hoàn tất, sinh model artifact thật trên S3:
```
s3://quanvan-ml-forecasting-2026/ml-forecasting/models/pipeline-artifacts/pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc/output/model.tar.gz
```

Xác nhận Pipeline đã tĩnh hoàn toàn qua `list-pipelines`:
```
-----------------------------
|       ListPipelines       |
+---------------------------+
|  Rossmann-Sales-Pipeline  |
+---------------------------+
```

Model này chính là model được deploy và validate ở Phần 5.5.

{{% notice note %}}
**Vì sao "Evaluation metrics" trên SageMaker Console luôn trống:** mục này chỉ đọc từ `FinalMetricDataList`, trường này rỗng cho mọi Training Job trong Pipeline (đã xác nhận qua CLI) — đúng như lỗi #2 ở trên, **không phải dấu hiệu model kém chất lượng**. Chất lượng thật được xác nhận qua log CloudWatch (`print(f"Val RMSE: ...")`) và Quality Gate độc lập (mục 5.5.1).
{{% /notice %}}

## Pipeline tự chủ hoàn toàn (self-contained)

Phiên bản đầu tiên chạy được của `pipeline_definition.py` trỏ `source_dir` vào một vị trí S3 cố định — thực chất là artifact của một lần chạy Training Job đơn lẻ, thủ công, có trước cả Pipeline này. Điều này khiến Pipeline phụ thuộc vào 1 object S3 nằm ngoài phạm vi quản lý của chính nó.

**Đã khắc phục:** `pipeline_definition.py` giờ có hàm `package_and_upload_source()` — tự động đóng gói `train_xgboost_sm.py` + `feature_utils.py` thành `sourcedir.tar.gz` và upload lên S3 **ngay trong mỗi lần chạy**, trước khi tạo Estimator:

```python
def package_and_upload_source():
    for fname in SOURCE_FILES:
        fpath = os.path.join(SCRIPT_DIR, fname)
        if not os.path.exists(fpath):
            raise FileNotFoundError(f"Khong tim thay {fname}...")
    with tarfile.open(LOCAL_TARBALL, "w:gz") as tar:
        for fname in SOURCE_FILES:
            tar.add(os.path.join(SCRIPT_DIR, fname), arcname=fname)
    s3 = boto3.client("s3", region_name=REGION)
    s3.upload_file(LOCAL_TARBALL, BUCKET, SOURCE_S3_KEY)
    return f"s3://{BUCKET}/{SOURCE_S3_KEY}"
```

Pipeline giờ hoàn toàn tự chủ — không còn phụ thuộc artifact của một job cũ, không liên quan.

## Thử nghiệm SDK v3 (phụ lục kỹ thuật, không phải giải pháp chính thức)

Thử nghiệm migrate sang SageMaker Python SDK v3 trong môi trường ảo riêng (`sagemaker-v3-env`), không ảnh hưởng Pipeline v2 đang chạy chính thức.

**Động lực:** kỳ vọng `ModelTrainer.with_metric_definitions()` của v3 giải quyết được vấn đề "Evaluation metrics trống" mà không cần đổi container.

| Vấn đề | Kết quả |
|---|---|
| Migrate `Estimator` → `ModelTrainer`, `Pipeline` → `sagemaker.mlops.workflow.pipeline.Pipeline` | Chạy được, tạo Pipeline và khởi động Training Job thành công về cấu hình |
| `MetricDefinition` tùy chỉnh qua `.with_metric_definitions()` | **Vẫn bị AWS từ chối** với đúng lỗi y hệt v2 — xác nhận đây là giới hạn từ **AWS Service backend** đối với container built-in algorithm, không liên quan SDK version |
| Đóng gói code tự động qua `SourceCode(source_dir=...)` trên Windows | Gặp lỗi `sm_train.sh: line 1: $'\r': command not found` — script wrapper do SDK tự sinh bị lỗi line-ending CRLF, dù code nguồn (`train_xgboost_sm.py`, `feature_utils.py`) đã xác nhận sạch (LF) qua kiểm tra trực tiếp. Hạn chế đã biết của SDK v3 trên Windows |

**Kết luận:** Xác nhận 2 phát hiện: (1) giới hạn `MetricDefinitions` nằm ở tầng AWS Service, độc lập SDK version; (2) SDK v3 hiện chưa ổn định hoàn toàn trên Windows. Quyết định giữ SDK v2 (đã kiểm chứng `Succeeded` hoàn chỉnh) làm giải pháp chính thức; migrate v3 ghi nhận là hướng phát triển tiếp theo.

## Giải pháp thay thế trước đó, nay đã được thay thế

Trước khi quota được duyệt, `simple_orchestration.py` được xây dựng làm giải pháp thay thế: một script Python thuần gọi tuần tự `subprocess` qua Preprocessing → Training (local) → Deploy → Test, chứng minh cùng luồng logic mà không cần SageMaker Pipelines. Có kèm quality gate tự động (ngưỡng MAPE 15%). Script này đã hoàn thành đúng vai trò ở giai đoạn đó, được giữ lại trong repo như dấu mốc, nhưng SageMaker Pipeline thật (mô tả ở trên) mới là hệ thống dùng cho kết quả cuối cùng của dự án.

## Kiến trúc tổng thể vòng đời Training

```text
[Data Preprocessing] → S3 (train.csv, val.csv)
        │
        ▼
pipeline_definition.py (SDK v2, ghim 2.257.5)
   package_and_upload_source() → sourcedir.tar.gz tự đóng gói
   pipeline.upsert("Rossmann-Sales-Pipeline") + start(run-YYYYMMDD-HHMMSS)
        │
        ▼
   [Pipeline: Rossmann-Sales-Pipeline — Tĩnh, 1 object duy nhất]
   — TrainingStep: ml.m5.large (Completed: ~12 phút)
   — Status: Succeeded
        │
        ▼
   Model Artifact thật lưu trên S3 (pipeline-artifacts/)
```

{{% notice tip %}}
Pipeline chỉ đảm nhiệm phần Train (CI). Deploy Endpoint là bước riêng biệt, có kiểm soát (CD) — không tự động kích hoạt ngay cả khi Training Job thành công, tránh deploy một model chưa qua Quality Gate (mục 5.5.1).
{{% /notice %}}
