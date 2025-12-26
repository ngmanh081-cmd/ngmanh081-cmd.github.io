---
title: "Kiến trúc REST API: Những nguyên tắc cốt lõi cho Backend Developer"
date: 2025-12-16
draft: false
tags: ["API", "REST", "Architecture", "Design"]
summary: "API không chỉ là URL. Tìm hiểu về Stateless, Resource-based, HTTP Verbs và cách thiết kế một API chuẩn mực mà ai cũng muốn dùng."
---

Sau khi đã nắm vững Socket và giao thức, bước tiếp theo của một Network Programmer là xây dựng các ứng dụng Web Service để các hệ thống khác kết nối vào. Tiêu chuẩn phổ biến nhất hiện nay là **REST (Representational State Transfer)**.

REST không phải là một giao thức (như TCP/UDP), mà là một **kiến trúc** (Architecture Style).

## 1. Tư duy "Resource" (Tài nguyên)

Trong REST, mọi thứ đều là tài nguyên. Chúng ta định danh tài nguyên bằng danh từ, không dùng động từ.

* **Sai:** `GET /getAllUsers`, `POST /createNewUser`
* **Đúng:** `GET /users`, `POST /users`

Tại sao? Vì hành động (Lấy, Tạo, Sửa, Xóa) đã được quy định bởi **HTTP Method** rồi.

## 2. 4 Động từ HTTP quyền lực (HTTP Verbs)

Một RESTful API chuẩn sẽ map đúng các hành động CRUD vào các method của HTTP:

1.  **GET:** Dùng để lấy dữ liệu.
    * Ví dụ: `GET /users/1` (Lấy thông tin user có ID là 1).
    * Đặc điểm: An toàn (Safe), không làm thay đổi dữ liệu server.
2.  **POST:** Dùng để tạo mới dữ liệu.
    * Ví dụ: `POST /users` (Tạo một user mới với dữ liệu trong Body).
3.  **PUT:** Dùng để cập nhật toàn bộ (Ghi đè).
    * Ví dụ: `PUT /users/1` (Thay thế user 1 bằng thông tin mới).
4.  **DELETE:** Dùng để xóa dữ liệu.
    * Ví dụ: `DELETE /users/1` (Xóa user 1).

## 3. Nguyên tắc Stateless (Phi trạng thái)

Đây là nguyên tắc quan trọng nhất giúp REST API có thể mở rộng (Scale) lên hàng triệu người dùng.

**Stateless nghĩa là:** Server không lưu giữ trạng thái phiên làm việc (Session) của Client giữa các request. Mỗi request gửi lên phải chứa **đầy đủ thông tin** để Server hiểu (ví dụ: luôn kèm theo Token xác thực).

* **Ví dụ có trạng thái (Stateful):** Client A đăng nhập -> Server lưu "A đã đăng nhập" vào RAM. Nếu Server khởi động lại, A bị văng ra.
* **Ví dụ phi trạng thái (Stateless - REST):** Client A đăng nhập -> Server cấp cho A một cái thẻ bài (Token). Lần sau A gửi request kèm thẻ bài. Server chỉ cần kiểm tra thẻ bài là biết A là ai, không cần nhớ gì cả.

## 4. HTTP Status Codes: Ngôn ngữ của API

Đừng bao giờ trả về 200 OK kèm theo nội dung lỗi. Hãy dùng đúng mã code:

* **2xx (Thành công):** 200 OK, 201 Created.
* **4xx (Lỗi do Client):**
    * `400 Bad Request`: Gửi sai định dạng dữ liệu.
    * `401 Unauthorized`: Chưa đăng nhập.
    * `403 Forbidden`: Đã đăng nhập nhưng không có quyền.
    * `404 Not Found`: Không tìm thấy tài nguyên.
* **5xx (Lỗi do Server):** `500 Internal Server Error` (Code của bạn bị bug).

Thiết kế API tốt là nghệ thuật giao tiếp giữa người viết Server và người viết Client.