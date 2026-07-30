---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---
### Mục tiêu tuần 2:
* Tiền xử lý dữ liệu (Data preprocessing) — Exploratory Data Analysis (EDA), làm sạch dữ liệu, phân chia (split), chuẩn hóa (scale) và tải lên S3.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tải Rossmann Store Sales dataset từ Kaggle bao gồm `train.csv` (~38MB) và `store.csv`, lưu vào thư mục `week2_preprocessing/data/`. <br> - Upload raw data lên đường dẫn `s3://aws-internship-hkq-2026/ml-forecasting/data/raw/`. <br> - Tải `train.csv` qua `boto3` và `store.csv` qua AWS CLI. | 22/06/2026 | 22/06/2026 | Kaggle, Boto3 Docs |
| 3 | - Viết và chạy file notebook EDA (`eda.ipynb`). <br> - Phân tích chi tiết shape, data types, và missing values của tập dữ liệu. <br> - Phân tích phân phối của biến Sales và sự biến động của Sales theo thời gian, ngày trong tuần, tháng. <br> - Đánh giá ảnh hưởng của các biến Promo và StateHoliday đến doanh số. | 23/06/2026 | 23/06/2026 | Pandas/Seaborn Docs |
| 4 | - Viết script `preprocessing.py` (Phần 1). <br> - Merge `train.csv` và `store.csv` dựa theo ID `Store`. <br> - Loại bỏ các bản ghi của cửa hàng đóng cửa (`Open=0`) và các ngày không có doanh thu (`Sales=0`). <br> - Điền giá trị thiếu (fill missing values) cho các cột Competition và Promo2. <br> - Encode các categorical features (`StateHoliday`, `StoreType`, `Assortment`). | 24/06/2026 | 24/06/2026 | Pandas Docs |
| 5 | - Viết script `preprocessing.py` (Phần 2). <br> - Thêm các calendar features: `Year`, `Month`, `Day`, `WeekOfYear`, `IsWeekend`. <br> - Thực hiện time-series split theo đúng thứ tự thời gian. <br> - Log-transform biến mục tiêu và fit hàm StandardScaler (chỉ fit trên tập train). | 25/06/2026 | 25/06/2026 | Scikit-Learn Docs |
| 6 | - Upload processed data lên đường dẫn `s3://aws-internship-hkq-2026/ml-forecasting/data/processed/`. <br> - Đảm bảo đẩy đủ 4 files: `train.csv` (103MB), `val.csv` (3.7MB), `test.csv` (3.9MB), và `scaler.pkl` (538B). | 26/06/2026 | 26/06/2026 | Boto3 Docs |

### Kết quả đạt được tuần 2:

* **Phát hiện chính từ EDA:**
  * Sales bị lệch phải (right-skewed), cần log-transform target khi train.
  * Có 172,817 records đóng cửa, quyết định loại bỏ khi train.
  * Seasonality rõ ràng theo năm nên cần thêm calendar features.
  * Doanh số Thứ 2/Chủ nhật cao nhất, Thứ 7 thấp nhất nên DayOfWeek là feature quan trọng.
  * Tháng 12 cao vượt trội nên Month là feature quan trọng.
  * Promo tăng Sales lên ~37%, đây là feature quan trọng nhất.
* **Dữ liệu phân chia (Data Split):**
  * Train: 785,727 rows (2013-01-01 → 2015-05-31).
  * Val: 28,423 rows (2015-06-01 → 2015-06-30).
  * Test: 30,188 rows (2015-07-01 → 2015-07-31).
* **Thành phẩm lưu trữ trên S3:**
  * Dữ liệu thô: `train.csv` (38MB), `store.csv` (45KB).
  * Dữ liệu xử lý: `train.csv` (103MB), `val.csv` (3.7MB), `test.csv` (3.9MB), `scaler.pkl` (538B).

### Ghi chú Kỹ thuật (Technical Notes):
* AWS CLI bị lỗi multipart upload với file lớn — phải dùng `boto3.upload_file()` thay thế để đẩy `train.csv` lên S3.
* Cú pháp `fillna(inplace=True)` đã bị deprecated trong pandas mới — nhóm sẽ refactor sang cú pháp mới ở tuần 8.
* Dataset gốc chỉ có đến 2015-07-31, không phải 2015-12-31 như dự kiến ban đầu — do đó split boundaries đã được điều chỉnh phù hợp.
* Scaler được đảm bảo chỉ fit trên train set, sau đó mới transform val và test — tránh data leakage thành công.