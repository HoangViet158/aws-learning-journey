---
title: "Tuần 5: Nhận diện ảnh với Amazon Rekognition"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## 1. Mục tiêu

- Tích hợp Amazon Rekognition với ảnh lưu trên S3.
- Gọi `DetectLabels` từ Backend NestJS để nhận diện nội dung ảnh.
- Lọc và chuẩn hóa label theo confidence để tạo danh sách nguyên liệu đầu vào.
- Chuyển kết quả Rekognition sang bước tạo công thức bằng Amazon Bedrock.

## 2. Kế hoạch công việc

> **Thời gian tuần 5:** Thứ 2, 20/07/2026 – Thứ 6, 24/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu Rekognition `DetectLabels`, quyền IAM và định dạng S3 object | Xác định được cấu hình cần thiết |
| Thứ 3 | Xây `RekognitionService` và tích hợp endpoint phân tích ảnh | Backend nhận diện được ảnh đã upload lên S3 |
| Thứ 4 | Lọc label theo confidence, loại nhãn chung và chuẩn hóa tên | Có danh sách nguyên liệu phù hợp |
| Thứ 5 | Kết nối Rekognition với prompt builder và lịch sử AI | Hoàn thiện luồng dữ liệu cho Bedrock |
| Thứ 6 | Thử nghiệm nhiều ảnh và xử lý lỗi AWS SDK | Xác định ngưỡng và trường hợp fallback |

## 3. Nội dung thực hiện

### 3.1. Upload ảnh và gọi Amazon Rekognition

Frontend gửi ảnh dạng `multipart/form-data` đến `POST /api/ai/analyze-image`. Endpoint được bảo vệ bằng `AuthGuard`; Backend lấy `userId`, upload file vào prefix `ai-images/` của bucket S3 rồi truyền object key cho `RekognitionService`. Rekognition đọc trực tiếp object trên S3 nên Backend không cần tải ảnh về lần thứ hai.

```ts
const command = new DetectLabelsCommand({
  Image: {
    S3Object: {
      Bucket: config.getOrThrow<string>('AWS_BUCKET_NAME'),
      Name: imageKey,
    },
  },
  MaxLabels: 30,
  MinConfidence: 50,
});

return rekognitionClient.send(command);
```

**Người dùng chọn ảnh nguyên liệu trước khi bắt đầu phân tích:**

![Ảnh nguyên liệu đã được chọn để phân tích](/images/1-Worklog/1.5-Week5/ai-image-selected.png)

### 3.2. Lọc và chuẩn hóa kết quả

Backend giữ response Rekognition dưới dạng `{ name, confidence }`, làm tròn confidence đến hai chữ số. `IngredientService` chỉ lấy label từ **80%** trở lên, bỏ các nhãn quá chung như `Food`, `Meal`, `Dish`, `Plate` và `Ingredient`, chuẩn hóa một số tên đồng nghĩa, loại bỏ phần tử trùng rồi sắp xếp confidence giảm dần.

Các label gốc vẫn được trả cho Frontend để hiển thị, còn danh sách đã lọc được đưa vào `PromptBuilderService` làm ngữ cảnh cho Bedrock.

**Kết quả nhận diện label và confidence được hiển thị trên giao diện:**

![Kết quả Amazon Rekognition nhận diện ảnh món ăn](/images/1-Worklog/1.5-Week5/rekognition-labels-result.png)

### 3.3. Lưu kết quả và xử lý lỗi

- Ghi log số lượng label Rekognition trả về và số nguyên liệu trích xuất được.
- Lưu danh sách label đã chuẩn hóa trong `ai_generation_history` cùng người dùng, model, trạng thái và công thức được tạo.
- Nếu upload S3 hoặc Rekognition thất bại, Backend ghi log lỗi và trả exception chuẩn hóa; không tiếp tục gọi Bedrock.
- Phân biệt lỗi thiếu object, thiếu quyền `rekognition:DetectLabels`, sai Region và lỗi dịch vụ tạm thời.

**Các label nhận diện được lưu trong lịch sử tạo công thức AI:**

![Label Rekognition được lưu trong cơ sở dữ liệu](/images/1-Worklog/1.5-Week5/rekognition-labels-history.png)

### 3.4. Cấu hình và quyền cần thiết

```env
AWS_REGION=
AWS_BUCKET_NAME=
```

Khi phát triển local, AWS SDK sử dụng credential chain đã cấu hình. Khi chạy trên EC2, ứng dụng dùng IAM Role thay cho static access key. Role của Backend chỉ cần `s3:PutObject`, `s3:GetObject` trên prefix ảnh và `rekognition:DetectLabels`.

## 4. Kiến thức và kỹ năng đạt được

- Gọi Rekognition bằng bucket và object key trên S3.
- Hiểu sự khác nhau giữa ngưỡng request 50% và ngưỡng lọc nghiệp vụ 80%.
- Chuẩn hóa, loại trùng và sắp xếp label trước khi đưa vào prompt.
- Tổ chức log và xử lý lỗi cho pipeline AWS SDK.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Label quá chung chung | Dùng danh sách bỏ qua và ngưỡng nghiệp vụ 80% |
| Nhiều label trùng hoặc đồng nghĩa | Chuẩn hóa tên, loại trùng và sắp xếp theo confidence |
| Rekognition không đọc được object | Kiểm tra Region, bucket, object key và quyền IAM |
| Ảnh không tạo ra nguyên liệu hữu ích | Trả thông báo rõ ràng và yêu cầu người dùng chọn ảnh khác |

## 6. Kết quả đầu ra

- `RekognitionService` tích hợp với `AIGenerationService` trong NestJS.
- Endpoint xác thực nhận ảnh, upload S3 và gọi `DetectLabels` thành công.
- Label được lọc, chuẩn hóa, hiển thị trên Frontend và lưu vào lịch sử AI.
- Danh sách nguyên liệu đã sẵn sàng để xây prompt cho Amazon Bedrock.

## 7. Kế hoạch tuần tiếp theo

Dùng Amazon Bedrock phân tích danh sách label/nguyên liệu từ Rekognition để tạo nội dung công thức.

## 8. Tài liệu tham khảo

- [Amazon Rekognition DetectLabels API](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectLabels.html)
- [Nhận diện nhãn trong ảnh bằng Amazon Rekognition](https://docs.aws.amazon.com/rekognition/latest/dg/labels-detect-labels-image.html)
- [Phân tích ảnh lưu trong Amazon S3](https://docs.aws.amazon.com/rekognition/latest/dg/images-s3.html)
- [Hướng dẫn bảo mật Amazon Rekognition bằng IAM](https://docs.aws.amazon.com/rekognition/latest/dg/security-iam.html)
- [Hạn mức dịch vụ Amazon Rekognition](https://docs.aws.amazon.com/rekognition/latest/dg/limits.html)
