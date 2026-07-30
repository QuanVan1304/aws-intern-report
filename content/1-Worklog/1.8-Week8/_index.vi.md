---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
* Tự động hóa toàn diện vòng đời huấn luyện và triển khai mô hình bằng AWS SageMaker Pipelines theo chuẩn kiến trúc thực tiễn (Best Practices).
* Cấp phát, vận hành và dọn dẹp triệt để tài nguyên điện toán đám mây vào cuối kỳ thực tập.

### Bối cảnh triển khai:
* Account cá nhân ban đầu bị giới hạn quota SageMaker Training Job bằng 0, dẫn đến mọi nỗ lực huấn luyện đều bị từ chối với lỗi `ResourceLimitExceeded`. 
* Sau khi gửi yêu cầu qua Service Quotas API/Console, AWS đã phê duyệt tăng quota cho instance `ml.m5.large` từ 0 lên 1. Bước ngoặt này cho phép dự án từ bỏ mô phỏng local để chính thức chạy hạ tầng Pipeline thực tế trên AWS.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Giải quyết triệt để lỗi `ModuleNotFoundError`/`ImportError` liên quan đến `sagemaker.workflow`. <br> - Nguyên nhân do lệnh `pip install sagemaker` tự động kéo bản mới nhất, gây xung đột giữa `sagemaker` và `sagemaker-core`. <br> - Khắc phục bằng cách ghim cứng phiên bản `sagemaker==2.257.5` vào file `requirements.txt` để đảm bảo tính ổn định cho toàn hệ thống. | 27/07/2026 | 27/07/2026 | SageMaker SDK Docs |
| 3 | - Xây dựng và gỡ lỗi 3 vấn đề cốt lõi của SageMaker Pipeline. <br> - Lỗi 1: Khắc phục `ClientError: 404 HeadObject` bằng cách bổ sung `HyperParameters` trỏ đúng vị trí file `sourcedir.tar.gz`. <br> - Lỗi 2: Xử lý giới hạn không cho phép override `MetricDefinitions` của AWS bằng cách tháo gỡ bước Condition khỏi Pipeline, chuyển việc kiểm tra mô hình thành bước hậu kiểm độc lập (phân tách CI/CD). <br> - Lỗi 3: Khắc phục tình trạng sinh rác bằng cách cấu hình **Pipeline Tĩnh** (`Rossmann-Sales-Pipeline`) kết hợp lệnh `pipeline.upsert()`, loại bỏ timestamp khỏi tên pipeline. | 28/07/2026 | 28/07/2026 | Boto3/AWS Docs |
| 4 | - Nâng cấp script `pipeline_definition.py` để tự động đóng gói mã nguồn (`train_xgboost_sm.py` & `feature_utils.py`) thành `sourcedir.tar.gz` và upload lên S3 ở mỗi lần chạy, giúp Pipeline tự chủ hoàn toàn. <br> - Khởi chạy thành công Pipeline, sinh ra Training Job mang ID `pipelines-8pk6dxn4cfvl-Rossmann-Model-Train-bPVYsMzUlc` cùng Model Artifact chuẩn trên S3. <br> - Triển khai Endpoint `rossmann-forecasting-endpoint` và chạy Quality Gate (`build_real_features.py`) bằng dữ liệu lịch sử Store 1 (ngày 15/06/2015), đạt MAPE 4.75% (đạt chuẩn PASS). | 29/07/2026 | 29/07/2026 | AWS MLOps Architecture |
| 5 | - Thiết lập môi trường ảo biệt lập (`sagemaker-v3-env`) để thử nghiệm SageMaker Python SDK v3. <br> - Khởi tạo thành công `ModelTrainer` và Pipeline, nhưng xác nhận giới hạn `MetricDefinitions` vẫn bị AWS Service từ chối (lỗi xuất phát từ tầng dịch vụ, không phải từ version SDK). <br> - Phát hiện lỗi CRLF line-ending (`sm_train.sh: line 1: $'\r': command not found`) do SDK tự sinh wrapper script không tương thích hoàn toàn trên Windows. | 30/07/2026 | 30/07/2026 | SageMaker v3 Docs |
| 6 | - Thực hiện Housekeeping (Dọn dẹp hệ thống) toàn diện trước khi kết thúc kỳ thực tập. <br> - Xóa bỏ thư mục `sagemaker-v3-env/` khỏi Git tracking và cập nhật vào `.gitignore`. <br> - Xóa Pipeline thử nghiệm `Rossmann-Sales-Pipeline-V3` và dùng AWS CLI rà soát thủ công để tiêu hủy các Model mồ côi (do quy ước đổi tên làm script tự động bị lọt lưới). <br> - Xóa Endpoint cuối cùng, xác nhận toàn bộ tài nguyên tính phí đã được giải phóng hoàn toàn. | 31/07/2026 | 31/07/2026 | AWS CLI Docs |

### Kết quả đạt được tuần 8:
* **Hệ thống Pipeline & MLOps:**
  * Kiến trúc CI/CD vận hành trơn tru: Pipeline (CI) đã tự động hóa trọn vẹn việc huấn luyện, tạo ra mô hình thực tế, trong khi khâu triển khai (CD) được kiểm soát nghiêm ngặt bằng Quality Gate độc lập bên ngoài.
  * Độ chính xác của mô hình sinh ra từ Pipeline được kiểm định qua dữ liệu thực tế đạt mức sai số cực thấp (MAPE 4.75%), nằm sâu trong ngưỡng an toàn 15.0% của dự án.
