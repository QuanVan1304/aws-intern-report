---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---
### Mục tiêu tuần 3:
* Phát triển và đánh giá mô hình cơ sở XGBoost (XGBoost baseline) để làm chuẩn mực đo lường.
* Xây dựng bộ khung kiến trúc mạng nơ-ron học sâu LSTM (LSTM skeleton) chuẩn bị cho giai đoạn huấn luyện tiếp theo.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tải lại tập dữ liệu đã qua tiền xử lý từ thành quả của tuần 2. <br> - Thực hiện Kỹ thuật đặc trưng (Feature Engineering): Trích xuất các biến trễ (lag features) với chu kỳ 7, 14 và 30 ngày để mô hình bắt được xu hướng quá khứ. <br> - Tạo các đặc trưng trượt (rolling features) bao gồm giá trị trung bình (mean) và độ lệch chuẩn (std) dựa trên các cửa sổ thời gian 7, 14, 30 ngày. | 29/06/2026 | 29/06/2026 | Pandas Docs |
| 3 | - Xây dựng kịch bản huấn luyện thông qua file `week3_xgboost/train_xgboost.py`. <br> - Xử lý triệt để các giá trị rỗng (NaN) phát sinh ở những dòng đầu tiên do việc tạo lag features bằng cách sử dụng hàm `dropna()`. <br> - Thiết lập cấu hình huấn luyện XGBoost, áp dụng cơ chế dừng sớm (`early_stopping_rounds=50`) để kiểm soát hiện tượng quá khớp (overfitting). | 30/06/2026 | 30/06/2026 | XGBoost Docs |
| 4 | - Tiến hành huấn luyện và đánh giá (evaluate) toàn diện hiệu năng của mô hình XGBoost trên hai tập dữ liệu độc lập: validation set và test set. <br> - Trích xuất các chỉ số đo lường sai số. <br> - Đóng gói và lưu trữ model artifact vào thư mục `week3_xgboost/models/`. | 01/07/2026 | 01/07/2026 | Scikit-Learn Docs |
| 5 | - Khởi tạo file `week4_lstm/model.py` để định nghĩa cấu trúc mạng học sâu. <br> - Lập trình class `LSTMForecaster` với kiến trúc bao gồm: 2 lớp ẩn LSTM (LSTM layers), kết hợp với lớp Dropout để giảm thiểu overfit và lớp Linear để xuất dự đoán. <br> - Kiểm thử thành công luồng truyền xuôi (forward pass) của mạng neural. | 02/07/2026 | 02/07/2026 | PyTorch Docs |
| 6 | - Lập trình file `week4_lstm/dataset.py` nhằm chuẩn hóa định dạng dữ liệu cho PyTorch. <br> - Xây dựng class `TimeSeriesDataset` để tự động cắt dữ liệu thành các chuỗi cửa sổ trượt (sliding window sequences) riêng biệt theo từng cửa hàng (Store). <br> - Kiểm thử thành công PyTorch DataLoader để đảm bảo khả năng cấp phát dữ liệu theo batch. | 03/07/2026 | 03/07/2026 | PyTorch Docs |

### Kết quả đạt được tuần 3:

* **Hiệu năng Mô hình XGBoost Baseline:**
  * Khớp trên Validation set: RMSE đạt 941.21, độ lệch MAPE đạt 9.92%.
  * Khớp trên Test set: RMSE đạt 925.28, độ lệch MAPE đạt 9.92%.
  * *Nhận định chuyên môn:* Kết quả này vượt xa mức kỳ vọng ban đầu (~1,200 RMSE). Mô hình XGBoost này hoàn toàn đủ độ tin cậy để làm baseline so sánh với mô hình học sâu LSTM ở tuần 4.
* **Thành quả thiết lập LSTM Skeleton:**
  * Kịch bản `model.py`: Mô hình xử lý thành công Input tensor có hình dạng (32, 30, 10) và xuất ra Output tensor chuẩn xác với hình dạng (32,).
  * Kịch bản `dataset.py`: Hệ thống tạo thành công 752,277 sequences; cấu trúc phân bổ chuẩn với tập dữ liệu đầu vào X có hình dạng (32, 30, 14) và tập nhãn y có hình dạng (32,).

### Ghi chú Kỹ thuật & Kinh nghiệm rút ra (Technical Notes):
* **Tối ưu huấn luyện:** Thuật toán XGBoost sử dụng tham số `early_stopping_rounds=50` đã tự động kích hoạt dừng sớm tại epoch thứ 499 trên tổng số 500, qua đó chứng minh mô hình học tối đa giới hạn mà chưa xảy ra hiện tượng overfit.
* **Xử lý dữ liệu khuyết thiếu:** Quá trình tạo Lag và Rolling features chắc chắn sinh ra các giá trị NaN ở các chu kỳ thời gian đầu tiên, giải pháp dùng `dropna()` đã làm sạch dữ liệu an toàn mà không làm hỏng tính liên tục của time-series.
* **Toán học trong biến đổi Target:** Do biến mục tiêu `Sales` đã được chuẩn hóa bằng logarit tự nhiên `log1p(Sales)` ở tuần 2, mọi kết quả dự đoán của mô hình đều bắt buộc phải đi qua hàm nghịch đảo `expm1()` để chuyển đổi về đơn vị doanh số gốc nhằm tính toán sai số thực tế.
* **Kế hoạch tiếp nối:** Hiện tại bộ khung LSTM (skeleton) mới chỉ hoàn thiện khâu input/output, kịch bản huấn luyện mạng neural chính thức sẽ được tiến hành trong script `train_lstm.py` vào tuần 4.