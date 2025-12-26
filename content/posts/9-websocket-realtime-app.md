---
title: "WebSocket: Công nghệ đằng sau các ứng dụng Real-time"
date: 2025-12-17
draft: false
tags: ["Web", "WebSocket", "Realtime", "Protocol"]
summary: "Tại sao HTTP không thể làm ứng dụng chat thời gian thực tốt? So sánh sự khác biệt giữa HTTP Polling và kết nối WebSocket hai chiều (Full-duplex)."
---

Khi bạn lướt Facebook, bạn thấy thông báo (Notification) hiện lên ngay lập tức mà không cần F5 lại trang. Khi bạn chơi game web, nhân vật di chuyển mượt mà. Công nghệ đứng sau những trải nghiệm đó chính là **WebSocket**.

## 1. Vấn đề của HTTP truyền thống

HTTP là giao thức dạng **Request - Response** (Hỏi - Đáp).
* **Client:** "Server ơi, có tin nhắn mới không?"
* **Server:** "Không."
* (1 giây sau)
* **Client:** "Giờ có chưa?"
* **Server:** "Chưa."

Kỹ thuật này gọi là **Polling** (Hỏi liên tục). Nó cực kỳ lãng phí băng thông và tài nguyên server. Server phải trả lời hàng ngàn request vô nghĩa.

## 2. WebSocket: Đường hầm hai chiều

WebSocket giải quyết vấn đề này bằng cách tạo ra một kết nối **Persistent** (bền vững).
1.  **Handshake:** Ban đầu, Client gửi một HTTP Request bình thường nhưng đính kèm header `Upgrade: websocket`.
2.  **Connection Established:** Nếu Server đồng ý, kết nối HTTP sẽ được "nâng cấp" thành WebSocket. Kể từ lúc này, kết nối TCP được giữ nguyên, không đóng lại.
3.  **Full-duplex:**
    * Client có thể gửi tin cho Server bất cứ lúc nào.
    * **Quan trọng:** Server có thể chủ động **đẩy (push)** tin nhắn xuống Client bất cứ lúc nào mà không cần Client phải hỏi.

## 3. So sánh HTTP vs WebSocket

| Đặc điểm | HTTP | WebSocket |
| :--- | :--- | :--- |
| **Kết nối** | Đóng ngay sau khi nhận phản hồi | Giữ kết nối liên tục (Keep-alive) |
| **Giao tiếp** | Một chiều (Client luôn phải mở lời) | Hai chiều (Song công - Full Duplex) |
| **Độ trễ** | Cao (Do phải tạo kết nối mới liên tục) | Rất thấp (Real-time) |
| **Sử dụng** | Web thường, REST API | Chat, Game, Chứng khoán, Live Dashboard |

## 4. Ví dụ luồng hoạt động (Sequence)

```text
Client                         Server
  |      HTTP GET /chat          |
  |   Upgrade: websocket         |
  | ---------------------------> |
  |                              |
  |    HTTP 101 Switching        |
  |        Protocols             |
  | <--------------------------- |
  |                              |
  |    (Kết nối đã mở)           |
  |                              |
  |   [Data Frame: Hello]        |
  | ---------------------------> |
  |                              |
  |   [Data Frame: Hi there]     |
  | <--------------------------- |