# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## Mục tiêu
* Cấu hình máy chủ ảo Nginx Server Block làm Reverse Proxy định tuyến lưu lượng mạng.
* Sử dụng Path Matching để phân chia lưu lượng tĩnh phục vụ trực tiếp và lưu lượng động chuyển tiếp đến API backend Spring Boot.

## Yêu cầu
**Bối cảnh:** Hệ thống phục vụ người dùng truy cập trang chủ tĩnh tại cổng 80 mặc định, và mọi yêu cầu tiền tố `/api/` phải được chuyển tiếp đến Spring Boot ở cổng 8082.
**Ràng buộc:**
* Tạo trang web tĩnh tại `/var/www/html/index.html` hiển thị thông tin học viên.
* Viết cấu hình nginx tại `/etc/nginx/sites-available/spring-proxy.conf`.
* Path `/` trỏ vào `/var/www/html/`.
* Path `/api/` dùng `proxy_pass` trỏ về `http://127.0.0.1:8082/`.
* Test cú pháp Nginx và đảm bảo API trả về dữ liệu chuẩn (không 502 Bad Gateway).

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khởi tạo trang tĩnh
Tạo thư mục nếu chưa có và ghi file `index.html`:
```bash
$ sudo mkdir -p /var/www/html
$ sudo bash -c 'echo "<h1>Học viên: Quang Anh - Lớp: DevOps 209</h1>" > /var/www/html/index.html'
```

### Bước 2: Thiết lập cấu hình Nginx Reverse Proxy
File cấu hình `spring-proxy.conf` đã được thiết kế sẵn theo chuẩn (kèm header nhận diện người dùng thật `X-Real-IP`). Sau khi sao chép vào `/etc/nginx/sites-available`, tiến hành kích hoạt nó bằng lệnh symlink:
```bash
# Tạo symlink
$ sudo ln -s /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/

# Xóa bỏ file cấu hình mặc định để không gây tranh chấp cổng 80
$ sudo rm /etc/nginx/sites-enabled/default
```

### Bước 3: Kiểm tra cấu hình và nạp lại dịch vụ Nginx (Minh chứng nginx -t)
```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo systemctl reload nginx
```

### Bước 4: Kiểm thử hoạt động thực tế qua cURL (Minh chứng)

**1. Truy vấn vào trang chủ tĩnh (đường dẫn `/`):**
```bash
$ curl -i http://localhost/
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Thu, 08 Oct 2026 16:10:05 GMT
Content-Type: text/html
Content-Length: 52
Last-Modified: Thu, 08 Oct 2026 16:05:12 GMT
Connection: keep-alive
ETag: "34-614c3a"
Accept-Ranges: bytes

<h1>Học viên: Quang Anh - Lớp: DevOps 209</h1>
```
*(Kết quả: Trang tĩnh tĩnh phục vụ thành công nội dung HTML chứa mã lớp DevOps 209 với HTTP Code 200)*

**2. Truy vấn API backend xuyên qua Reverse Proxy (đường dẫn `/api/health`):**
```bash
$ curl -i http://localhost/api/health
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Thu, 08 Oct 2026 16:10:15 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"status":"UP"}
```
*(Kết quả: Proxy định tuyến thành công sang cổng 8082, Spring Boot nhận yêu cầu và phản hồi HTTP Code 200 kèm body JSON thể hiện health status. Lỗi 502 hoàn toàn không xảy ra).*
