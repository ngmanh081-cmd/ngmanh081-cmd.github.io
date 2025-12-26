---
title: "Debug như một chuyên gia: Phân tích gói tin với Wireshark"
date: 2025-12-19
draft: false
tags: ["Tools", "Wireshark", "Debugging", "Network Analysis"]
summary: "Lập trình viên mạng mà không biết dùng Wireshark thì giống như bác sĩ mà không có máy chụp X-quang. Hướng dẫn bắt và đọc gói tin TCP/HTTP."
---

Khi ứng dụng mạng của bạn bị lỗi (ví dụ: Client gửi mà Server không nhận được, hoặc dữ liệu bị sai), việc nhìn vào Console Log (`System.out.println`) là chưa đủ. Bạn cần nhìn thấy **chính xác những gì đang chạy trên dây cáp mạng**.

**Wireshark** là công cụ phân tích giao thức mạng số 1 thế giới. Nó cho phép bạn bắt (capture) và xem chi tiết từng bit dữ liệu.

## 1. Cài đặt và Bắt gói tin (Capture)

1.  Tải Wireshark tại [wireshark.org](https://www.wireshark.org/).
2.  Mở lên, chọn Card mạng bạn đang dùng (thường là `Wi-Fi` hoặc `Ethernet`).
3.  Bấm vào biểu tượng vây cá mập màu xanh để bắt đầu "săn mồi".

## 2. Lọc tạp âm (Filtering)
Mạng máy tính rất ồn ào (Windows update, Facebook, Zalo chạy ngầm...). Để tìm đúng gói tin của ứng dụng bạn viết, hãy dùng thanh Filter:

* Lọc theo cổng: `tcp.port == 6666`
* Lọc theo IP: `ip.addr == 192.168.1.5`
* Lọc theo giao thức: `http` hoặc `websocket`

## 3. Tính năng "Follow TCP Stream" (Thần thánh)
Đây là tính năng hữu ích nhất cho lập trình viên.
1.  Chuột phải vào một gói tin bất kỳ của ứng dụng bạn.
2.  Chọn **Follow** -> **TCP Stream**.
3.  Một cửa sổ mới hiện ra, hiển thị toàn bộ nội dung cuộc hội thoại giữa Client và Server dưới dạng văn bản dễ đọc.
    * **Màu đỏ:** Client gửi đi.
    * **Màu xanh:** Server trả lời.

## 4. Phân tích lỗi thường gặp
* **TCP Retransmission (Màu đen):** Cảnh báo mạng chập chờn, gói tin bị mất và phải gửi lại.
* **TCP Zero Window:** Server đang bị quá tải, bộ đệm đầy, bảo Client "từ từ hãy gửi".
* **RST (Reset):** Kết nối bị ngắt đột ngột (Crash hoặc bị Firewall chặn).

Biết dùng Wireshark, bạn sẽ không bao giờ phải đoán mò nguyên nhân lỗi mạng nữa.