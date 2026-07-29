---
title: "Tổng quan về workshop"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---
# Phần 1 — Tổng quan Workshop & Kiến trúc Dự án

> **Dự án:** E-commerce Sales Forecasting System on AWS SageMaker
> **Chương trình:** AWS First Cloud AI Journey
> **Đơn vị thực hiện:** Amazon Web Services Viet Nam Company Limited
> **Tài liệu dùng chung:** Hướng dẫn triển khai chi tiết từng bước, kèm log thực tế và các lỗi đã gặp/xử lý trong quá trình phát triển.

---

## 1.1. Bối cảnh Dự án

**Rossmann Store Sales** là bộ dữ liệu dự báo doanh số bán lẻ nổi tiếng trên Kaggle, được dùng làm nền tảng cho bài toán dự báo trong dự án này. Bộ dữ liệu bao gồm:

- **1,017,209** bản ghi giao dịch doanh số hàng ngày.
- **1,115** cửa hàng trên toàn nước Đức.
- Lịch sử từ **2013-01-01** đến **2015-07-31** (khoảng 942 ngày).
- **Đặc trưng gốc:** `Store`, `DayOfWeek`, `Date`, `Sales`, `Customers`, `Open`, `Promo`, `StateHoliday`, `SchoolHoliday` (từ `train.csv`), cùng metadata cửa hàng (`StoreType`, `Assortment`, `CompetitionDistance`, `Promo2`...) từ `store.csv`.

## 1.2. Bài toán Kinh doanh

| Thách thức | Chưa có Dự báo | Có Dự báo Machine Learning |
|-----------|----------------|--------------------------|
| **Hàng tồn kho** | Thừa/thiếu hàng hóa cục bộ | Tối ưu hóa mức tồn kho theo từng ngày |
| **Nhân sự** | Phân công ca làm việc dư/thiếu | Đúng người, đúng thời điểm cao điểm |
| **Marketing** | Khuyến mại không đạt hiệu quả | Chọn thời điểm chiến dịch tối ưu (Promo tăng ~37% doanh số — xem Phần 4) |
| **Tài chính** | Biến động doanh thu bất ngờ | Dự báo doanh thu chủ động |

## 1.3. Sơ đồ Kiến trúc Hệ thống (4 Tầng)

```
┌─────────────────────────────────────────────────────────────┐
│                       TẦNG DỮ LIỆU                          │
│  Kaggle Rossmann CSV → Amazon S3 (raw + processed)           │
│  Bucket: quanvan-ml-forecasting-2026 (ap-southeast-1)         │
└─────────────────────────┬─────────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────────┐
│                  TẦNG MACHINE LEARNING                        │
│  Tiền xử lý & Feature Engineering (22 features)                │
│  Train XGBoost (baseline) so sánh với PyTorch LSTM              │
│  Tự động hóa qua SageMaker Pipeline (Training Step thật)        │
│  Kết quả: RMSE 925.28, MAPE 9.92% (XGBoost, model chính)         │
└─────────────────────────┬─────────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────────┐
│                  TẦNG PHỤC VỤ (SERVING)                       │
│  SageMaker Endpoint (ml.t2.medium)                             │
│  → AWS Lambda (rossmann-forecast-api)                           │
│  → API Gateway REST API (POST /forecast, stage prod)             │
│  + Demo UI Dashboard (chạy local, không phụ thuộc AWS)            │
└─────────────────────────┬─────────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────────┐
│                     TẦNG GIÁM SÁT                              │
│  Data Drift Detection (Z-Score, mô phỏng qua drift_simulator.py) │
│  CloudWatch Dashboard: RossmannForecastingDashboard              │
└─────────────────────────────────────────────────────────────────┘
```

## 1.4. Lộ trình Thực hiện (Timeline)

| Giai đoạn | Nội dung | Trạng thái |
|---|---|---|
| Tuần 1–5 | Setup môi trường, EDA, tiền xử lý, train XGBoost/LSTM baseline, SHAP, Model Registry (JSON workaround) | ✅ Hoàn thành |
| Tuần 6 | Deploy SageMaker Endpoint + REST API (Lambda + API Gateway) | ✅ Hoàn thành |
| Tuần 7 | Data Drift Detection + CloudWatch Dashboard | ✅ Hoàn thành |
| Tuần 8 | Tự động hóa bằng SageMaker Pipeline thật trên AWS | ✅ Hoàn thành |

> **Ghi chú quan trọng về hạ tầng:** Từ Tuần 6, phần Deployment/Pipeline được chuyển sang thực hiện trên **tài khoản AWS cá nhân** (`897355252080`, region `ap-southeast-1`) do tài khoản gốc dùng ở Tuần 1–5 (`119505195050`) bị giới hạn quota SageMaker Training Jobs = 0, không thể mở rộng kịp tiến độ. Toàn bộ dữ liệu và model được tự tải/tự train lại độc lập trên tài khoản mới.

## 1.5. Kết quả Thực tế Đạt được

- **Mô hình cốt lõi:** XGBoost Regressor, train bằng `xgboost==1.7.6` (khớp version container serving — xem lý do ở Phần 5).
- **Test RMSE:** 925.28
- **Test MAPE:** 9.92%
- So sánh: PyTorch LSTM (2-layer) đạt RMSE 3,044.43 / MAPE 32.79% — kém hơn nhiều, không được chọn làm model chính (chi tiết Phần 4).
- **Độ chính xác khi validate bằng dữ liệu lịch sử thật:** sai lệch trong khoảng **4.75%–5.14%** qua các lần chạy validate độc lập (mỗi lần train lại model cho kết quả hơi khác do random seed) — đều nằm sâu trong ngưỡng quality-gate 15% đặt ra cho dự án.
- **Chi phí Endpoint:** ~$0.065/giờ (`ml.t2.medium`, region `ap-southeast-1`), được cleanup ngay sau mỗi lần test/demo (xem Phần 6).
- **SageMaker Pipeline:** chạy thành công thật trên AWS sau khi được duyệt tăng quota (xem Phần 4.4), không chỉ là phương án mô phỏng local.

## 1.6. Những thách thức kỹ thuật lớn đã vượt qua

Dự án này không đơn thuần "làm theo hướng dẫn suôn sẻ" — phần lớn giá trị thực học nằm ở việc debug các lỗi hạ tầng thực tế. Tổng cộng đã gặp và xử lý:

- 3 lỗi khi deploy Endpoint (region mismatch, version mismatch XGBoost, lỗi logic `inference.py`) — Phần 5
- 3 lỗi khi dựng SageMaker Pipeline thật (thiếu HyperParameters, giới hạn MetricDefinitions, pipeline sinh rác) — Phần 4
- 1 giới hạn quota ban đầu, được giải quyết bằng cách chuyển account + đồng thời xin tăng quota song song — Phần 2 & Phần 4
- Thử nghiệm SDK v3 độc lập, phát hiện thêm 2 giới hạn kỹ thuật (metric definitions ở tầng AWS Service, và vấn đề line-ending trên Windows) — ghi nhận làm hướng phát triển