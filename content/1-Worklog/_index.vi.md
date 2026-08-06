---
title: "Nhật ký công việc"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Tổng quan

Nhật ký ghi lại quá trình 8 tuần xây dựng tính năng xử lý ảnh cho **FoodieRecipe** — ứng dụng web cho phép người dùng khám phá, đăng tải và quản lý công thức nấu ăn. Frontend sử dụng **Next.js**, Backend sử dụng **NestJS**; ảnh được lưu trên **Amazon S3**, phân tích bằng **Amazon Rekognition** và **Amazon Bedrock**, sau đó phân phối qua **Amazon CloudFront** để tối ưu tốc độ truy cập.

| Tuần | Chủ đề | Chi tiết |
| :---: | ------- | -------- |
| **Tuần 1** | Nền tảng AWS và thiết kế FoodieRecipe | [Tìm hiểu AWS, IAM, Budget Alert; phân tích yêu cầu và thiết kế pipeline xử lý ảnh](1.1-week1/) |
| **Tuần 2** | Amazon S3 và quản lý ảnh | [Thiết kế bucket, object key, phân quyền, metadata và luồng upload ảnh](1.2-week2/) |
| **Tuần 3** | Backend NestJS và S3 | [Xây dựng API upload, validation, pre-signed URL và quản lý trạng thái ảnh](1.3-week3/) |
| **Tuần 4** | Frontend Next.js | [Xây dựng giao diện upload, preview, theo dõi tiến trình và tích hợp NestJS](1.4-week4/) |
| **Tuần 5** | Amazon Rekognition | [Nhận diện nhãn, kiểm duyệt nội dung và trích xuất thông tin từ ảnh món ăn](1.5-week5/) |
| **Tuần 6** | Amazon Bedrock | [Dùng mô hình AI đa phương thức để phân tích ảnh và gợi ý nội dung công thức](1.6-week6/) |
| **Tuần 7** | Amazon CloudFront | [Phân phối ảnh S3 qua CDN, cấu hình cache và tối ưu tốc độ truy cập](1.7-week7/) |
| **Tuần 8** | Tích hợp và hoàn thiện | [Hoàn thiện pipeline S3–Rekognition–Bedrock–CloudFront, kiểm thử và viết tài liệu](1.8-week8/) |
