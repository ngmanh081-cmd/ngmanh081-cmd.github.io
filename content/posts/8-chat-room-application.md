---
title: "Project thực chiến: Code ứng dụng Chat Room nhiều người dùng"
date: 2025-12-18
draft: false
tags: ["Project", "Java", "Socket", "Multi-thread"]
summary: "Tổng hợp toàn bộ kiến thức Socket và Thread để xây dựng một phòng chat nơi mọi người có thể nói chuyện với nhau (Broadcast)."
---

Chúng ta đã học về Socket (kết nối) và Thread (đa luồng). Bây giờ là lúc ghép chúng lại để làm một sản phẩm thực tế: **Chat Room Console Application**.

Mô hình hoạt động của Chat Room khác với Echo Server ở chỗ: Khi User A nhắn tin, Server không chỉ trả lại cho A, mà phải **gửi tin nhắn đó cho cả B, C và D**. Kỹ thuật này gọi là **Broadcasting**.

## 1. Kiến trúc hệ thống

Server sẽ cần duy trì một "Danh sách các user đang online".
* Khi có tin nhắn mới, Server duyệt qua danh sách này và gửi tin đi cho từng người.
* Khi có người ngắt kết nối, Server xóa họ khỏi danh sách.

## 2. Code Server hoàn chỉnh

```java
import java.io.*;
import java.net.*;
import java.util.*;

public class ChatServer {
    private static final int PORT = 9090;
    
    // Set chứa các luồng writer của tất cả client đang kết nối
    // Dùng HashSet để tránh trùng lặp
    private static Set<PrintWriter> clientWriters = new HashSet<>();

    public static void main(String[] args) throws Exception {
        System.out.println("Chat Server đang chạy...");
        ServerSocket listener = new ServerSocket(PORT);

        try {
            while (true) {
                // Chấp nhận kết nối và giao cho Handler xử lý
                new Handler(listener.accept()).start();
            }
        } finally {
            listener.close();
        }
    }

    // Class Handler chịu trách nhiệm xử lý từng client riêng biệt
    private static class Handler extends Thread {
        private Socket socket;
        private PrintWriter out;
        private BufferedReader in;

        public Handler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            try {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(), true);

                // 1. Thêm writer của user này vào danh sách chung (đồng bộ hóa)
                synchronized (clientWriters) {
                    clientWriters.add(out);
                }

                // 2. Lắng nghe tin nhắn và Broadcast
                String message;
                while ((message = in.readLine()) != null) {
                    System.out.println("Nhận tin: " + message);
                    
                    // Duyệt qua tất cả user khác và gửi tin nhắn này đi
                    synchronized (clientWriters) {
                        for (PrintWriter writer : clientWriters) {
                            writer.println(message);
                        }
                    }
                }
            } catch (IOException e) {
                System.out.println(e);
            } finally {
                // 3. Nếu user thoát, xóa khỏi danh sách
                if (out != null) {
                    synchronized (clientWriters) {
                        clientWriters.remove(out);
                    }
                }
                try { socket.close(); } catch (IOException e) {}
            }
        }
    }
}