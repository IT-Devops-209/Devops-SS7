# Bài 3: Thiết lập Cơ sở dữ liệu và Tự cấu hình dịch vụ Systemd cho Spring Boot

## Mục tiêu
* Thực hành khởi tạo cơ sở dữ liệu MySQL và cấu hình phân quyền người dùng bảo mật.
* Tự thiết kế và viết tệp tin cấu hình dịch vụ Systemd hoàn chỉnh từ đầu để quản lý tiến trình ứng dụng Spring Boot.
* Thực thi ứng dụng dưới quyền một người dùng giới hạn (non-root) để bảo vệ hệ điều hành.

## Yêu cầu
**Bối cảnh:** Cần khởi tạo database trong MySQL, cấu hình app Spring Boot kết nối DB và chạy tự động bằng Systemd.
**Ràng buộc:**
* Tạo DB `springboot_db` và cấp toàn quyền cho user `spring-admin` (mật khẩu `SpringSecure@123`).
* Tạo user hệ thống `spring-runner` (không có shell login).
* Viết tệp dịch vụ `/etc/systemd/system/spring-app.service` chứa cấu hình chạy app dưới quyền `spring-runner`, auto restart sau 10 giây nếu bị lỗi.
* Ứng dụng phải lắng nghe ở cổng `8082`.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khởi tạo cơ sở dữ liệu MySQL và người dùng
Truy cập MySQL và cấu hình database:
```bash
$ sudo mysql -u root
mysql> CREATE DATABASE springboot_db;
Query OK, 1 row affected (0.01 sec)

mysql> CREATE USER 'spring-admin'@'localhost' IDENTIFIED BY 'SpringSecure@123';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> EXIT;
```

### Bước 2: Tạo User dịch vụ và Cấu hình Systemd
Tạo user chạy dịch vụ bị vô hiệu hóa shell login để ngăn chặn việc bị chiếm quyền điều khiển trực tiếp:
```bash
$ sudo useradd -r -s /usr/sbin/nologin spring-runner
```
Tự viết file cấu hình `/etc/systemd/system/spring-app.service` (nội dung cụ thể xem file đính kèm trong thư mục này).

Sau đó nạp lại cấu hình và khởi động:
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start spring-app.service
$ sudo systemctl enable spring-app.service
Created symlink /etc/systemd/system/multi-user.target.wants/spring-app.service → /etc/systemd/system/spring-app.service.
```

### Bước 3: Kiểm tra minh chứng (Systemctl và ss)

**1. Log hiển thị trạng thái dịch vụ đang chạy ổn định:**
```bash
$ sudo systemctl status spring-app.service
● spring-app.service - Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/spring-app.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-10-08 23:20:00 +07; 15s ago
   Main PID: 12450 (java)
      Tasks: 35 (limit: 1120)
     Memory: 250.0M
     CGroup: /system.slice/spring-app.service
             └─12450 /usr/bin/java -jar /opt/spring-app/app.jar
```
*(Xác nhận: Trạng thái `active (running)` và cấu trúc file Unit chạy chuẩn xác).*

**2. Log kiểm tra cổng lắng nghe (port 8082):**
```bash
$ sudo ss -tlnp | grep 8082
LISTEN   0        100                    *:8082                  *:*      users:(("java",pid=12450,fd=15))
```
*(Xác nhận: Tiến trình `java` mang mã 12450 đang chiếm quyền lắng nghe tại cổng `8082` một cách chính xác).*
