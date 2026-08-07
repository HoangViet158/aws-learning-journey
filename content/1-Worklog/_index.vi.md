---
title: "Nhật ký công việc"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Tổng quan

Nhật ký ghi lại quá trình tám tuần tự xây dựng và triển khai hoàn chỉnh **FoodieRecipe** — ứng dụng web cho phép người dùng đăng ký, đăng nhập, khám phá, đăng tải, quản lý, thích và bình luận công thức nấu ăn. Frontend sử dụng **Next.js**, Backend sử dụng **NestJS**, dữ liệu lưu trên **Amazon RDS for PostgreSQL**; ảnh lưu trên **Amazon S3**, được phân tích bằng **Amazon Rekognition** và **Amazon Bedrock**, sau đó phân phối qua **Amazon CloudFront**. Backend được đóng gói bằng Docker, chạy trên EC2 sau Nginx; IAM, Secrets Manager và CloudWatch đảm nhiệm bảo mật, quản lý secret và giám sát.

| Tuần | Chủ đề | Chi tiết |
| :---: | ------- | -------- |
| **Tuần 1** | Nền tảng AWS và thiết kế FoodieRecipe | [Tìm hiểu AWS, IAM, Budget Alert; phân tích yêu cầu và thiết kế pipeline xử lý ảnh](1.1-week1/) |
| **Tuần 2** | S3, RDS và bảo mật dữ liệu | [Thiết kế image/web bucket, mô hình PostgreSQL, IAM và Secrets Manager](1.2-week2/) |
| **Tuần 3** | Backend NestJS | [Xây dựng auth, API công thức, lượt thích, bình luận, upload ảnh và Dockerfile](1.3-week3/) |
| **Tuần 4** | Frontend Next.js | [Xây dựng tài khoản, công thức, tương tác, tìm kiếm, upload ảnh và tích hợp NestJS](1.4-week4/) |
| **Tuần 5** | Amazon Rekognition | [Nhận diện nhãn, kiểm duyệt nội dung và trích xuất thông tin từ ảnh món ăn](1.5-week5/) |
| **Tuần 6** | Amazon Bedrock | [Dùng mô hình AI đa phương thức để phân tích ảnh và gợi ý nội dung công thức](1.6-week6/) |
| **Tuần 7** | Triển khai AWS và CloudFront | [Tạo EC2/RDS, deploy NestJS bằng Docker/Nginx và cấu hình CloudFront](1.7-week7/) |
| **Tuần 8** | Deploy và hoàn thiện sản phẩm | [Deploy Next.js, cấu hình CloudWatch, kiểm thử end-to-end và hoàn thiện tài liệu](1.8-week8/) |
