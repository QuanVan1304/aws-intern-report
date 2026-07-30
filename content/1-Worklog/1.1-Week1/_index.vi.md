---
title: "Worklog Tuần 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:
* Setup môi trường AWS và khởi tạo repo.
* Kết nối thành công với AWS.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tạo IAM Role trên AWS Management Console. <br> &emsp; + Thiết lập Trusted entity: AWS Service → SageMaker Execution. <br> &emsp; + Gắn các policies: `AmazonSageMakerFullAccess`, `AmazonS3FullAccess`, `CloudWatchFullAccess`. <br> - Tạo S3 bucket trên AWS Management Console để lưu trữ tài nguyên. | 15/06/2026 | 15/06/2026 | AWS Documentation |
| 3 | - Cài đặt AWS CLI v2. <br> - Cấu hình xác thực SSO authentication (sử dụng lệnh `aws login`). <br> - Tạo Python 3.11 virtual environment (venv) để chuẩn bị môi trường lập trình. | 16/06/2026 | 16/06/2026 | AWS Documentation |
| 4 | - Khởi tạo GitHub repository mang tên `ml-forecasting-aws` (chế độ private). <br> - Xây dựng cấu trúc thư mục theo tuần: từ `week1_setup/` đến `week8_pipeline/`. <br> - Cấu hình file `.gitignore` (đảm bảo loại bỏ `.env`, `*.csv`, `__pycache__/`...). <br> - Tạo file `requirements.txt` với đầy đủ các dependencies cần thiết. | 17/06/2026 | 17/06/2026 | GitHub Docs |
| 5 | - Viết script `config.py`. <br> &emsp; + Sử dụng thư viện `python-dotenv` để load credentials bảo mật từ file `.env`. <br> &emsp; + Định nghĩa S3 paths, ARNs, và các constants dùng chung cho toàn bộ project. | 18/06/2026 | 18/06/2026 | Boto3 Docs |
| 6 | - Viết script `verify_setup.py` để tự động xác minh môi trường. <br> - Lập trình 5 bước kiểm tra (checks) cốt lõi: <br> &emsp; + Check 1: AWS credentials (STS). <br> &emsp; + Check 2: S3 bucket accessible. <br> &emsp; + Check 3: IAM Role exists. <br> &emsp; + Check 4: SageMaker API reachable. <br> &emsp; + Check 5: Region correct. | 19/06/2026 | 19/06/2026 | Boto3 Docs |

### Kết quả đạt được tuần 1:

* **IAM Role:** Role `SageMaker-ExecutionRole-hkq` đã được tạo thành công (created).
* **S3 Bucket:** Bucket `aws-internship-hkq-2026` đã được tạo và xác nhận có thể truy cập (accessible).
* **GitHub Repo:** Đã khởi tạo thành công repository `ml-forecasting-aws` (initialized).
* **Môi trường Python:** Python 3.11 virtual environment (venv) đã được kích hoạt (active).
* **Quản lý cấu hình:** Hoàn thành file `config.py` với đầy đủ định nghĩa về S3 paths, ARNs và hằng số.
* **Kiểm tra môi trường:** Script `verify_setup.py` chạy thành công, vượt qua toàn bộ 5/5 bài kiểm tra (pass). Thông tin chi tiết từ output:
  * *AWS Account:* 119505195050.
  * *User/Role:* arn:aws:iam::119505195050:root.
  * *S3 Bucket:* s3://aws-internship-hkq-2026 — accessible.
  * *IAM Role:* SageMaker-ExecutionRole-hkq — exists.
  * *SageMaker API:* OK — region ap-southeast-1.

### Ghi chú kỹ thuật (Lessons Learned):
* **Xác thực SSO:** Do account công ty dùng SSO, phiên đăng nhập sẽ hết hạn định kỳ. Giải pháp là cần chạy lại lệnh `aws login` mỗi khi session hết hạn.
* **Sự thay đổi của SageMaker SDK:** Phát hiện ra rằng thư viện `sagemaker` SDK 3.x đã thay đổi hoàn toàn cấu trúc. Để giải quyết, nhóm đã quyết định dùng trực tiếp `boto3.client("sagemaker")` thay vì sử dụng `sagemaker.Session()` nhằm tránh các lỗi phát sinh.
* **Tương thích phiên bản Python:** Ghi nhận lỗi Python 3.13 không tương thích với thư viện `numpy` và `sagemaker`. Vấn đề này đã được xử lý triệt để bằng cách buộc phải sử dụng môi trường Python 3.11 venv.