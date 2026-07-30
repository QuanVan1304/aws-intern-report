---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---
### Mục tiêu tuần 6:
* Triển khai mô hình dự báo XGBoost lên môi trường thực tế thông qua SageMaker Endpoint.
* Xây dựng kiến trúc Serverless, đóng gói (expose) REST API thông qua AWS Lambda và API Gateway.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết kịch bản tự động hóa truy xuất file mô hình nguyên thủy (`model.tar.gz`) từ S3 dựa trên ID của Training Job do SageMaker Pipeline sinh ra. <br> - Lập trình luồng "Tái đóng gói động" (Dynamic Repackaging): tải model về, giải nén, tiêm (inject) kịch bản `inference.py`, đóng gói lại thành `model_from_pipeline.tar.gz` và đẩy ngược lên S3. | 20/07/2026 | 20/07/2026 | Boto3 Docs |
| 3 | - Xây dựng script `deploy_endpoint.py` (Infrastructure as Code bằng Boto3) để tự động hóa 4 bước triển khai Endpoint. <br> - Viết logic tự động kiểm tra trạng thái Endpoint: thực hiện cập nhật không gián đoạn (Zero-downtime update) nếu đã tồn tại, hoặc tạo mới nếu chưa có. | 21/07/2026 | 21/07/2026 | SageMaker Docs |
| 4 | - Xử lý loạt sự cố kỹ thuật (Troubleshooting) phát sinh trong quá trình deploy bao gồm: lỗi phân quyền ECR, lỗi xung đột phiên bản thư viện và lỗi tràn số do sai lệch logic suy luận. | 22/07/2026 | 22/07/2026 | AWS Docs |
| 5 | - Khởi chạy quy trình Kiểm định Chất lượng (Quality Gate). <br> - Chạy kịch bản `invoke_test.py` (Smoke Test) với payload giả lập gồm 22 features để kiểm tra giao tiếp API. <br> - Chạy kịch bản `build_real_features.py` để validate độ chính xác của dự đoán đối với Cửa hàng số 1 (ngày 2015-06-15). | 23/07/2026 | 23/07/2026 | Boto3 Docs |
| 6 | - Thiết lập kiến trúc Serverless API (Client ➔ API Gateway ➔ AWS Lambda ➔ SageMaker Endpoint ➔ S3). <br> - Thực thi kịch bản `cleanup.py` và rà soát chéo bằng AWS CLI để xóa bỏ toàn bộ tài nguyên (Cost Optimization), ngăn chặn phát sinh cước phí ngoài ý muốn. | 24/07/2026 | 24/07/2026 | AWS Architecture |

### Kết quả đạt được tuần 6:
* **Trạng thái Triển khai:** Endpoint (`rossmann-forecasting-endpoint`) đã chuyển sang trạng thái *InService* thành công thông qua file cấu hình `rossmann-config-1785086724`.
* **Thống kê Quality Gate:**
  * **Smoke Test:** API phản hồi định dạng chuẩn JSON thành công (`{"predicted_sales": [5267.64]}`).
  * **Validation thực tế:** Đối với Cửa hàng số 1, doanh số thực tế là 5518.00; hệ thống API dự báo đạt 5780.13. Mức độ sai lệch chỉ ở mức **4.75%**, vượt xuất sắc qua bài kiểm tra tiêu chuẩn (ngưỡng cho phép là 15.0%).
* **Kiến trúc hoàn thiện:** Luồng gọi API Gateway ➔ Lambda ➔ Endpoint hoạt động trơn tru với HTTP 200 OK.

### Ghi chú Kỹ thuật & Giải pháp chuyên sâu (Troubleshooting & Workarounds):

**1. Khắc phục lỗi luồng dữ liệu suy luận:**
* *Vấn đề (Overflow/Dự đoán vô nghĩa):* Hàm suy luận trả về kết quả tràn số. Nguyên nhân gốc rễ là do script `inference.py` gọi hàm `np.expm1()` lên kết quả đầu ra, nhưng thực tế mô hình XGBoost đang được sử dụng lại được huấn luyện trực tiếp trên biến Sales gốc chứ không thông qua biến đổi `log1p`.
* *Giải pháp:* Can thiệp vào `predict_fn`, xóa bỏ dòng `np.expm1()` và cấu hình để API trả thẳng mảng giá trị `model.predict(X)` ra ngoài.

**2. Khắc phục sự cố Container & Cấu hình môi trường:**
* *Lỗi ValidationException (Phân quyền ECR):* Khi tạo model, hệ thống báo lỗi ECR do Container Image URI bị hardcode cứng Account ID của region `us-east-1`. *Giải pháp:* Tra cứu tài liệu và trỏ chính xác về ECR của `ap-southeast-1` (`121021644041.dkr.ecr.ap-southeast-1.amazonaws.com/sagemaker-xgboost:1.7-1`).
* *Lỗi ModelError HTTP 500:* API sập khi gọi. Nguyên nhân do mô hình được train bằng version XGBoost mới, nhưng container suy luận chỉ hỗ trợ bản `1.7-1`. *Giải pháp:* Đồng bộ chặt chẽ phiên bản thư viện giữa môi trường huấn luyện (training job) và môi trường phục vụ (serving container).

**3. Tối ưu chi phí (Cost Optimization) và Lỗ hổng Cleanup:**
* *Ngăn chặn Data Leakage:* Trong quá trình test thực tế (`build_real_features.py`), logic trích xuất tính toán các độ trễ (`sales_lag_*`, `rolling_*`) được ép buộc phải dùng dữ liệu lịch sử *trước* ngày dự đoán để chặn hoàn toàn rò rỉ dữ liệu.
* *Fix lỗi script dọn dẹp:* Script `cleanup.py` sử dụng bộ lọc `NameContains="rossmann-xgboost"`, tuy nhiên điều này vô tình bỏ sót các Model sinh ra từ Pipeline mang tên `rossmann-pipeline-xgboost-*` (do chuỗi con không liền mạch).
* *Giải pháp:* Phát hiện kịp thời tài nguyên rác, tiến hành rà soát và xóa thủ công toàn bộ Model/Endpoint Config/Endpoint qua AWS CLI (`list-models`, `list-endpoints`). Cuối cùng xác nhận cả 3 danh sách đã hoàn toàn trống, đảm bảo dự án kết thúc không tốn thêm bất kỳ một đồng cước phí nào (0$ leakage).