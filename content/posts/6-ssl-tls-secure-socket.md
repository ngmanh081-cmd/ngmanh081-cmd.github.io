---
title: "Bảo mật ứng dụng Chat: Nâng cấp Socket thường thành SSLSocket"
date: 2025-12-20
draft: false
tags: ["Security", "SSL/TLS", "Java Security", "Cryptography"]
summary: "Dữ liệu truyền qua mạng rất dễ bị nghe lén. Hướng dẫn chi tiết cách mã hóa đường truyền bằng SSL/TLS trong Java."
---

Ở bài viết đầu tiên về Echo Server, chúng ta gửi tin nhắn dưới dạng văn bản thuần (Clear text). Nếu một Hacker dùng chung mạng Wifi với bạn và bật công cụ bắt gói tin lên, họ sẽ đọc được toàn bộ nội dung chat, bao gồm cả mật khẩu nếu có.

Để bảo vệ người dùng, chúng ta cần mã hóa đường truyền bằng **SSL/TLS** (Secure Sockets Layer / Transport Layer Security). Trong Java, chúng ta dùng `SSLSocket` thay vì `Socket` thường.

## 1. Cơ chế hoạt động
SSLSocket hoạt động dựa trên mã hóa khóa công khai (Public Key Infrastructure - PKI):
1.  **Keystore (Server):** Chứa Private Key và Chứng chỉ (Certificate) của Server.
2.  **Truststore (Client):** Chứa danh sách các chứng chỉ mà Client tin tưởng.

Khi kết nối, Server sẽ gửi Chứng chỉ cho Client. Client kiểm tra trong Truststore, nếu khớp thì mới tiến hành trao đổi khóa mã hóa.

## 2. Tạo Keystore (Bước chuẩn bị)
Trước khi code, bạn cần tạo một file chứng chỉ giả (Self-signed certificate) để test. Mở CMD và gõ:

```bash
keytool -genkey -alias myserver -keyalg RSA -keystore server.jks -storepass 123456