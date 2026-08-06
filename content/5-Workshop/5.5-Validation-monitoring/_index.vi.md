---
title: "Kiểm thử, bảo mật và giám sát"
date: 2026-06-22
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Phần này xác nhận workflow hoạt động đúng, lỗi có thể truy vết và tài nguyên không cấp quyền rộng hơn cần thiết.

#### 1. Kiểm thử end-to-end

| Trường hợp | Kết quả mong đợi |
| ---------- | ---------------- |
| Upload JPEG/PNG/WebP hợp lệ | `pending → processing → completed` |
| File sai định dạng/quá dung lượng | Bị từ chối trước khi tạo URL hoặc chuyển `failed` |
| Pre-signed URL hết hạn | S3 từ chối; người dùng yêu cầu URL mới |
| Ảnh vi phạm moderation | `failed` với mã lý do kiểm duyệt |
| Rekognition timeout | Retry giới hạn, không tạo job trùng |
| Bedrock trả JSON sai | Validate thất bại, retry/fallback có kiểm soát |
| Copy từ `uploads/` sang `delivery/` lỗi | Không chuyển sang `completed` |
| Truy cập direct S3 URL | 403 |
| Truy cập CloudFront URL | 200 và cache hoạt động |

Thực hiện mỗi test bằng image ID riêng và ghi lại request ID, latency, trạng thái cuối.

#### 2. Ghi log có cấu trúc

NestJS nên ghi JSON log với các trường:

```json
{
  "event": "image.processing.completed",
  "imageId": "uuid",
  "userId": "masked-or-internal-id",
  "stage": "bedrock",
  "durationMs": 820,
  "status": "completed"
}
```

Không log password, token, AWS credential, raw secret, pre-signed URL đầy đủ hoặc toàn bộ ảnh/base64.

#### 3. Cấu hình CloudWatch Logs

1. Tạo log group `/foodierecipe/api`.
2. Đặt retention phù hợp môi trường workshop.
3. Cấu hình CloudWatch Agent hoặc Docker logging để chuyển log NestJS/Nginx.
4. Xác nhận EC2 role có quyền tạo log stream và ghi log event.
5. Tìm log theo `imageId` để theo dõi xuyên suốt workflow.

Ví dụ Logs Insights:

```text
fields @timestamp, event, imageId, stage, durationMs, status
| filter imageId = "<image-id>"
| sort @timestamp asc
```

#### 4. Metrics và cảnh báo

Theo dõi tối thiểu:

- Số ảnh `completed` và `failed`.
- Thời gian upload, Rekognition, Bedrock và copy S3.
- Tỷ lệ lỗi theo `error_code`.
- Số lần retry và Bedrock invocation.
- CloudFront requests, error rate và cache hit ratio.
- EC2 CPU/disk và RDS connections/storage.

Tạo alarm cho tỷ lệ `failed` tăng cao, EC2 thiếu tài nguyên, CloudFront 5xx và Budget vượt ngưỡng.

#### 5. Rà soát bảo mật

1. S3 image bucket vẫn bật Block Public Access.
2. CloudFront chỉ được đọc prefix `delivery/` qua OAC.
3. Prefix `uploads/` chỉ nhận PUT đã ký từ origin CORS được phép.
4. EC2 dùng IAM role, không lưu static access key.
5. RDS không public và chỉ nhận kết nối từ Backend.
6. Secret không nằm trong Git, Docker image hoặc log.
7. IAM policy không dùng `s3:*`, `rekognition:*` hoặc `bedrock:*` nếu không cần.
8. API kiểm tra ownership trước confirm/delete ảnh.

#### 6. Tiêu chí hoàn thành Workshop

- Một ảnh hợp lệ đi hết luồng và hiển thị bằng CloudFront URL.
- AI suggestion đúng schema và có thể chỉnh sửa.
- Một ảnh bị kiểm duyệt chuyển `failed` với lý do.
- Direct S3 URL không truy cập được.
- Có thể truy vết một image ID trong CloudWatch Logs.
- Không còn quyền public hoặc credential hard-code.

Sau khi kiểm tra xong, chuyển sang [Dọn dẹp tài nguyên](../5.6-cleanup/).
