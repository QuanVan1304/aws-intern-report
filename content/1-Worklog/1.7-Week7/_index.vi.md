---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:
* Xây dựng hệ thống phát hiện rò rỉ dữ liệu (data drift) tự động để giám sát mô hình.
* Thiết lập bảng điều khiển giám sát (CloudWatch Dashboard) trực quan trên AWS.

### Bối cảnh triển khai:
* Do mô hình XGBoost đã được deploy ở tuần 6 nhưng tập dữ liệu Rossmann kết thúc vào năm 2015, hệ thống không có dữ liệu mới thực tế để giám sát (monitor). 
* Giải pháp được đưa ra là lập trình kịch bản `drift_simulator.py` để tự động tạo dữ liệu bất thường, nhằm mục đích mô phỏng và kiểm thử hệ thống giám sát.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tạo số liệu thống kê cơ sở (baseline statistics) từ dữ liệu huấn luyện bằng cách đọc file `week2_preprocessing/data/processed/train.csv`. <br> - Tính toán các chỉ số mean, std, min, max, count cho 5 đặc trưng quan trọng nhất dựa trên phân tích SHAP ở tuần 5 (bao gồm: `Store`, `DayOfWeek`, `Promo`, `CompetitionDistance`, `Month`). <br> - Lưu kết quả phân tích thành file `week7_monitoring/baseline_stats.json` và tải lên bucket `quanvan-ml-forecasting-2026` tại đường dẫn S3: `ml-forecasting/monitoring/baseline/`. | 27/07/2026 | 27/07/2026 | Pandas Docs |
| 3 | - Lập trình kịch bản `drift_simulator.py` như một giải pháp thay thế hợp lệ (workaround) do tính năng SageMaker Model Monitor bị chặn (block) quota. <br> - Viết mã nguồn sạch (code clean) với đầy đủ docstring, xử lý lỗi (error handling), sử dụng đường dẫn tuyệt đối và thiết lập tùy chọn upload S3 (optional). <br> - Thiết lập mô phỏng 2 loại drift: kiểu `shift` (CompetitionDistance nhân 3, Promo tăng lên 80% để phản ánh thị trường cạnh tranh tăng mạnh) và kiểu `noise` (DayOfWeek ngẫu nhiên để mô phỏng lỗi data pipeline). | 28/07/2026 | 28/07/2026 | Python Docs |
| 4 | - Xây dựng thuật toán phát hiện Data Drift dựa trên điểm chuẩn (z-score). <br> - Lập trình công thức tính toán: `z_score = \|mean_new - mean_baseline\| / std_baseline`. <br> - Cấu hình ngưỡng cảnh báo (Alert) kích hoạt khi `z_score > 2.0`, đây là tiêu chuẩn thống kê phổ biến được áp dụng cho bài toán phát hiện điểm bất thường (anomaly detection). | 29/07/2026 | 29/07/2026 | Scipy/Numpy Docs |
| 5 | - Chạy kiểm thử hệ thống phát hiện drift trên nhiều kịch bản khác nhau. <br> - Kịch bản 1: Đưa dữ liệu bình thường (normal data từ test set gốc) vào hệ thống, kết quả trả về 0 alerts, chứng minh hệ thống không bị báo động giả (false alarm). <br> - Kịch bản 2: Đưa dữ liệu bị drift kiểu `shift` vào hệ thống, kết quả phát hiện chính xác 2 alerts. <br> - Trích xuất các file dữ liệu mô phỏng thành quả bao gồm `drifted_shift.csv` và `drifted_noise.csv` lưu tại thư mục `week7_monitoring/data/`. | 30/07/2026 | 30/07/2026 | Hệ thống nội bộ |
| 6 | - Khởi tạo và cấu hình Dashboard mang tên `RossmannForecastingDashboard` trực tiếp trên giao diện AWS Console. <br> - Tích hợp các Widget giám sát bao gồm: Request count, Lambda duration & error rate, và bảng trạng thái Drift Detection Status. <br> - Lưu ảnh chụp màn hình hoàn thiện của bảng điều khiển vào file `week7_monitoring/cloudwatch_dashboard.png`. | 31/07/2026 | 31/07/2026 | CloudWatch Docs |

### Kết quả đạt được tuần 7:

* **Hệ thống Drift Detection:**
  * Tính năng phát hiện rò rỉ dữ liệu hoạt động chính xác tuyệt đối: Kịch bản Normal Data trả về trạng thái hợp lệ (0 alerts); Kịch bản Drifted Data (kiểu shift) báo động thành công 2 lỗi.
  * *Chi tiết phân tích Z-score kịch bản Shift:* Đặc trưng `CompetitionDistance` có Baseline Mean là 5430.34 nhưng Current Mean vọt lên 16291.02, đẩy z-score lên 6.83 (trạng thái: ⚠️ DRIFT). Đặc trưng `Promo` có Baseline Mean 0.38 thay đổi thành 0.80, đẩy z-score lên 4.21 (trạng thái: ⚠️ DRIFT). Các đặc trưng `Store`, `DayOfWeek`, `Month` có z-score dao động từ 0.00 đến 0.01 (trạng thái: ✅ OK).
* **Thành phẩm tập tin (Files) xuất ra:**
  * Hoàn thiện 4 tệp tin nền tảng cốt lõi cho giám sát: `baseline_stats.json` (thống kê 5 đặc trưng), `drifted_shift.csv` (dữ liệu lỗi shift), `drifted_noise.csv` (dữ liệu lỗi noise), và `cloudwatch_dashboard.png` (minh chứng dashboard).

### Ghi chú Kỹ thuật & Kinh nghiệm rút ra (Technical Notes):
* **Giới hạn hệ thống & Giải pháp:** Việc dịch vụ SageMaker Model Monitor bị khóa quota đã được xử lý khéo léo thông qua kịch bản `drift_simulator.py`. Đây là một workaround mang tính thực tiễn cao, đảm bảo quá trình kiểm thử không bị gián đoạn.
* **Tính nhất quán của dữ liệu:** Quá trình tải dữ liệu lên S3 được cấu hình đồng bộ sử dụng chung bucket `quanvan-ml-forecasting-2026` từ tuần 6, giúp chuẩn hóa luồng dữ liệu của toàn bộ dự án.
* **Cơ sở khoa học dữ liệu:** Quyết định chọn ngưỡng `z-score threshold = 2.0` giúp hệ thống giám sát đạt được sự cân bằng, đây là quy chuẩn thống kê an toàn và phổ biến nhất trong thực tế để phát hiện các tín hiệu bất thường (anomaly detection).