* **Thành quả Quản trị Hạ tầng:**
  * Toàn bộ tài nguyên rác và tài nguyên phát sinh chi phí theo giờ (Endpoint) đã được xóa bỏ hoàn toàn. Chỉ duy nhất cấu trúc Pipeline tĩnh `Rossmann-Sales-Pipeline` được giữ lại trên AWS làm minh chứng kiến trúc hoàn chỉnh cho dự án.

---

### Ghi chú Kỹ thuật & Kinh nghiệm rút ra (Lessons Learned):

Phần này tổng hợp những kinh nghiệm thực chiến đắt giá nhất trong suốt quá trình xây dựng hệ thống tự động hóa và xử lý sự cố hạ tầng trên AWS:

1. **Quản trị Phiên bản Thư viện (Dependency Management) nghiêm ngặt:** 
   Việc sử dụng lệnh cài đặt lỏng lẻo (`pip install sagemaker`) sẽ khiến môi trường tự động kéo về các bản cập nhật mới nhất nhưng chưa tương thích hoàn toàn, dẫn đến xung đột nội bộ nghiêm trọng. Quyết định ghim cứng phiên bản đã kiểm chứng (`sagemaker==2.257.5`) là chiến lược thực dụng, đảm bảo tính ổn định tuyệt đối cho hệ thống production thay vì mạo hiểm chạy theo công nghệ mới nhất.

2. **Tư duy MLOps: Tách biệt Định nghĩa và Thực thi:**
   Một sai lầm phổ biến là gắn trực tiếp `datetime.now()` vào tên Pipeline, khiến mỗi lần chạy sẽ rác ra một object mới trên Cloud. Áp dụng tư duy chuẩn bằng cách tách bạch "Định nghĩa luồng tĩnh" (tên cố định, dùng `upsert()`) và "Thực thi động" (Execution ID chứa timestamp) giúp tài nguyên AWS luôn gọn gàng và dễ dàng quản lý.

3. **Phân định Giới hạn Dịch vụ (AWS Service) vs Công cụ (SDK):** 
   Khi đối mặt với lỗi `"You can't override the metric definitions..."`, việc thử nghiệm chéo trên cả SDK v2 và v3 đã giúp xác nhận ranh giới kỹ thuật: lỗi nằm ở bản chất container thuật toán (built-in algorithm image) bị giới hạn bởi hệ thống backend của AWS, hoàn toàn độc lập với phiên bản SDK đang sử dụng. Khi một lỗi lặp lại bất kể thay đổi công cụ, cần nghi ngờ ngay giới hạn của bản thân tầng dịch vụ đám mây.

4. **Tách biệt CI (Huấn luyện) và CD (Triển khai có kiểm soát):**
   Pipeline của SageMaker chỉ nên giới hạn ở trách nhiệm tự động hóa huấn luyện (CI). Việc đưa mô hình vào phục vụ thực tế (Endpoint) tuyệt đối phải được tách riêng thành một khâu độc lập và phải đi qua "chốt chặn" kiểm định (Quality Gate). Điều này triệt tiêu hoàn toàn rủi ro hệ thống tự động deploy một mô hình chưa được xác minh chất lượng ra môi trường thực tế.

5. **Đảm bảo tính Tự chủ (Self-containment) cho Pipeline:**
   Ở những phiên bản đầu, Pipeline bị phụ thuộc vào một file `sourcedir.tar.gz` lưu trữ từ các lần chạy thủ công cũ, gây ra rủi ro hỏng hóc cao nếu file này bị xóa vô tình. Việc nâng cấp script để tự động đóng gói và tải mã nguồn hiện hành lên S3 ngay trước mỗi lần khởi chạy đã cắt đứt mọi phụ thuộc ngoại lai, đưa Pipeline trở thành một thực thể độc lập và tự chủ 100%.

6. **Rủi ro tự động hóa & Tầm quan trọng của đối soát CLI:**
   Do có sự thay đổi quy ước đặt tên mô hình giữa các giai đoạn (từ `rossmann-xgboost-*` sang `rossmann-pipeline-xgboost-*`), kịch bản dọn dẹp tự động (`cleanup.py`) đã bị lọt lưới và bỏ sót nhiều tài nguyên. Bài học rút ra là không bao giờ tin tưởng mù quáng vào script tự động; luôn phải thực hiện rà soát chéo (cross-verification) định kỳ bằng AWS CLI để bảo vệ ngân sách dự án.

7. **Cô lập Môi trường Thử nghiệm (Environment Isolation):**
   Mọi thử nghiệm công nghệ mới (như việc test SDK v3) bắt buộc phải diễn ra trong môi trường ảo biệt lập (`sagemaker-v3-env`). Quan trọng hơn, các môi trường nháp này phải được cấu hình loại trừ khỏi version control (cập nhật `.gitignore`) và dọn sạch khỏi AWS ngay khi hoàn tất để tránh làm nhiễu loạn kiến trúc chính thức khi bàn giao dự án.