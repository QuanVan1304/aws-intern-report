---
title: "Dọn dẹp tài nguyên"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Dọn dẹp tài nguyên sau demo

#### Vì sao phải dọn dẹp ngay

**SageMaker Endpoint** (`rossmann-forecasting-endpoint`, instance `ml.t2.medium`) tính phí **theo giờ liên tục** kể từ lúc deploy, bất kể có request gọi đến hay không. Đây là tài nguyên tốn kém nhất trong toàn bộ kiến trúc nếu quên xóa.

#### Script `cleanup.py`

Xóa theo đúng thứ tự phụ thuộc: Endpoint → Endpoint Config → Model.

Log chạy chính thức cuối cùng, sau khi deploy lại và validate lại model từ SageMaker Pipeline:
```text
Ban chac chan muon xoa Endpoint/Model? (yes/no): yes
=== BAT DAU DON DEP RESOURCE ===
✅ Da xoa Endpoint: rossmann-forecasting-endpoint
✅ Da xoa Endpoint Config: rossmann-config-1785134679
✅ Da xoa Model: rossmann-pipeline-xgboost-1785134679
✅ Da xoa Model: rossmann-pipeline-xgboost-1785125081
=== DON DEP HOAN TAT ===
```

{{% notice warning %}}
**Một lỗ hổng thực tế đã được phát hiện và sửa ở đây:** một phiên bản `cleanup.py` trước đó lọc model bằng `NameContains="rossmann-xgboost"`, **không khớp** với tên dạng `rossmann-pipeline-xgboost-*` (chuỗi con không liền mạch). Điều này khiến 1 Model bị bỏ sót sau 1 lần chạy cleanup, chỉ phát hiện được sau khi đối chiếu thủ công với `aws sagemaker list-models`. Bộ lọc sau đó đã được mở rộng — log trên cho thấy nó bắt đúng cả Model hiện tại lẫn 1 Model mồ côi còn sót từ trước, trong cùng 1 lần chạy.
{{% /notice %}}

#### Những gì được giữ lại vs. đã xóa khi kết thúc dự án

| Tài nguyên | Giữ lại? | Lý do |
|---|---|---|
| `model.tar.gz` trên S3 | ✅ Giữ | Dung lượng nhỏ, phí lưu trữ không đáng kể — giữ để deploy lại nhanh sau này |
| API Gateway + Lambda (`rossmann-forecast-api`) | ✅ Giữ | Không tính phí theo giờ khi idle, chỉ tính theo lượt gọi; quản lý qua IaC (`deploy_lambda.py`) nên tạo lại bất cứ lúc nào |
| SageMaker Endpoint | ❌ Xóa | Tính phí theo giờ liên tục — nguồn phát sinh chi phí chính |
| Pipeline thử nghiệm dư thừa (`Rossmann-Sales-Pipeline-V3`, từ thử nghiệm SDK v3) | ❌ Xóa | Không phải giải pháp chính thức (xem quyết định ghim SDK v2); giữ lại chỉ gây rác |
| `Rossmann-Sales-Pipeline` (chính thức) | ✅ Giữ | Pipeline không tính phí theo giờ — giữ lại làm bằng chứng kiến trúc hoàn chỉnh, có thể chạy lại bất cứ lúc nào |

Trạng thái cuối cùng, đã xác nhận qua AWS CLI khi kết thúc dự án: `list-endpoints`, `list-endpoint-configs`, và `list-models` đều trống; `list-pipelines` trả về đúng 1 dòng.

#### Thực hành quản lý chi phí đã áp dụng trong dự án

- Thiết lập **AWS Budget alert**, vì bảng cước AWS có độ trễ — không thể chỉ dựa vào billing dashboard để phát hiện sớm.
- Chạy cleanup ngay sau mỗi lần validate, thay vì để dồn lại sau, tránh quên xóa giữa các lần test.
- Đối chiếu chéo kết quả cleanup với `list-models` / `list-endpoint-configs` / `list-endpoints` qua CLI, thay vì chỉ tin vào thông báo thành công của script — đây chính là cách phát hiện ra lỗ hổng bộ lọc ở trên.

{{% notice warning %}}
Đây là nguyên tắc bắt buộc trong toàn bộ dự án: **mọi Endpoint tạo ra để demo/kiểm thử đều phải được xóa ngay sau đó**, không để chạy qua đêm nếu không có lý do cụ thể.
{{% /notice %}}
