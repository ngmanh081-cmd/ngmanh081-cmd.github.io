---
title: "Tạm biệt HttpURLConnection: Hướng dẫn sử dụng Java 11 HttpClient"
date: 2025-12-21
draft: false
tags: ["Java 11", "HTTP", "REST Client", "Modern Java"]
summary: "Trước Java 11, gọi API trong Java là một cực hình. Giờ đây, chúng ta đã có một API hiện đại, hỗ trợ HTTP/2 và Asynchronous xịn xò."
---

Nếu bạn đã từng code Java thời kỳ trước (Java 8 trở về trước), hẳn bạn đã từng "phát điên" với `HttpURLConnection`. Code dài dòng, khó cấu hình, và không hỗ trợ bất đồng bộ (Async). Đa số chúng ta phải tìm đến thư viện bên thứ 3 như **Apache HttpClient** hay **OkHttp**.

Kể từ **Java 11**, Oracle đã chính thức giới thiệu `java.net.http.HttpClient` - một làn gió mới thay đổi hoàn toàn cách chúng ta giao tiếp với Web Server.

## 1. Gửi GET Request đơn giản

Hãy so sánh code. Mục tiêu: Gọi API lấy thông tin từ Github.

### Cách cũ (HttpURLConnection):
*(Thôi đừng nhìn, đau mắt lắm...)*

### Cách mới (Java 11 HttpClient):

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class ModernHttp {
    public static void main(String[] args) throws Exception {
        // 1. Tạo Client
        HttpClient client = HttpClient.newHttpClient();

        // 2. Tạo Request (Dùng Builder pattern rất đẹp)
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("[https://api.github.com/users/google](https://api.github.com/users/google)"))
                .GET() // Mặc định là GET, có thể bỏ qua
                .build();

        // 3. Gửi và nhận phản hồi (Synchronous)
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        System.out.println("Status Code: " + response.statusCode());
        System.out.println("Body: " + response.body());
    }
}