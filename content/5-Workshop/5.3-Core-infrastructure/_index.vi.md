---
title: "Xây dựng hạ tầng cốt lõi"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Phần này chuẩn bị các thành phần nền tảng để NestJS lưu ảnh, gọi dịch vụ AI và ghi dữ liệu FoodieRecipe.

#### Nội dung

1. [Tạo S3, IAM và Secrets Manager](5.3.1-storage-security/)
2. [Chuẩn bị NestJS, EC2 và Amazon RDS](5.3.2-backend-data/)

#### Kiến trúc sau phần này

- Một S3 image bucket private với prefix upload/delivery đã sẵn sàng.
- EC2 có IAM role theo nguyên tắc least privilege.
- Database credential được lưu trong Secrets Manager.
- NestJS chạy trong Docker sau Nginx và kết nối được RDS.

{{% notice warning %}}
Chỉ tạo EC2/RDS nếu bạn thực hành kiến trúc sản phẩm đầy đủ. Nếu phần việc của bạn chỉ là tích hợp ảnh, có thể dùng Backend/database hiện có và vẫn hoàn thành các phần S3, Rekognition, Bedrock và CloudFront.
{{% /notice %}}
