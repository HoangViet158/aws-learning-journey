---
title: "Tuần 6: Phân tích ảnh với Amazon Bedrock"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

## 1. Mục tiêu

- Tích hợp mô hình đa phương thức qua Amazon Bedrock.
- Kết hợp ảnh S3 và label Rekognition để hiểu ngữ cảnh món ăn.
- Gợi ý tên món, mô tả, nguyên liệu và tag công thức.
- Yêu cầu mô hình trả dữ liệu có cấu trúc và kiểm tra trước khi sử dụng.

## 2. Kế hoạch công việc

> **Thời gian tuần 6:** Thứ 2, 27/07/2026 – Thứ 6, 31/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu Bedrock Runtime và khả năng mô hình đa phương thức | Chọn cách gửi ảnh và prompt phù hợp |
| Thứ 3 | Xây Bedrock service trong NestJS | Gọi mô hình và nhận response thành công |
| Thứ 4 | Thiết kế prompt kết hợp ảnh và Rekognition labels | Tăng tính liên quan của kết quả |
| Thứ 5 | Chuẩn hóa JSON schema, validation và fallback | Backend xử lý response ổn định |
| Thứ 6 | Đánh giá chất lượng, latency, token và chi phí | Có bộ prompt và giới hạn sử dụng hợp lý |

## 3. Nội dung thực hiện

### 3.1. Chuẩn bị dữ liệu đầu vào

- Chỉ gửi ảnh đã vượt qua bước Rekognition moderation.
- Chuẩn hóa kích thước và định dạng ảnh theo giới hạn của mô hình.
- Kèm các label có confidence cao để bổ sung ngữ cảnh.
- Không đưa thông tin nhạy cảm hoặc dữ liệu người dùng không cần thiết vào prompt.

### 3.2. Prompt và kết quả có cấu trúc

Yêu cầu mô hình trả JSON gồm tên món gợi ý, mô tả ngắn, nguyên liệu có thể nhận biết, tag và mức độ chắc chắn. NestJS parse và validate response; dữ liệu AI chỉ là gợi ý để người dùng xem lại trước khi lưu.

### 3.3. Độ tin cậy và chi phí

- Giới hạn độ dài output và số lần retry.
- Cache kết quả theo image hash để tránh phân tích lại cùng một ảnh.
- Lưu model ID, phiên bản prompt và thời gian xử lý để truy vết.
- Hiển thị rõ nội dung do AI gợi ý và cho phép người dùng chỉnh sửa.

## 4. Kiến thức và kỹ năng đạt được

- Gọi mô hình đa phương thức thông qua Amazon Bedrock Runtime.
- Thiết kế prompt dựa trên ảnh và dữ liệu Rekognition.
- Parse, validate và fallback khi output AI không đúng schema.
- Cân bằng chất lượng kết quả, latency và chi phí sử dụng mô hình.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| AI trả dữ liệu không đúng JSON | Yêu cầu schema rõ ràng và validate ở Backend |
| Kết quả có thông tin không chắc chắn | Gắn confidence, dùng wording gợi ý và yêu cầu người dùng xác nhận |
| Phân tích lại ảnh làm tăng chi phí | Cache theo hash và tái sử dụng kết quả hợp lệ |

## 6. Kết quả đầu ra

- Bedrock service trong NestJS nhận ảnh và label đã chuẩn hóa.
- Prompt gợi ý thông tin công thức theo JSON schema.
- Cơ chế validation, cache, fallback và xác nhận của người dùng.

## 7. Kế hoạch tuần tiếp theo

Cấu hình CloudFront trước S3 để tăng tốc và kiểm soát truy cập ảnh FoodieRecipe.
