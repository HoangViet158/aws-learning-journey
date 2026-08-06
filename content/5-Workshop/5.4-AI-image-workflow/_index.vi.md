---
title: "Xây dựng workflow xử lý ảnh AI"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Phần này xây dựng luồng chính của FoodieRecipe từ lúc người dùng chọn ảnh đến khi ảnh được phân phối qua CloudFront.

#### Nội dung

1. [Upload ảnh bằng pre-signed URL](5.4.1-upload-flow/)
2. [Nhận diện và kiểm duyệt với Rekognition](5.4.2-rekognition/)
3. [Gợi ý công thức với Amazon Bedrock](5.4.3-bedrock/)
4. [Phân phối ảnh với Amazon CloudFront](5.4.4-cloudfront/)

#### Quy tắc workflow

- Chỉ dùng `pending`, `processing`, `completed`, `failed`.
- Mỗi ảnh có một `imageId` và object key duy nhất.
- Rekognition chạy trước Bedrock.
- Bedrock output phải được validate và người dùng xác nhận.
- Chỉ ảnh `completed` mới được đưa vào prefix `delivery/`.
- Không public bất kỳ S3 bucket nào.
