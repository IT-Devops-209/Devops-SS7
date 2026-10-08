# Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## Mục tiêu
* Thiết lập tường lửa UFW để bảo mật hệ thống máy chủ chạy nhiều cổng dịch vụ đồng thời.
* Hiểu cách cấu hình mở rộng cổng cho các cổng dịch vụ web và cô lập các cổng cơ sở dữ liệu.

## Yêu cầu
**Bối cảnh:** Máy chủ dự kiến chạy 3 dịch vụ: SSH (22), Nginx (80), Spring Boot (8082) và MySQL (3306). Cần cấu hình tường lửa để bảo vệ hệ thống trước các cuộc tấn công quét cổng từ Internet.
**Ràng buộc:**
* Cấu hình UFW chặn mặc định tất cả lưu lượng đi vào (`deny incoming`).
* Cho phép kết nối SSH (`port 22/tcp`).
* Cho phép cổng kết nối Web HTTP tiêu chuẩn (`port 80/tcp`).
* Cho phép cổng ứng dụng Spring Boot (`port 8082/tcp`).
* Cổng MySQL (`3306/tcp`) bị chặn hoàn toàn từ Internet (không thêm luật allow cho cổng này).

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khai báo chính sách mặc định của Tường lửa
Khóa chặt toàn bộ kết nối đi vào và chỉ cho phép lưu lượng mạng thoát ra ngoài:
```bash
$ sudo ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)

$ sudo ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

### Bước 2: Thiết lập quy tắc cho phép các cổng dịch vụ (Web/App/SSH)
Lần lượt mở cổng cho giao thức TCP của các dịch vụ cần thiết lập kết nối từ Internet:
```bash
$ sudo ufw allow 22/tcp
Rule added
Rule added (v6)

$ sudo ufw allow 80/tcp
Rule added
Rule added (v6)

$ sudo ufw allow 8082/tcp
Rule added
Rule added (v6)
```
*(Lưu ý: Bỏ qua hoàn toàn lệnh mở cổng 3306. Khi không có rule ALLOW cụ thể, cổng 3306 sẽ mặc nhiên bị rớt gói tin (drop) do rule `deny incoming` mặc định)*

### Bước 3: Kích hoạt UFW
Bật tường lửa bảo vệ máy chủ sau khi đã hoàn tất gán các rule:
```bash
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

### Bước 4: Kiểm tra trạng thái và log minh chứng
Sử dụng cờ `verbose` để hiển thị cả trạng thái, chế độ log, rule mặc định và danh sách các Port chi tiết:
```bash
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
8082/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
8082/tcp (v6)              ALLOW IN    Anywhere (v6)
```

**Đánh giá kết quả:**
* Trạng thái đã lên `active` và `Default` đã thiết lập chuẩn xác.
* Các cổng `22`, `80`, và `8082` đã mở cho cả 2 dải mạng IPv4 và IPv6.
* Port `3306` (MySQL) không có mặt trong danh sách, đồng nghĩa hoàn toàn bị cô lập khỏi mạng Internet.
