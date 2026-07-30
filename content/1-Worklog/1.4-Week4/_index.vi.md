---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Huấn luyện mô hình mạng học sâu PyTorch LSTM (Train PyTorch LSTM).
* Thực hiện đánh giá và so sánh trực tiếp hiệu năng với mô hình cơ sở XGBoost baseline để quyết định mô hình đưa lên môi trường triển khai (deployment).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết kịch bản huấn luyện chính trong file `week4_lstm/train_lstm.py`. <br> - Lập trình vòng lặp huấn luyện tùy chỉnh (custom PyTorch training loop) để kiểm soát chặt chẽ quá trình truyền xuôi và lan truyền ngược. <br> - Khởi tạo đối tượng DataLoader với tham số `sequence_length=7`. | 06/07/2026 | 06/07/2026 | PyTorch Docs |
| 3 | - Thiết lập thuật toán tối ưu hóa Adam optimizer. <br> - Tích hợp bộ lập lịch giảm tốc độ học `ReduceLROnPlateau` scheduler nhằm giúp mô hình hội tụ tốt hơn khi hàm mất mát chững lại. <br> - Viết logic tự động lưu lại phiên bản mô hình tốt nhất (save best model) dựa trên sự cải thiện của chỉ số validation loss. | 07/07/2026 | 07/07/2026 | PyTorch Docs |
| 4 | - Khởi chạy tiến trình huấn luyện mô hình mạng nơ-ron LSTM kéo dài 20 epochs trên môi trường CPU. <br> - Theo dõi, thu thập và ghi log các chỉ số đo lường (Train Loss, Val Loss, Val RMSE, Val MAPE) qua từng chu kỳ huấn luyện. | 08/07/2026 | 08/07/2026 | PyTorch Docs |
| 5 | - Thực hiện đánh giá (Evaluate) mô hình LSTM trên tập dữ liệu kiểm thử (test set). <br> - Áp dụng phép biến đổi nghịch đảo (inverse transform) để khôi phục giá trị dự đoán về đơn vị doanh số bán hàng (Sales) gốc, đảm bảo tính nhất quán khi đo lường sai số. <br> - Phân tích, so sánh trực tiếp các chỉ số RMSE và MAPE giữa LSTM và XGBoost baseline. | 09/07/2026 | 09/07/2026 | Scikit-Learn Docs |
| 6 | - Lưu trữ mô hình tốt nhất vào hệ thống nội bộ tại đường dẫn `week4_lstm/models/lstm_best.pt`. <br> - Tải model artifact (trọng số mô hình đã huấn luyện) lên kho lưu trữ đám mây S3 tại đường dẫn: `s3://aws-internship-hkq-2026/ml-forecasting/models/artifacts/lstm_best.pt`. | 10/07/2026 | 10/07/2026 | Boto3 Docs |

### Kết quả đạt được tuần 4:

* **Tiến trình Huấn luyện LSTM (Training Log):** Quá trình huấn luyện kéo dài 20 epochs ghi nhận sự hội tụ đều đặn. Tại Epoch 20, Train Loss đạt 0.9734, Val Loss giảm xuống 0.9866, Val RMSE đạt 3236.20 và Val MAPE ở mức 35.01%.
* **Thống kê Hiệu năng Test Set (Model Comparison):**
  * Mô hình LSTM: Test RMSE đạt 3044.43, Test MAPE đạt 32.79%.
  * Mô hình XGBoost: Test RMSE đạt **925.28**, Test MAPE đạt **9.92%**.
  * *Quyết định kỹ thuật:* Mô hình XGBoost thể hiện sự vượt trội hoàn toàn (chính xác hơn gấp 3 lần so với LSTM). Do đó, XGBoost chính thức được chọn làm mô hình cốt lõi (production model) để tiến hành deploy ở các tuần tiếp theo.

### Phân tích chuyên sâu nguyên nhân mô hình LSTM kém hiệu quả:
1. **Thiếu chuẩn hóa đặc trưng (Features không được normalize):** Bản chất mạng LSTM cực kỳ nhạy cảm với độ lệch thang đo (scale) của dữ liệu đầu vào. Các cột như `Store` (giá trị 1–1115) hay `CompetitionDistance` (dao động từ vài trăm đến hàng chục nghìn) có sự chênh lệch scale quá lớn, làm nhiễu quá trình học của nơ-ron.
2. **Cửa sổ thời gian hẹp (Sequence length quá ngắn):** Biến `sequence_length` bị ép xuống mức 7. Khoảng thời gian này là quá ngắn, không cung cấp đủ không gian thông tin để LSTM học được các chu kỳ (pattern) mang tính dài hạn.
3. **Sự thiếu hụt biến trễ (Thiếu lag features):** Trong khi XGBoost được cung cấp lượng thông tin trễ lớn (lag 7, 14, 30 ngày), LSTM lại chỉ sử dụng các đặc trưng thô (raw features), dẫn đến sự suy giảm khả năng nắm bắt xu hướng.
4. **Hạn chế phần cứng:** Việc huấn luyện hoàn toàn trên CPU giới hạn nghiêm trọng khả năng tính toán, khiến mô hình không đủ tài nguyên và thời gian để chạy qua nhiều epochs hơn nhằm hội tụ hoàn toàn.

### Ghi chú Kỹ thuật & Giải pháp (Technical Notes):
* Tham số `sequence_length` bắt buộc phải đặt ở mức nhỏ (bằng 7) vì tập validation và test set chỉ chứa vỏn vẹn 1 tháng dữ liệu, nếu đặt lớn hơn sẽ gây lỗi index.
* Ghi nhận sự cố dịch vụ: Việc khởi chạy SageMaker Training Job thất bại do AWS account bị khóa hạn mức (block quota). Giải pháp tình thế (workaround) thành công: Chuyển toàn bộ quá trình huấn luyện xuống máy cục bộ (train local) và thực hiện đẩy artifact lên S3 thủ công.