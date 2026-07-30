---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# TỪ LOCAL NOTEBOOK ĐẾN MLOPS THỰC CHIẾN: HÀNH TRÌNH "THUẦN PHỤC" AWS SAGEMAKER PIPELINES

Khi mới bước chân vào lĩnh vực Machine Learning, Jupyter Notebook cục bộ luôn là "vùng an toàn" quen thuộc để xử lý dữ liệu và huấn luyện mô hình. Tuy nhiên, bài toán thực tế của doanh nghiệp lại khắc nghiệt hơn nhiều: *Làm thế nào để tự động hóa toàn bộ quy trình, dễ dàng mở rộng (scale) và đưa mô hình vào phục vụ các ứng dụng thực tế một cách ổn định?*

Để giải quyết bài toán này, mình đã quyết định đưa dự án dự báo doanh số (sử dụng tập dữ liệu Rossmann Store Sales) lên đám mây AWS. Dưới đây là bức tranh toàn cảnh về kiến trúc hệ thống và những "vết xước" thực chiến đắt giá.

### 1. Phân rã hệ thống với Kiến trúc MLOps 3 Lớp

Thay vì nhồi nhét mọi thứ vào một nơi, hệ thống được thiết kế phân tách rõ ràng thành 3 phân hệ độc lập:

*   **Lớp 1 - Data Lake & Lựa chọn Mô hình (Baseline Modeling):** Amazon S3 được sử dụng làm kho lưu trữ dữ liệu trung tâm. Quá trình thử nghiệm cho thấy thuật toán XGBoost (sai số MAPE 9.92%) đã đánh bại hoàn toàn mô hình Deep Learning phức tạp PyTorch LSTM (MAPE 32.79%). Sự tinh gọn và hiệu quả đã giúp XGBoost được chọn làm mô hình lõi.
*   **Lớp 2 - Tự động hóa Huấn luyện (CI với SageMaker Pipelines):** Quá trình huấn luyện thủ công được thay thế bằng luồng `Rossmann-Sales-Pipeline`. Bất kỳ thay đổi nào trong code cũng sẽ tự động kích hoạt Pipeline: từ việc cấp phát máy chủ, kéo dữ liệu từ S3, huấn luyện XGBoost, cho đến việc đóng gói Model Artifact và lưu trữ ngược lại S3 một cách khép kín.
*   **Lớp 3 - Phục vụ Mô hình Thời gian thực (CD & Serving):** Để các ứng dụng bên ngoài có thể tương tác với SageMaker Endpoint, mình thiết lập một lớp Serverless REST API trung gian sử dụng **Amazon API Gateway** kết hợp **AWS Lambda**. Lambda làm nhiệm vụ phiên dịch các HTTP request thành lệnh `invoke_endpoint` và trả về kết quả dự báo. Nhờ kiến trúc này, sai số trên tập dữ liệu thực tế (Quality Gate) được kéo giảm xuống mức ấn tượng: **4.75%**.

### 2. 3 Bài học "xương máu" khi triển khai Cloud

Việc đưa hệ thống lên Cloud không bao giờ là trải nghiệm dễ dàng. Dưới đây là 3 cạm bẫy lớn nhất mà mình đã vượt qua:

*   **Rào cản Service Quotas:** Hệ thống từng bị tê liệt ngay từ bước đầu tiên do AWS mặc định giới hạn quota SageMaker Training Jobs bằng 0. Thay vì hoang mang sửa code, việc kiểm tra Quota Dashboard và mở ticket yêu cầu AWS Support cấp phép (instance `ml.m5.large`) là thao tác sống còn.
*   **Cơn ác mộng xung đột thư viện (Dependency Hell):** Lỗi `ModuleNotFoundError` từng phá hỏng toàn bộ pipeline do sự chênh lệch phiên bản SDK. Bài học rút ra là tuyệt đối không cài đặt thư viện một cách lỏng lẻo. Lệnh chốt cứng phiên bản `sagemaker==2.257.5` chính là "chiếc phao" cứu sinh của dự án.
*   **Quản trị Chi phí (Cost Optimization):** Trái ngược với AWS Lambda, SageMaker Endpoint sẽ tính phí theo giờ ngay cả khi không có request nào. Do đó, việc viết thêm script `cleanup.py` để tự động dọn dẹp Endpoint sau mỗi lần test là kỹ năng bắt buộc để tránh "thủng ví".

### Lời kết

Hành trình dịch chuyển từ Local Notebook lên một hệ thống MLOps hoàn chỉnh đòi hỏi sự thay đổi lớn về tư duy: từ việc viết code chạy được sang việc thiết kế hạ tầng, tối ưu chi phí và dự phòng lỗi. Thành quả thu lại là một hệ thống tự động, mạnh mẽ và hoàn toàn đáp ứng được các tiêu chuẩn khắt khe trong môi trường doanh nghiệp.

[*...\[Link bài đăng trên AWS Study Group\]...*](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2227791551319138/)