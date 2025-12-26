---
title: "Nhập môn Lập trình mạng: Xây dựng TCP Echo Server với Java"
date: 2025-12-26
draft: false
weight: 1
tags: ["Java", "TCP", "Socket", "Tutorial"]
summary: "Hướng dẫn chi tiết từ A-Z cách hoạt động của mô hình Client-Server, khái niệm Handshake và code hoàn chỉnh cho ứng dụng Echo."
---

Lập trình mạng (Network Programming) là xương sống của Internet. Mọi ứng dụng từ Web, Game Online, cho đến hệ thống ngân hàng đều dựa trên việc truyền tải dữ liệu giữa các máy tính. Trong bài viết này, chúng ta sẽ bắt đầu với giao thức phổ biến nhất: **TCP (Transmission Control Protocol)** thông qua thư viện `java.net` của Java.

Chúng ta sẽ xây dựng một ứng dụng "Hello World" của thế giới mạng: **Echo Server**. Nguyên lý của nó rất đơn giản: Client gửi câu gì, Server sẽ "nhại" lại y hệt câu đó.

## 1. Lý thuyết cơ bản về TCP Socket

Trước khi đi vào code, bạn cần hiểu điều gì xảy ra "bên dưới nắp capo".

### Socket là gì?
Hãy tưởng tượng Socket giống như một bến cảng. Để một con tàu (dữ liệu) đi từ thành phố A (Client) sang thành phố B (Server), nó cần biết:
1.  **Địa chỉ IP:** Thành phố đích đến (Ví dụ: 192.168.1.1 hoặc localhost).
2.  **Port (Cổng):** Số hiệu bến cảng cụ thể (Ví dụ: 80, 443, 8080).

### Quy trình bắt tay 3 bước (3-way Handshake)
TCP là giao thức tin cậy (reliable). Để đảm bảo kết nối, nó thực hiện quy trình bắt tay:
1.  **SYN:** Client gửi yêu cầu kết nối ("Em muốn kết nối với anh").
2.  **SYN-ACK:** Server đồng ý và xác nhận ("Ok, anh nghe đây").
3.  **ACK:** Client xác nhận lại ("Vậy mình bắt đầu nói chuyện nhé").

Trong Java, toàn bộ quy trình phức tạp này được gói gọn trong một dòng lệnh `new Socket()` hoặc `server.accept()`.

---

## 2. Xây dựng Server (Phía máy chủ)

Server cần làm hai việc chính: **Lắng nghe** tại một cổng cụ thể và **Chờ đợi** Client kết nối.

Dưới đây là code đầy đủ cho `EchoServer.java`:

```java
import java.io.*;
import java.net.*;

public class EchoServer {
    // Chọn một cổng chưa được sử dụng (thường > 1024)
    public static final int PORT = 6666;

    public static void main(String[] args) {
        System.out.println("Server đang khởi động...");
        
        // Sử dụng try-with-resources để tự động đóng socket khi xong
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Server đang lắng nghe tại cổng " + PORT);

            // Vòng lặp vô tận để server luôn chạy
            while (true) {
                // 1. Chặn (Block) cho đến khi có Client kết nối
                Socket socket = serverSocket.accept();
                System.out.println("Client mới đã kết nối: " + socket.getInetAddress());

                // 2. Tạo luồng đọc/ghi dữ liệu
                InputStream input = socket.getInputStream();
                BufferedReader reader = new BufferedReader(new InputStreamReader(input));

                OutputStream output = socket.getOutputStream();
                PrintWriter writer = new PrintWriter(output, true);

                // 3. Đọc dữ liệu từ Client gửi lên
                String text;
                while ((text = reader.readLine()) != null) {
                    System.out.println("Nhận được: " + text);
                    
                    // Logic của Echo Server: Gửi lại chính câu đó
                    writer.println("Server phản hồi: " + text);
                    
                    // Nếu Client gõ 'bye' thì ngắt kết nối
                    if ("bye".equalsIgnoreCase(text)) {
                        break;
                    }
                }
                
                // Đóng kết nối với client này
                socket.close();
                System.out.println("Client đã ngắt kết nối.");
            }

        } catch (IOException ex) {
            System.out.println("Lỗi Server: " + ex.getMessage());
            ex.printStackTrace();
        }
    }
}