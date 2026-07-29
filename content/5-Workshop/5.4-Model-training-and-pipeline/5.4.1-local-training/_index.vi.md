---
title: "Huấn luyện Local: XGBoost vs. LSTM"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

# Huấn luyện Local: XGBoost vs. LSTM

## Kết quả So sánh 2 Mô hình

| Mô hình | Test RMSE | Test MAPE | Thời gian Train | Quyết định |
|---------|----------|----------|-----------------|-----------|
| **XGBoost Regressor (v1.7.6)** | **925.28** | **9.92%** | **~45 giây** (local) | ✅ Chọn làm Production Model |
| PyTorch LSTM (2-layer, hidden_size=128) | 3,044.43 | 32.79% | ~8 phút (20 epochs, CPU) | ❌ Mô hình thử nghiệm |

**Nguyên nhân LSTM kém hơn:**
1. Features không được normalize đúng cách — LSTM nhạy cảm với scale input (`Store` 1–1115 vs `CompetitionDistance` hàng nghìn).
2. `sequence_length=7` quá ngắn để học pattern dài hạn.
3. LSTM chỉ dùng raw features, không có lag/rolling như XGBoost.
4. Train trên CPU, giới hạn số epoch có thể chạy.

{{% notice note %}}
Với dữ liệu chuỗi thời gian dạng bảng (tabular), XGBoost kết hợp lag/rolling features vượt trội hơn nhiều so với LSTM thuần — cả về độ chính xác lẫn tốc độ.
{{% /notice %}}

## Thực thi Huấn luyện XGBoost

```bash
python week3_xgboost/train_xgboost.py
```

Cấu hình: `n_estimators=500`, `max_depth=6`, `learning_rate=0.05`, `subsample=0.8`, `colsample_bytree=0.8`, `early_stopping_rounds=20`.

## Phân tích Mức độ Quan trọng Đặc trưng (SHAP)

```bash
python week5_registry/shap_analysis.py
```

| Thứ tự | Đặc trưng | Mức độ ảnh hưởng | Mô tả |
|--------|-----------|------------------|-------|
| 1 | `rolling_mean_14` | Rất cao | Trung bình doanh số 14 ngày gần nhất |
| 2 | `Promo` | Rất cao | Khuyến mại (tăng ~37% doanh số) |
| 3 | `rolling_mean_30` | Cao | Xu hướng doanh số 30 ngày |
| 4 | `DayOfWeek` | Trung bình | Chu kỳ tiêu dùng trong tuần |
| 5 | `Day` | Trung bình | Ngày trong tháng |
