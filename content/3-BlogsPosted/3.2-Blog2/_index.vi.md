---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# DEEP LEARNING CÓ LUÔN TỐT HƠN MACHINE LEARNING TRUYỀN THỐNG? BÀI HỌC TỐI ƯU HIỆU NĂNG VÀ CHI PHÍ TRÊN AWS SAGEMAKER

Khi bắt tay vào các dự án AI/ML (như dự báo chuỗi thời gian - time series forecasting), nhiều người thường mang tư duy: *Sử dụng các mô hình Deep Learning phức tạp (như mạng LSTM hay Transformer) chắc chắn sẽ đem lại kết quả tốt hơn so với Machine Learning truyền thống*. Tuy nhiên, thực tế triển khai trên môi trường Cloud như AWS lại chứng minh một điều ngược lại: "Dao mổ trâu" chưa chắc đã giải quyết tốt bài toán "giết gà".

Dưới đây là bài học thực chiến của mình về việc cân bằng giữa độ phức tạp của mô hình, hiệu năng và chi phí hạ tầng trên AWS SageMaker.

### 1. Cuộc đối đầu trên dữ liệu bảng (Tabular Data)
Trong dự án dự báo doanh số E-commerce, mình đã thử nghiệm cả hai phương pháp:
*   **Deep Learning (LSTM):** Mặc dù được kỳ vọng cao, việc huấn luyện trên CPU diễn ra rất chậm, đòi hỏi cấu hình `sequence_length` phức tạp và kết quả trả về khá tệ với mức sai số (MAPE) lên tới **~32.79%**.
*   **Machine Learning truyền thống (XGBoost):** Chạy cực kỳ mượt mà trên các instance CPU cơ bản, hội tụ nhanh và đạt mức sai số (MAPE) chỉ còn **~9.92%**. 

Nguyên nhân cốt lõi nằm ở **Feature Engineering** và **Độ nhạy cảm với dữ liệu**. XGBoost tận dụng rất tốt các biến trễ (Lag) và trung bình trượt (Rolling mean) do mình tạo ra, đồng thời không bị nhiễu bởi sự chênh lệch scale của dữ liệu thô (như khoảng cách đối thủ hay mã cửa hàng). Ngược lại, LSTM cực kỳ nhạy cảm với scale và sẽ học ra "rác" nếu dữ liệu chưa được chuẩn hóa kỹ lưỡng.

### 2. Bài toán kinh tế trên AWS SageMaker
Trên AWS SageMaker, chi phí của một Training Job được tính bằng: **Đơn giá Instance × Thời gian hội tụ**.
*   **XGBoost:** Chỉ cần dùng CPU rẻ, hội tụ nhanh sau vài chục epochs.
*   **LSTM:** Thường đòi hỏi GPU đắt tiền và tốn hàng trăm epochs để hội tụ. 

Hơn thế nữa, khi triển khai (Deploy) phục vụ suy luận (Inference), một mô hình nhẹ như XGBoost chỉ cần instance `ml.t2.medium` giá rẻ là đủ chạy mượt mà 24/7. Việc cố chấp sử dụng một mô hình Deep Learning nặng nề có thể khiến chi phí Cloud tăng phi mã mà không mang lại giá trị tương xứng.

### 3. Quản lý vòng đời mô hình chuyên nghiệp
Thay vì theo dõi các chỉ số một cách thủ công, mình đã sử dụng **SageMaker Experiments** để chạy song song nhiều cấu hình (XGBoost vs LSTM), tự động tạo bảng so sánh các metrics (RMSE, MAPE, Thời gian huấn luyện). Sau khi xác định được "nhà vô địch" là XGBoost, mô hình tốt nhất được đẩy vào **SageMaker Model Registry** để quản lý phiên bản chuyên nghiệp trước khi đưa lên Production.

### 4. Những cạm bẫy dễ mắc phải (MLOps Anti-patterns)
Qua dự án này, mình đúc kết được một số lỗi kinh điển cần tránh:
1.  **Chạy theo Hype (Hype-driven development):** Cố nhét Deep Learning vào dự án trong khi dữ liệu chỉ có vài ngàn dòng CSV. Deep Learning chỉ thực sự tỏa sáng ở dữ liệu phi cấu trúc (Unstructured Data) như CV, NLP hay Big Data.
2.  **Bỏ qua Feature Engineering:** Phụ thuộc hoàn toàn vào kiến trúc mạng nơ-ron mà quên mất việc làm sạch và biến đổi dữ liệu mới là "vua".
3.  **Bỏ quên Inference Cost:** Thiết kế mô hình cực xịn nặng 2GB, dẫn đến việc phải thuê Endpoint dung lượng RAM khổng lồ chạy 24/7 gây lãng phí.

### Lời kết
Sức mạnh của AI/ML không chỉ nằm ở thuật toán phức tạp, mà là sự tương thích với dữ liệu và khả năng tối ưu trên cơ sở hạ tầng. Câu hỏi đúng dành cho một Kỹ sư AI/ML trên Cloud không phải là *"Thuật toán nào phức tạp nhất?"*, mà phải là: **"Mô hình nào mang lại sự cân bằng tốt nhất giữa độ chính xác, thời gian huấn luyện và chi phí triển khai?"**.


[*...\[Link bài đăng trên AWS Study Group\]...*](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2229015587863401/)