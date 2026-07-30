---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Quản lý vòng đời mô hình (Model Registry) và xây dựng bộ khung kịch bản suy luận (Inference skeleton) chuẩn bị cho việc triển khai.
* Phân tích khả năng giải thích của mô hình (Explainability) thông qua thư viện SHAP nhằm minh bạch hóa các quyết định dự đoán của XGBoost.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Khởi tạo kịch bản phân tích `week5_registry/shap_analysis.py`. <br> - Khai báo cấu trúc SHAP TreeExplainer chuyên dụng cho mô hình XGBoost. <br> - Thực hiện trích xuất ngẫu nhiên (sample) 1,000 dòng dữ liệu từ tập huấn luyện (train set) để tối ưu thời gian tính toán giá trị SHAP. | 13/07/2026 | 13/07/2026 | SHAP Docs |
| 3 | - Trực quan hóa mức độ đóng góp của các đặc trưng thông qua việc xuất 2 biểu đồ: Feature Importance bar chart (biểu đồ cột) và Summary dot plot (biểu đồ phân tán). <br> - Tải các biểu đồ phân tích (plots) lên lưu trữ đám mây S3 tại thư mục `outputs/evaluation/`. | 14/07/2026 | 14/07/2026 | SHAP, Boto3 |
| 4 | - Viết kịch bản `week5_registry/model_registry.py`. <br> - Thiết kế giải pháp tình thế (workaround) do dịch vụ SageMaker Model Registry chính thức bị block quota (limit = 0). <br> - Xây dựng cơ chế lưu trữ siêu dữ liệu (metadata) của mô hình dưới định dạng JSON ngay trên máy cục bộ (local). | 15/07/2026 | 15/07/2026 | Boto3 Docs |
| 5 | - Cập nhật và đăng ký siêu dữ liệu của cả 2 mô hình (XGBoost và LSTM) vào custom registry. <br> - Gắn đầy đủ các chỉ số hiệu năng (metrics) và trạng thái phê duyệt (`Status: Approved`) vào file JSON. <br> - Tải các file registry JSON lên S3 bucket. | 16/07/2026 | 16/07/2026 | Boto3 Docs |
| 6 | - Viết bộ khung suy luận `week5_registry/inference.py` skeleton. <br> - Lập trình 4 hàm cốt lõi phục vụ SageMaker Endpoint: `model_fn` (load mô hình vào bộ nhớ khi container khởi động); `input_fn` (phân tích chuỗi JSON request thành DataFrame); `predict_fn` (chạy inference và áp dụng nghịch đảo log); và `output_fn` (định dạng kết quả trả về dạng JSON). | 17/07/2026 | 17/07/2026 | SageMaker Docs |

### Kết quả đạt được tuần 5:

* **Khả năng giải thích của mô hình (SHAP Feature Importance):**
  * Hạng 1: Đóng góp cao nhất thuộc về biến `rolling_mean_14` (⭐⭐⭐⭐⭐).
  * Hạng 2: Biến `Promo` (⭐⭐⭐⭐⭐) chứng tỏ các chiến dịch khuyến mãi chi phối mạnh mẽ doanh thu.
  * Hạng 3-5: Lần lượt là `rolling_mean_30` (⭐⭐⭐⭐), `DayOfWeek` (⭐⭐⭐), và `Day` (⭐⭐⭐).
  * Các biến có ít ảnh hưởng nhất (⭐) là `Promo2` và `Assortment`.
* **Thành quả Quản lý Mô hình (Model Registry):**
  * File `XGBoost-Baseline.json` (RMSE 925.28, MAPE 9.92%, Status: Approved ✅).
  * File `LSTM-Forecaster.json` (RMSE 3044.43, MAPE 32.79%, Status: Approved ✅).
* **Cấu trúc lưu trữ S3 sau Tuần 5:** Các tệp tin phân tích SHAP và registry JSON đã được tổ chức gọn gàng và lưu trữ an toàn trên bucket `aws-internship-hkq-2026`.

### Ghi chú Kỹ thuật & Giải pháp (Technical Notes):
* Ghi nhận sự cố dịch vụ: SageMaker Model Registry bị chặn hạn mức (block quota), nhóm đã xử lý khéo léo bằng cách dùng siêu dữ liệu JSON trên S3 để thay thế hoàn toàn chức năng của registry.
* Quy trình trích xuất SHAP bắt buộc phải sử dụng train data thay vì test data. Nguyên nhân là do tập test chỉ có 1 tháng dữ liệu, không đủ bề dày lịch sử để tính toán chính xác các biến trễ (lag features).
* Dựa trên phân tích toàn diện, mô hình XGBoost chính thức được chốt làm mô hình cốt lõi (production model) để triển khai (deploy) vào tuần 6.