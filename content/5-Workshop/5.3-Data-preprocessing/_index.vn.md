---
title: "Tiền xử lý Dữ liệu"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
# Phần 3 — Tiền xử lý Dữ liệu & Đưa Dữ liệu lên Amazon S3

## 3.1. Chuẩn bị dữ liệu (độc lập, không phụ thuộc tài khoản cũ)

1. Tải trực tiếp raw dataset Rossmann Store Sales (`train.csv`, `store.csv`) từ Kaggle.
2. Đặt vào đúng cấu trúc thư mục local dự án.
3. Chạy `preprocessing.py` để tạo `train.csv`, `val.csv`, `test.csv`, `scaler.pkl` đã xử lý.
4. Upload toàn bộ data (raw + processed) lên bucket `quanvan-ml-forecasting-2026`.

### Cấu trúc Tập tin Dữ liệu Gốc

```
data/raw/
├── train.csv    — 1,017,209 dòng × 9 cột (lịch sử giao dịch)
└── store.csv    — 1,115 dòng × 10 cột (thông tin cửa hàng)
```

## 3.2. Thực thi Kịch bản Tiền xử lý

```bash
python week2_preprocessing/preprocessing.py
```

## 3.3. EDA — Những phát hiện quan trọng định hướng feature engineering

| Phát hiện | Ý nghĩa |
|---|---|
| Phân phối `Sales` lệch phải (right-skewed) | Cân nhắc log-transform nếu train theo hướng đó (dự án cuối cùng train trực tiếp trên `Sales` gốc — xem lưu ý ở Phần 5.3 về lỗi liên quan) |
| 172,817 bản ghi có `Open = 0` hoặc `Sales = 0` | Loại bỏ hoàn toàn trước khi train, vì đây không phải tín hiệu dự báo hữu ích |
| `Promo` làm tăng ~37% doanh số trung bình | Trở thành 1 trong 2 feature quan trọng nhất theo phân tích SHAP (Phần 4.3) |
| Doanh số Tháng 12 cao vượt trội | Có seasonality mạnh theo mùa — hợp lý vì dữ liệu chỉ đến 2015-07-31 nên hiệu ứng này quan sát được ở các năm trước đó trong tập train |

## 3.4. Quy trình 4 Bước Tiền xử lý & Tạo Đặc trưng

### Bước 1 — Gộp dữ liệu (Data Merging)
Kết hợp metadata từ `store.csv` vào `train.csv` theo khóa chính `Store`.

### Bước 2 — Lọc dữ liệu rác (Data Cleaning)
Loại bỏ 172,817 bản ghi khi cửa hàng đóng cửa (`Open = 0`) hoặc doanh số bằng 0.

### Bước 3 — Tạo 22 Đặc trưng Kỹ thuật (Feature Engineering)

| Nhóm | Features |
|---|---|
| **Đặc trưng Lịch (Calendar)** | `Year`, `Month`, `Day`, `DayOfWeek`, `WeekOfYear`, `IsWeekend` |
| **Business-known (đã biết trước, không phải data leakage)** | `Promo`, `StateHoliday`, `SchoolHoliday` — đây là quyết định kinh doanh đã được lên lịch từ trước, khác hẳn `Sales` (kết quả chưa biết) |
| **Đặc trưng cửa hàng (tĩnh)** | `StoreType`, `Assortment`, `CompetitionDistance`, `Promo2` |
| **Độ trễ Thời gian (Lag Features)** | `sales_lag_7`, `sales_lag_14`, `sales_lag_30` |
| **Trung bình/Độ lệch chuẩn Trượt (Rolling)** | `rolling_mean_7/14/30`, `rolling_std_7/14/30` |

> **Lưu ý quan trọng — phân biệt rõ 2 loại feature khi build input cho inference:** `Promo`/`StateHoliday`/`SchoolHoliday` phải lấy đúng giá trị của **chính ngày cần dự đoán** (đã biết trước). Ngược lại, `sales_lag_*` và `rolling_*` chỉ được tính từ dữ liệu **trước** ngày dự đoán, để tránh nhìn thấy tương lai (data leakage). Nhầm lẫn 2 loại này từng gây sai lệch 23.01% trong lần validate đầu tiên (xem chi tiết Phần 5.4).

### Bước 4 — Phân chia Dữ liệu theo Thời gian (Chronological Split)

| Split | Khoảng thời gian | Số dòng |
|---|---|---|
| Train | 2013-01-01 → 2015-05-31 | 785,727 |
| Validation | 2015-06-01 → 2015-06-30 | 28,423 |
| Test | 2015-07-01 → 2015-07-31 | 30,188 |

> Split theo thời gian (không random) là bắt buộc cho bài toán time series, tránh data leakage từ tương lai vào quá khứ khi tạo lag/rolling features.

## 3.5. Upload lên S3

```powershell
aws s3 cp week2_preprocessing/data/processed/ s3://quanvan-ml-forecasting-2026/ml-forecasting/data/processed/ --recursive
aws s3 cp week2_preprocessing/data/raw/train.csv s3://quanvan-ml-forecasting-2026/ml-forecasting/data/raw/train.csv
aws s3 cp week2_preprocessing/data/raw/store.csv s3://quanvan-ml-forecasting-2026/ml-forecasting/data/raw/store.csv
```

> **Version consistency:** model, scaler, và code tiền xử lý phải luôn khớp phiên bản với nhau — sai lệch ở đây là nguồn lỗi khó debug phổ biến khi serving về sau (xem lỗi version mismatch XGBoost ở Phần 5.3).