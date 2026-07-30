---
title: "Serverless API & Demo UI"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---
## Xây dựng REST API (Lambda + API Gateway)

SageMaker Endpoint chỉ có thể được gọi bằng AWS credentials/SDK (`sagemaker-runtime`) — nó không phải là một REST endpoint công khai. Để client bên ngoài gọi được bằng HTTP request thông thường, một lớp Lambda + API Gateway đã được thêm vào.

### AWS Lambda
- Role chuyên biệt: `Lambda-InvokeSageMaker-QuanVan`, chỉ cấp quyền gọi SageMaker Endpoint cộng với quyền ghi CloudWatch Logs cơ bản.
- Code Lambda được quản lý cục bộ trong `lambda_function.py` — nhận request từ API Gateway, gọi `sagemaker-runtime.invoke_endpoint()`, trả về JSON.
- `deploy_lambda.py` (Infrastructure as Code) tự động hóa hoàn toàn việc triển khai Lambda: kiểm tra xem function đã tồn tại chưa, cập nhật code nếu có, tạo mới nếu chưa.

### API Gateway
- REST API mới, Resource `/forecast`, Method `POST`, tích hợp **Lambda Proxy** trỏ tới `rossmann-forecast-api`.
- Deploy vào stage: `prod`.
- Invoke URL: `https://81nxjqyb91.execute-api.ap-southeast-1.amazonaws.com/prod/forecast`
*(Sơ đồ cấu hình phương thức POST trên API Gateway, định tuyến trực tiếp yêu cầu từ phía người dùng (Client) tới thẳng hàm Lambda để xử lý logic)*
![API Gateway Lambda Integration](/images/5-Workshop/5.5-Endpoint-and-serverless-api/apigateway-lambda-integration.png)
### Kiểm thử toàn trình (3 lớp: Lambda → boto3 → REST API thật)

```text
PS ...\week6_deployment> python deploy_lambda.py
Function 'rossmann-forecast-api' da ton tai — update code...
Deploy hoan tat!

PS ...\week6_deployment> python invoke_test.py
✅ KET QUA TU ENDPOINT:
{ "predicted_sales": [ 5267.64 ] }

PS ...\week6_deployment> curl.exe -X POST [https://81nxjqyb91.execute-api.ap-southeast-1.amazonaws.com/prod/forecast](https://81nxjqyb91.execute-api.ap-southeast-1.amazonaws.com/prod/forecast) -H "Content-Type: application/json" -d '{...}'
{"predicted_sales": [5267.64]}
```

Kết quả từ `curl` (qua REST API công khai) khớp với kết quả từ việc gọi Endpoint trực tiếp bằng boto3 — xác nhận toàn bộ chuỗi API Gateway → Lambda → SageMaker Endpoint hoạt động chính xác.

{{% notice tip %}}
Thứ tự test khuyến nghị: `model.predict()` local → gọi SageMaker Endpoint bằng boto3 → gọi Lambda trực tiếp → gọi qua API Gateway. Việc này giúp khoanh vùng lỗi nhanh chóng, vì mỗi lớp chỉ thêm đúng một điểm có thể gây lỗi.
{{% /notice %}}

## Kiến trúc tổng thể luồng Serving

![Serving Architecture](/images/5-Workshop/5.5-Endpoint-and-serverless-api/serving-architecture.png)

{{% notice tip %}}
Toàn bộ luồng Serving độc lập với luồng Training (mục 5.4.2) — SageMaker Endpoint chỉ **đọc** model artifact đã có sẵn trên S3, không tự động kích hoạt lại khi Pipeline Training chạy xong (đúng nguyên tắc CI tách biệt CD đã đề cập ở 5.4.2).
{{% /notice %}}

---

## Demo UI Dashboard (chạy local, không phụ thuộc AWS)

Bên cạnh REST API trên cloud, dự án bao gồm một web UI để demo nhanh mà không cần cấu hình AWS credentials — phù hợp cho thuyết trình.

### Kiến trúc

```
Browser (index.html + app.js + style.css)
      │  fetch POST /api/forecast
      ▼
server.py (Python http.server thuần, cổng 8000)
   — Load xgboost_model.pkl trực tiếp vào RAM lúc khởi động
   — Tái sử dụng build_features_for_store() từ week6_deployment/build_real_features.py
   — Tự predict() ngay trong process, KHÔNG gọi SageMaker Endpoint
```

### Chạy thử

```powershell
python demo_ui/server.py
```
Truy cập: **`http://localhost:8000`**

### Tính năng chính

- **Chọn Store + ngày dự báo**, bật/tắt `Promo`/`SchoolHoliday` → tự động gọi lại `/api/forecast` khi thay đổi.
- **Mô phỏng What-If:** tự động tính toán thêm một dự đoán với trạng thái `Promo` đảo ngược, hiển thị % chênh lệch — minh họa trực quan tác động của khuyến mãi (khớp với insight SHAP ở mục 5.4.1: `Promo` là một trong hai đặc trưng quan trọng nhất).
- **Biểu đồ xu hướng 14 ngày** (Chart.js): nối dữ liệu lịch sử thực tế với điểm dự đoán.
- **Bảng toàn bộ 22 features được tính toán** kèm công thức (ví dụ `rolling_mean_7` → `Mean(Sales[t-7 : t-1])`), minh bạch hóa input của mô hình thay vì là một hộp đen.

### Cơ chế backend (`server.py`)

```python
from build_real_features import load_full_history, build_features_for_store

MODEL = pickle.load(open(MODEL_PATH, "rb"))
DF_HISTORY = load_full_history()

class ForecastRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_POST(self):
        if self.path == '/api/forecast':
            # 1. Nhận store_id, target_date, override Promo/SchoolHoliday từ UI
            # 2. Gọi build_features_for_store() — tái sử dụng logic đã validate ở 5.5.1
            # 3. Áp dụng override từ UI cho các feature Promo/SchoolHoliday nếu bị thay đổi
            # 4. Predict 2 lần: 1 lần với input hiện tại, 1 lần đảo trạng thái Promo cho What-If
            # 5. Lấy 14 ngày lịch sử gần nhất + giá trị thực tế của target_date để vẽ biểu đồ
            # 6. Trả JSON cho frontend
```

Luồng này hoàn toàn nhất quán với logic đã được validate ở mục 5.5.1 (`build_features_for_store`) — đảm bảo demo UI không dùng logic tính toán feature khác với hệ thống production.

{{% notice note %}}
Tiêu đề UI hiện tại ghi "23 Inputs" cho bảng feature, nhưng thực tế có 22 features — một sai lệch hiển thị nhỏ không ảnh hưởng đến tính toán ngầm.
{{% /notice %}}