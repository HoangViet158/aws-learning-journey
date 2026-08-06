---
title: "Tuần 7: Tối ưu truy cập ảnh với Amazon CloudFront"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## 1. Mục tiêu

- Phân phối ảnh FoodieRecipe từ S3 thông qua CloudFront.
- Giữ bucket private và kiểm soát quyền đọc từ CloudFront.
- Thiết kế cache policy phù hợp với ảnh có phiên bản.
- Đo và cải thiện tốc độ tải ảnh trên Next.js.

## 2. Kế hoạch công việc

> **Thời gian tuần 7:** Thứ 2, 03/08/2026 – Thứ 6, 07/08/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Tìm hiểu edge location, cache key, TTL và invalidation | Hiểu cơ chế CDN |
| Thứ 3 | Tạo distribution với S3 origin và Origin Access Control | Bucket không cần public |
| Thứ 4 | Cấu hình cache policy, compression và response headers | Giảm request về S3 |
| Thứ 5 | Tích hợp CloudFront URL vào NestJS và Next.js | Ảnh được tải qua CDN |
| Thứ 6 | Đo cache hit, latency và thử thay/xóa ảnh | Xác nhận cải thiện hiệu năng |

## 3. Nội dung thực hiện

### 3.1. S3 origin riêng tư

- Giữ Block Public Access trên bucket.
- Dùng Origin Access Control và bucket policy chỉ cho CloudFront đọc object.
- Không trả trực tiếp S3 URL cho người dùng.
- Giới hạn method của behavior phục vụ ảnh ở `GET` và `HEAD`.

### 3.2. Chiến lược cache

- Dùng object key có version hoặc hash khi ảnh thay đổi.
- Đặt `Cache-Control` phù hợp cho nội dung ít thay đổi.
- Tránh query string không cần thiết làm phân mảnh cache key.
- Chỉ dùng invalidation cho trường hợp không thể thay đổi object key.

### 3.3. Tích hợp Next.js

NestJS trả CloudFront URL dựa trên object key sau khi ảnh sẵn sàng. Next.js dùng URL này để hiển thị ảnh, thiết lập kích thước phù hợp và lazy loading nhằm giảm dữ liệu tải ban đầu.

## 4. Kiến thức và kỹ năng đạt được

- Hiểu CloudFront distribution, origin, behavior và edge cache.
- Bảo vệ S3 origin bằng Origin Access Control.
- Thiết kế cache key, TTL và versioned object key.
- Đo cache hit/miss và thời gian tải ảnh.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| CloudFront trả 403 dù object tồn tại | Kiểm tra OAC, bucket policy và object key |
| Ảnh cũ còn trong cache | Dùng key có version/hash hoặc invalidation có chọn lọc |
| Cache hit thấp | Giảm biến thể query/header trong cache key |

## 6. Kết quả đầu ra

- CloudFront distribution đọc ảnh từ bucket S3 private.
- URL ảnh CloudFront được tích hợp vào NestJS và Next.js.
- Cache policy và object-versioning strategy giúp cải thiện tốc độ truy cập.

## 7. Kế hoạch tuần tiếp theo

Tích hợp toàn bộ pipeline, kiểm thử, tối ưu chi phí và hoàn thiện tài liệu.
