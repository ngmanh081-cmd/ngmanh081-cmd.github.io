---
title: "Java NIO: Bí mật đằng sau các High-Performance Server"
date: 2025-12-23
draft: false
weight: 3
tags: ["Java", "NIO", "Netty", "Architecture"]
summary: "Phân tích sự khác biệt giữa Blocking IO và Non-Blocking IO. Tại sao Node.js hay Netty lại nhanh đến vậy?"
---

Nếu bạn thắc mắc tại sao **Node.js** (đơn luồng) lại có thể xử lý hàng chục nghìn kết nối đồng thời, hay tại sao các framework Java hiện đại như **Netty**, **Vert.x** lại có hiệu năng vượt trội so với Tomcat truyền thống, thì câu trả lời nằm ở: **Non-blocking I/O (NIO)**.

Trong Java, gói thư viện này nằm ở `java.nio` (được giới thiệu từ Java 1.4).

## 1. IO Truyền thống (BIO) vs NIO

### BIO (Blocking I/O) - Mô hình cũ
Như các bài trước, BIO dựa trên **Stream** (Luồng dữ liệu).
* Khi gọi `input.read()`, Thread bị chặn (block) cho đến khi có dữ liệu.
* **Hệ quả:** Cần 1 Thread cho mỗi kết nối. 10k kết nối = 10k Threads. Rất tốn kém.

### NIO (Non-blocking I/O) - Mô hình mới
NIO dựa trên **Buffer** (Vùng nhớ) và **Channel** (Kênh).
* Khi gọi đọc dữ liệu, nếu chưa có dữ liệu, nó trả về ngay lập tức thay vì bắt chờ đợi. Thread đó có thể đi làm việc khác.
* **Hệ quả:** Chỉ cần **1 Thread duy nhất** có thể quản lý 10k kết nối.

## 2. Ba trụ cột của Java NIO

Để hiểu NIO, bạn cần nắm 3 khái niệm:

1.  **Buffer:** Là một vùng nhớ (block of memory) để chứa dữ liệu đọc/ghi. Trong NIO, bạn luôn đọc từ Channel vào Buffer, hoặc ghi từ Buffer ra Channel.
2.  **Channel:** Giống như Socket nhưng là đường 2 chiều (có thể vừa đọc vừa ghi). Quan trọng nhất là nó hỗ trợ chế độ phi đồng bộ (asynchronous).
3.  **Selector (Bộ chọn):** Đây là "nhạc trưởng". Một Selector có thể giám sát hàng trăm Channel. Khi nào Channel nào có dữ liệu ("Hey, tôi có tin nhắn mới!"), Selector sẽ báo cho Thread xử lý.

## 3. Code ví dụ: NIO Echo Server

Code NIO phức tạp hơn BIO rất nhiều, nhưng "đắt xắt ra miếng".

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class NioServer {
    public static void main(String[] args) throws IOException {
        // 1. Mở Selector
        Selector selector = Selector.open();

        // 2. Mở ServerSocketChannel (thay vì ServerSocket thường)
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.bind(new InetSocketAddress("localhost", 6666));
        
        // QUAN TRỌNG: Cấu hình chế độ Non-blocking
        serverSocket.configureBlocking(false);

        // 3. Đăng ký kênh server với Selector, lắng nghe sự kiện ACCEPT (kết nối mới)
        serverSocket.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("NIO Server đang chạy...");

        while (true) {
            // Chờ cho đến khi có ít nhất 1 sự kiện xảy ra
            selector.select();

            // Lấy danh sách các kênh đang kích hoạt (có sự kiện)
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iter = selectedKeys.iterator();

            while (iter.hasNext()) {
                SelectionKey key = iter.next();

                if (key.isAcceptable()) {
                    // Sự kiện: Có client mới kết nối
                    register(selector, serverSocket);
                }

                if (key.isReadable()) {
                    // Sự kiện: Có dữ liệu gửi đến để đọc
                    answerWithEcho(key);
                }

                // Xử lý xong thì xóa key khỏi danh sách để tránh lặp lại
                iter.remove();
            }
        }
    }

    private static void register(Selector selector, ServerSocketChannel serverSocket) throws IOException {
        SocketChannel client = serverSocket.accept();
        client.configureBlocking(false); // Client này cũng phải Non-blocking
        client.register(selector, SelectionKey.OP_READ); // Chỉ quan tâm khi nào nó gửi tin nhắn
        System.out.println("Client mới kết nối: " + client.getRemoteAddress());
    }

    private static void answerWithEcho(SelectionKey key) throws IOException {
        SocketChannel client = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(256); // Tạo vùng đệm 256 bytes

        int r = client.read(buffer);
        if (r == -1) {
            client.close(); // Client đã ngắt kết nối
            System.out.println("Client ngắt kết nối.");
        } else {
            // Chuyển chế độ Buffer từ ghi sang đọc
            buffer.flip(); 
            // Ghi dữ liệu từ Buffer trả ngược lại Client (Echo)
            client.write(buffer);
            buffer.clear();
        }
    }
}