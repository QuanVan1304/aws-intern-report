---
title: "Nhật ký công việc"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

Dự án **E-commerce Sales Forecasting trên AWS SageMaker** được nhóm (bao gồm Huỳnh Kim Quý, Văn Thái Quân, Nguyễn Ngọc Sáng) thực hiện và hoàn thành xuất sắc trong vòng **08 tuần**. 

Trong trang này, nhóm trình bày tổng quan nhật ký công việc (worklog) chi tiết, những thành quả cốt lõi đạt được, cũng như cách nhóm linh hoạt xử lý các giới hạn tài nguyên (quota) trên AWS Cloud.

---

## 1. Tóm tắt Tiến trình 8 Tuần

**Tuần 1:** [Thiết lập môi trường AWS (khởi tạo Account, S3 Bucket và cấu hình IAM Role)](1.1-week1/)

**Tuần 2:** [Khám phá dữ liệu (EDA) và Tiền xử lý dữ liệu](1.2-week2/)

**Tuần 3:** [Huấn luyện mô hình cơ sở (XGBoost Baseline) và xây dựng bộ khung LSTM](1.3-week3/)

**Tuần 4:** [Huấn luyện mô hình PyTorch LSTM và đánh giá hiệu năng](1.4-week4/)

**Tuần 5:** [Thiết lập Model Registry và phân tích khả năng giải thích của mô hình với SHAP](1.5-week5/)

**Tuần 6:** [Triển khai mô hình lên SageMaker Endpoint và xây dựng Serverless REST API](1.6-week6/)

**Tuần 7:** [Cấu hình Giám sát, giả lập Drift Detection và tạo CloudWatch Dashboard](1.7-week7/)

**Tuần 8:** [Tự động hóa luồng làm việc với SageMaker Pipeline (IaC) và cấu trúc lại mã nguồn](1.8-week8/)

---

## 2. Quản lý Môi trường & Giải pháp tình thế (Workarounds)

Trong quá trình thực tập, nhóm đã linh hoạt sử dụng **2 tài khoản AWS** để đảm bảo tiến độ dự án không bị gián đoạn bởi các giới hạn (quota) của dịch vụ:

*   **Account Team (Tuần 1–5):** `119505195050` – Sử dụng để lưu trữ dữ liệu thô/đã xử lý tại S3 Bucket `aws-internship-hkq-2026`.
*   **Account Cá nhân (Tuần 6–8):** `897355252080` – Sử dụng để deploy SageMaker Endpoint và Pipeline do account team bị block quota (S3 Bucket: `quanvan-ml-forecasting-2026`).

**Các rào cản kỹ thuật và cách nhóm vượt qua:**
*   **SageMaker Training Jobs/Pipelines (Quota = 0):** Chuyển sang huấn luyện local, log metrics thông qua boto3 và xây dựng kịch bản luồng tự động (local orchestration script).
*   **SageMaker Model Registry (Quota = 0):** Thiết kế workaround bằng cách lưu metadata dưới dạng JSON trực tiếp lên S3.
*   **Lỗi SDK & CLI:** Thay vì dùng SageMaker SDK 3.x (đang bị lỗi), nhóm trực tiếp sử dụng `boto3.client()` để kiểm soát hệ thống một cách tối đa.

---

## 3. Tổng kết Thành quả Cốt lõi

*   **Hiệu năng Mô hình:** XGBoost đạt Test RMSE **925.28** và Test MAPE **9.92%**, vượt qua LSTM (MAPE 32.79%) để trở thành mô hình Production.
*   **Hệ thống API Serverless:** Triển khai thành công REST API công khai với thời gian phản hồi ~1.1s. Khi kiểm chứng với dữ liệu thực tế, độ sai lệch dự báo chỉ ở mức **5.14%** (nằm sâu trong ngưỡng an toàn 15%).
*   **Giám sát Chủ động:** Kịch bản Drift Detection hoạt động hoàn hảo: phát hiện chính xác 2 cảnh báo khi dữ liệu bị trôi dạt và không có báo động giả (0 alerts) đối với dữ liệu gốc.
*   **Luồng dữ liệu tích hợp:** Xây dựng thành công kiến trúc End-to-End từ Khâu Tiền xử lý dữ liệu ➔ Huấn luyện ➔ Deploy ➔ Serverless API, với tổng thời gian chạy luồng chỉ mất **~587 giây**.