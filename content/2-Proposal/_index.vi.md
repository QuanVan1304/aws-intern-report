---
title: "Bản đề xuất"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS MLOps Platform for Sales Forecasting
## Hệ thống Tự động hóa Dự báo Doanh số Chuỗi Cửa hàng Rossmann trên AWS

### 1. Tóm tắt điều hành
Dự án "AWS MLOps Platform for Sales Forecasting" được thiết kế nhằm xây dựng một hệ thống học máy đầu cuối (end-to-end) để dự báo doanh số bán hàng trong tương lai cho chuỗi cửa hàng Rossmann. Thay vì huấn luyện và triển khai mô hình thủ công, nền tảng này tận dụng sức mạnh của AWS SageMaker Pipelines để tự động hóa toàn bộ vòng đời mô hình (CI/CD). Kết hợp với kiến trúc Serverless (AWS Lambda, API Gateway), hệ thống cung cấp một API dự báo thời gian thực với chi phí vận hành tối ưu, độ trễ thấp và khả năng mở rộng linh hoạt.

### 2. Tuyên bố vấn đề
**Vấn đề hiện tại:**
Quy trình dự báo doanh số tại các chuỗi bán lẻ thường phụ thuộc vào việc huấn luyện mô hình thủ công cục bộ. Điều này gây ra sự rời rạc trong quản lý dữ liệu, khó khăn trong việc cập nhật mô hình khi có dữ liệu mới, và không có một hệ thống API chuẩn hóa để tích hợp kết quả dự báo vào các ứng dụng quản lý kinh doanh hiện tại.

**Giải pháp đề xuất:**
Dự án sẽ xây dựng một quy trình MLOps hoàn chỉnh. Dữ liệu lịch sử được tiền xử lý và lưu trữ trên Amazon S3. AWS SageMaker Pipelines sẽ đảm nhiệm việc tự động huấn luyện mô hình XGBoost. Mô hình sau khi đạt chuẩn (Quality Gate) sẽ được triển khai lên SageMaker Endpoint. Cuối cùng, tầng giao tiếp sẽ được xây dựng qua Amazon API Gateway và AWS Lambda, cho phép người dùng hoặc ứng dụng client truy vấn kết quả dự báo thông qua một giao diện web trực quan.

**Lợi ích và hoàn vốn đầu tư (ROI):**
Giải pháp loại bỏ hoàn toàn các thao tác thủ công trong việc cập nhật và triển khai mô hình học máy. Bằng việc sử dụng kiến trúc Serverless cho tầng API và tính toán chi phí theo thời gian sử dụng (pay-as-you-go) cho SageMaker, hệ thống giúp tối ưu hóa chi phí vận hành ở mức thấp nhất. Độ chính xác cao của mô hình giúp doanh nghiệp tối ưu hóa chuỗi cung ứng, giảm lượng hàng tồn kho dư thừa và tăng hiệu quả kinh doanh.

### 3. Kiến trúc giải pháp
Hệ thống sử dụng các dịch vụ cốt lõi của AWS để tạo thành một luồng dữ liệu liên tục từ khâu lưu trữ, huấn luyện đến phục vụ dự báo, tuân thủ chặt chẽ các nguyên tắc bảo mật và hạ tầng của **AWS Well-Architected Framework**.

#### 3.1. Tổng quan các dịch vụ AWS sử dụng
- **Amazon S3:** Lưu trữ dữ liệu huấn luyện, dữ liệu kiểm thử và các Model Artifacts sinh ra từ hệ thống (Mã hóa SSE-KMS).
- **AWS SageMaker Pipelines:** Dịch vụ điều phối tự động vòng đời huấn luyện mô hình (CI).
- **AWS SageMaker Endpoint:** Máy chủ lưu trữ mô hình phục vụ dự báo thời gian thực (CD), hỗ trợ Auto Scaling và Multi-AZ.
- **AWS Lambda:** Hàm Serverless xử lý logic trung gian, áp dụng nguyên tắc đặc quyền tối thiểu (Least-privilege IAM Role).
- **Amazon API Gateway:** Cổng giao tiếp REST API nhận request từ phía Client, kết hợp xác thực Cognito.
- **AWS WAF (Web Application Firewall):** Lớp tường lửa bảo vệ toàn bộ tầng API, thiết lập các quy tắc giới hạn tần suất (Rate Limiting) và chống tấn công mạng.
- **VPC & Private Subnet:** Cách ly các tài nguyên tính toán lõi (Lambda, SageMaker) hoàn toàn khỏi Internet.
- **VPC Endpoint (PrivateLink):** Tạo đường kết nối riêng trên mạng xương sống AWS cho lưu lượng `sagemaker-runtime`.
- **Amazon CloudWatch:** Thu thập Execution Logs, giám sát độ lệch dữ liệu (Data Drift) và cảnh báo hiệu năng hệ thống.

#### 3.2. Sơ đồ Kiến trúc Hạ tầng Mạng & Luồng Phục vụ Dự báo (CD)

![Kiến trúc AWS MLOps Serverless API & Hạ tầng VPC](/images/2-Proposal/aws_cd_architecture.png)

*Hình 3.1: Sơ đồ kiến trúc hạ tầng bảo mật nhiều lớp và luồng phục vụ dự báo thời gian thực trên AWS.*

#### 3.3. Phân tích Chi tiết Các Lớp Kiến trúc (Layered Breakdown)

1. **Lớp Biên & Bảo vệ Vòng ngoài (Public / Edge Layer):**
   - **Client:** Gửi yêu cầu dự báo thời gian thực qua giao thức HTTPS.
   - **AWS WAF:** Chặn đứng các request bất thường, chống tấn công DDoS và áp dụng quy tắc giới hạn tốc độ truy cập.
   - **Amazon API Gateway:** Tiếp nhận request đã qua kiểm duyệt từ WAF, thực hiện xác thực người dùng và định tuyến yêu cầu tới tầng tính toán.

