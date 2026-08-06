---
title: "Tuần 4: Frontend Next.js và trải nghiệm upload ảnh"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## 1. Mục tiêu

- Xây dựng giao diện chọn, xem trước và upload ảnh bằng Next.js.
- Tích hợp luồng pre-signed URL từ Backend NestJS.
- Hiển thị tiến trình, trạng thái xử lý và lỗi rõ ràng.
- Tối ưu trải nghiệm trên máy tính và thiết bị di động.

## 2. Kế hoạch công việc

> **Thời gian tuần 4:** Thứ 2, 13/07/2026 – Thứ 6, 17/07/2026.

| Ngày | Công việc | Kết quả mong đợi |
| :--: | --------- | ---------------- |
| Thứ 2 | Thiết kế component upload và trạng thái giao diện | Có luồng sử dụng rõ ràng |
| Thứ 3 | Thêm drag-and-drop, preview và validation phía client | Phát hiện lỗi trước khi gửi file |
| Thứ 4 | Tích hợp API lấy pre-signed URL và upload trực tiếp S3 | Upload ảnh không đi qua NestJS |
| Thứ 5 | Hiển thị progress, retry và trạng thái xử lý | Người dùng theo dõi được toàn bộ quy trình |
| Thứ 6 | Kiểm tra responsive, accessibility và lỗi mạng | Giao diện ổn định trên nhiều thiết bị |

## 3. Nội dung thực hiện

### 3.1. Component upload

- Hỗ trợ chọn file và kéo thả ảnh.
- Hiển thị preview, tên file, dung lượng và nút thay/xóa ảnh.
- Kiểm tra định dạng, dung lượng trước khi gọi Backend.
- Giải phóng object URL khi component bị hủy để tránh rò rỉ bộ nhớ.

### 3.2. Tích hợp NestJS và S3

- Gọi NestJS để tạo image record và nhận pre-signed URL.
- Upload file trực tiếp lên S3 với đúng `Content-Type`.
- Gọi API xác nhận sau khi upload thành công.
- Không đưa AWS credential vào mã nguồn hoặc trình duyệt.

### 3.3. Trạng thái xử lý

Giao diện biểu diễn bốn trạng thái `pending`, `processing`, `completed` và `failed`. Khi mạng lỗi, người dùng có thể thử lại có kiểm soát mà không tạo nhiều object trùng nhau.

## 4. Kiến thức và kỹ năng đạt được

- Xây dựng component tương tác bằng Next.js và TypeScript.
- Quản lý file, preview và upload progress trên trình duyệt.
- Tích hợp Next.js với NestJS và S3 pre-signed URL.
- Cải thiện accessibility và trải nghiệm xử lý lỗi.

## 5. Khó khăn và hướng xử lý

| Khó khăn | Hướng xử lý |
| -------- | ----------- |
| Upload thất bại nhưng giao diện vẫn báo thành công | Chỉ xác nhận sau khi S3 trả response thành công |
| Người dùng bấm upload nhiều lần | Khóa nút khi đang xử lý và dùng image ID idempotent |
| Preview gây tốn bộ nhớ | Thu hồi object URL sau khi thay ảnh hoặc unmount |

## 6. Kết quả đầu ra

- Component upload ảnh Next.js có preview và validation.
- Luồng upload trực tiếp lên S3 qua pre-signed URL.
- Progress, retry và trạng thái xử lý đồng bộ với NestJS.

## 7. Kế hoạch tuần tiếp theo

Tích hợp Amazon Rekognition để nhận diện và kiểm duyệt ảnh món ăn.
