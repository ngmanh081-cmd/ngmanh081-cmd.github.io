---
title: "TCP vs UDP: Khi nào nên đánh đổi sự tin cậy để lấy tốc độ?"
date: 2025-12-22
draft: false
tags: ["Protocol", "Networking", "Theory"]
summary: "Tại sao xem Youtube (Streaming) lại dùng UDP, còn tải file lại dùng TCP? Phân tích sâu về ưu nhược điểm của hai giao thức vận chuyển phổ biến nhất Internet."
---

Trong tầng Giao vận (Transport Layer) của mô hình OSI, có hai "gã khổng lồ" thống trị Internet: **TCP** (Transmission Control Protocol) và **UDP** (User Datagram Protocol).

Là một lập trình viên mạng, câu hỏi phỏng vấn kinh điển bạn sẽ gặp là: *"Sự khác biệt giữa TCP và UDP là gì? Khi nào dùng cái nào?"*

## 1. TCP - "Người vận chuyển cẩn thận"

TCP giống như việc bạn gửi một lá thư bảo đảm (Registered Mail). Bạn muốn chắc chắn 100% người nhận đã nhận được thư, và thư còn nguyên vẹn.

### Đặc điểm của TCP:
1.  **Hướng kết nối (Connection-oriented):** Phải "bắt tay" (Handshake) thiết lập đường truyền trước khi gửi dữ liệu.
2.  **Độ tin cậy cao (Reliable):**
    * **Ack (Acknowledgement):** Gửi gói tin A đi, phải chờ bên kia báo "Đã nhận A" thì mới gửi tiếp B.
    * **Retransmission:** Nếu không thấy bên kia báo nhận, TCP sẽ tự động gửi lại.
3.  **Đảm bảo thứ tự (Ordered):** Gửi A, B, C thì bên nhận sẽ nhận đúng A, B, C. Không bao giờ có chuyện nhận C trước A.

### Nhược điểm:
* **Chậm:** Do phải chờ xác nhận (Ack) và bắt tay 3 bước.
* **Overhead lớn:** Header của gói tin TCP chứa nhiều thông tin kiểm soát (20-60 bytes).

### Code ví dụ (Java):
```java
// TCP luôn yêu cầu ServerSocket và Socket
Socket socket = new Socket("server.com", 80);
InputStream in = socket.getInputStream(); // Dữ liệu chảy như dòng nước (Stream)