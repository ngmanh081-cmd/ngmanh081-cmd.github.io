---


title: "Tối ưu hóa Server: Xử lý hàng ngàn kết nối với Multi-threading"
date: 2025-12-24
draft: false
weight: 2
tags: ["Java", "Concurrency", "Thread", "Advanced"]
summary: "Tại sao Server đơn luồng lại chậm? Hướng dẫn cách sử dụng Thread trong Java để phục vụ nhiều Client cùng lúc mà không bị nghẽn."
---

Ở bài trước, chúng ta đã thấy Server đơn luồng (Single-threaded) hoạt động giống như một cửa hàng chỉ có duy nhất một nhân viên. Nếu nhân viên đó đang phục vụ khách A, khách B buộc phải đứng chờ cho đến khi khách A rời đi. Điều này là không thể chấp nhận được trong thực tế.

Giải pháp là tuyển thêm nhân viên: Mỗi khi có khách hàng mới, ta thuê ngay một nhân viên riêng (Thread) để phục vụ khách đó.

## 1. Cơ chế Thread-per-Connection

Mô hình này hoạt động như sau:
1.  **Main Thread:** Chỉ làm nhiệm vụ đứng ở cửa (Port 6666) và chào đón khách (`accept()`).
2.  Ngay khi khách vào, Main Thread tạo ra một **Worker Thread** mới.
3.  Giao khách đó cho Worker Thread chăm sóc.
4.  Main Thread quay lại cửa để đón khách tiếp theo ngay lập tức.

## 2. Code triển khai Multi-threaded Server

Chúng ta sẽ tách phần xử lý logic ra một class riêng gọi là `ServerThread` thực thi interface `Runnable`.

```java
import java.io.*;
import java.net.*;

public class MultiThreadServer {
    public static void main(String[] args) {
        int port = 6666;
        
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Server Đa luồng đang chạy tại cổng " + port);

            while (true) {
                // 1. Chấp nhận kết nối
                Socket socket = serverSocket.accept();
                System.out.println("Client mới kết nối: " + socket.getInetAddress());

                // 2. Thay vì xử lý ngay, ta tạo một luồng mới
                // và ném socket đó vào cho luồng xử lý
                ServerThread worker = new ServerThread(socket);
                
                // 3. Khởi chạy luồng (nhân viên bắt đầu làm việc)
                new Thread(worker).start();
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}

// Class nhân viên xử lý riêng biệt
class ServerThread implements Runnable {
    private Socket socket;

    public ServerThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            InputStream input = socket.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(input));
            
            OutputStream output = socket.getOutputStream();
            PrintWriter writer = new PrintWriter(output, true);

            String text;
            // Vòng lặp giao tiếp riêng với client này
            while ((text = reader.readLine()) != null) {
                System.out.println("[" + Thread.currentThread().getName() + "] Nhận: " + text);
                writer.println("Server trả lời: " + text);
                
                if ("bye".equalsIgnoreCase(text)) {
                    break;
                }
            }
            socket.close();
        } catch (IOException ex) {
            System.out.println("Lỗi kết nối ServerThread: " + ex.getMessage());
        }
    }
}