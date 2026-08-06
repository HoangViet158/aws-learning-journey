---
title: "Tuần 5: Nhận diện ảnh với Amazon Rekognition"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## 1. Mục tiêu

- Tích hợp Amazon Rekognition với ảnh lưu trên S3.
- Nhận diện nhãn liên quan đến món ăn và nguyên liệu.
- Phát hiện nội dung không phù hợp trước khi công khai ảnh.
- Chuẩn hóa kết quả nhận diện để Backend và Frontend sử dụng.

## 2. Kế hoạch công việc

> **Thời gian tuần 5:** Thứ 2, 20/07/2026 – Thứ 6, 24/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu DetectLabels và DetectModerationLabels | Chọn API phù hợp cho từng mục đích |
| Thứ 3 | Tạo Rekognition service trong NestJS | Phân tích được ảnh từ S3 |
| Thứ 4 | Lọc label theo confidence và nhóm dữ liệu | Có kết quả phù hợp với FoodieRecipe |
| Thứ 5 | Xây quy tắc duyệt/từ chối ảnh và lưu kết quả | Kiểm soát nội dung tự động |
| Thứ 6 | Thử nghiệm với nhiều ảnh món ăn và trường hợp biên | Đánh giá độ chính xác và ngưỡng |

## 3. Nội dung thực hiện

### 3.1. Gọi Amazon Rekognition

- Truyền bucket và object key thay vì tải lại toàn bộ ảnh qua Backend.
- Gọi nhận diện label và kiểm duyệt nội dung sau khi upload được xác nhận.
- Giới hạn số label và đặt ngưỡng confidence để giảm nhiễu.
- Xử lý timeout, throttling và lỗi object không tồn tại.

### 3.2. Chuẩn hóa kết quả

Chuyển response thành cấu trúc nội bộ gồm tên label, confidence, loại kết quả và thời điểm phân tích. Các label liên quan đến food, dish hoặc ingredient được ưu tiên làm dữ liệu đầu vào cho bước Bedrock.

### 3.3. Kiểm duyệt ảnh

- Nếu moderation label vượt ngưỡng, chuyển ảnh sang `failed` và lưu lý do `moderation_rejected`.
- Với kết quả chưa rõ ràng, vẫn dùng `failed` nhưng lưu lý do `moderation_review` để kiểm tra thủ công.
- Ảnh vượt qua kiểm duyệt tiếp tục giữ trạng thái `processing` để chuyển sang bước Bedrock.

## 4. Kiến thức và kỹ năng đạt được

- Sử dụng Rekognition với object trên S3.
- Hiểu confidence score và cách chọn ngưỡng phù hợp.
- Thiết kế luồng kiểm duyệt tự động có phương án kiểm tra thủ công.
- Chuẩn hóa kết quả AI cho các bước xử lý tiếp theo.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Label quá chung chung | Lọc theo nhóm từ khóa FoodieRecipe và confidence |
| Ảnh món ăn phức tạp cho nhiều kết quả | Giữ top label và chuyển cho Bedrock tổng hợp ngữ cảnh |
| Kết quả kiểm duyệt có thể sai | Dùng ngưỡng thận trọng và trạng thái review thủ công |

## 6. Kết quả đầu ra

- Rekognition service tích hợp trong NestJS.
- Luồng nhận diện label và kiểm duyệt ảnh từ S3.
- Dữ liệu kết quả chuẩn hóa cho Frontend và Amazon Bedrock.

## 7. Kế hoạch tuần tiếp theo

Dùng Amazon Bedrock phân tích ảnh và label để gợi ý nội dung công thức.