2. **Lớp Hạ tầng Mạng Nội bộ (Private Network & Compute Layer):**
   - **VPC & Private Subnet:** Toàn bộ thành phần xử lý logic và tính toán được cô lập bên trong Mạng ảo riêng độc lập (Virtual Private Cloud).
   - **AWS Lambda:** Tiếp nhận dữ liệu từ API Gateway, tiền xử lý payload request và gửi lệnh gọi dự báo.
   - **VPC Endpoint (`sagemaker-runtime`):** Đảm bảo luồng giao tiếp giữa Lambda và SageMaker Endpoint chạy 100% trên mạng nội bộ của AWS mà không cần đi qua Internet công cộng.
   - **SageMaker Endpoint:** Khai thác sức mạnh mô hình XGBoost đã huấn luyện để đưa ra kết quả dự báo, được cấu hình Multi-AZ đảm bảo tính sẵn sàng cao.

3. **Lớp Lưu trữ & Bất đồng bộ (Storage & Control-Plane Layer):**
   - **Amazon S3:** SageMaker Endpoint chủ động kéo các gói mô hình (`.tar.gz`) đã qua đánh giá chất lượng từ S3 về để nạp vào bộ nhớ.
   - **Amazon CloudWatch:** Tiếp nhận bất đồng bộ các luồng *Execution Log* từ Lambda và *Async Logs / Data Drift* từ SageMaker Endpoint để phục vụ quản trị và phát triển mô hình liên tục.

### 4. Triển khai kỹ thuật
**Các giai đoạn triển khai dự kiến:**
- **Giai đoạn 1 (Tuần 1-2):** Tiền xử lý dữ liệu (Feature Engineering) và thiết lập môi trường lưu trữ S3.
- **Giai đoạn 2 (Tuần 3-4):** Xây dựng mô hình cơ sở (XGBoost baseline) và đánh giá các kiến trúc học sâu (LSTM).
- **Giai đoạn 3 (Tuần 5-6):** Xây dựng SageMaker Pipelines và đóng gói Endpoint, phát triển Serverless API.
- **Giai đoạn 4 (Tuần 7-8):** Xây dựng hệ thống giám sát (CloudWatch, Data Drift) và dọn dẹp tài nguyên (Housekeeping).

**Yêu cầu kỹ thuật:**
- Ngôn ngữ lập trình: Python 3.x.
- Thư viện cốt lõi: `boto3`, `sagemaker==2.257.5`, `xgboost`, `pandas`, `scikit-learn`.
- Giao diện (Demo UI): HTML, CSS, JavaScript (Fetch API).

### 5. Lộ trình & Mốc triển khai
- **Thời gian thực hiện:** 7 tuần (Từ 15/06/2026 đến 31/07/2026).
- **Cột mốc 1:** Hoàn thiện tiền xử lý và có được mô hình Baseline (Cuối tuần 3).
- **Cột mốc 2:** Luồng Pipeline chạy thành công trên AWS (Cuối tuần 5).
- **Cột mốc 3:** Tích hợp thành công API và giao diện Web (Cuối tuần 6).
- **Cột mốc 4:** Bàn giao hệ thống giám sát và hoàn tất thực tập (Cuối tuần 8).

### 6. Ước tính ngân sách
Dự án được tối ưu để vận hành trong mức ngân sách tiết kiệm nhất thông qua AWS Free Tier và cơ chế xóa tài nguyên tự động.
- **Amazon S3:** ~0.10 USD/tháng cho việc lưu trữ vài trăm MB dữ liệu.
- **SageMaker Training Job (`ml.m5.large`):** Trả phí theo phút, ước tính ~1-2 USD cho toàn bộ quá trình phát triển.
- **SageMaker Endpoint (`ml.t2.medium`):** ~0.05 USD/giờ (Chỉ bật khi demo và kiểm thử, sử dụng script cleanup để hủy).
- **API Gateway & Lambda:** Nằm hoàn toàn trong Free Tier (dưới 1 triệu request/tháng).
*Tổng chi phí ước tính trong suốt kỳ thực tập: Dưới 5.00 USD.*

### 7. Đánh giá rủi ro
- **Rủi ro Quota AWS (Cao):** Tài khoản mới có thể bị giới hạn Training Job quota = 0. 
  *Chiến lược:* Gửi yêu cầu tăng quota qua Service Quotas ngay từ tuần đầu tiên.
- **Rủi ro xung đột thư viện SDK (Trung bình):** Cập nhật SDK mới có thể gây lỗi Pipeline. 
  *Chiến lược:* Ghim cứng phiên bản thư viện (Dependency Pinning) trong `requirements.txt`.
- **Rủi ro vượt ngân sách (Thấp):** Quên tắt Endpoint dẫn đến trừ tiền theo giờ. 
  *Chiến lược:* Viết kịch bản `cleanup.py` chạy tự động để hủy mọi tài nguyên đang bật.

### 8. Kết quả kỳ vọng
- **Kỹ thuật:** Xây dựng thành công một hệ thống MLOps đạt chuẩn doanh nghiệp, tích hợp liền mạch giữa hạ tầng Model Training và Serverless API.
- **Độ chính xác:** Mô hình đạt mức sai số (MAPE) dưới 15%, đủ độ tin cậy để dự đoán kinh doanh.
- **Giá trị dài hạn:** Kiến trúc có thể được tái sử dụng làm tài liệu mẫu (template) cho các bài toán dự báo chuỗi thời gian khác trong tương lai.