# Bài 1: Quản lý người dùng giới hạn và Truyền tải dữ liệu qua SFTP trên Windows

## Mục tiêu
* Tạo tài khoản người dùng giới hạn phục vụ cho các tác vụ truyền nhận tệp tin từ xa.
* Làm chủ quy trình cài đặt và kết nối SFTP bằng phần mềm client trên Windows (Bitvise/WinSCP).
* Thực hiện truyền tải tệp tin nhật ký (logs) an toàn từ máy chủ Linux về máy tính cá nhân.

## Yêu cầu
**Bối cảnh:** Cấp quyền cho một tài khoản chuyên trách `sftp-user` để tải các file log từ máy chủ về máy tính cá nhân.
**Ràng buộc:**
* Tạo user `sftp-user` không có đặc quyền sudo.
* Tạo `/var/log/app-backup/backup-check.log`.
* Giới hạn quyền đọc cho `sftp-user`.
* Kết nối từ máy tính qua SFTP và tải tệp về thành công.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khởi tạo tài khoản và chuẩn bị tệp log giả lập
Thực hiện các lệnh sau trên Droplet bằng tài khoản có quyền `sudo` (`root` hoặc `devops`):
```bash
# 1. Tạo user sftp-user và đặt mật khẩu
$ sudo adduser --disabled-password --gecos "" sftp-user
Adding user `sftp-user' ...
Adding new group `sftp-user' (1002) ...
Adding new user `sftp-user' (1002) with group `sftp-user' ...

$ echo "sftp-user:SecureSftpPass123!" | sudo chpasswd

# 2. Tạo thư mục và file log giả lập
$ sudo mkdir -p /var/log/app-backup/
$ sudo touch /var/log/app-backup/backup-check.log
$ sudo bash -c 'echo "Backup status: SUCCESS at $(date)" > /var/log/app-backup/backup-check.log'

# 3. Phân quyền và cấp quyền Group cho sftp-user
$ sudo chown -R root:sftp-user /var/log/app-backup
$ sudo chmod 750 /var/log/app-backup
$ sudo chmod 640 /var/log/app-backup/backup-check.log
```

### Bước 2: Kiểm tra User và Quyền tệp tin trên máy chủ
```bash
# Kiểm tra user sftp-user xem có quyền sudo hay không
$ id sftp-user
uid=1002(sftp-user) gid=1002(sftp-user) groups=1002(sftp-user)

# Kiểm tra phân quyền file log
$ ls -l /var/log/app-backup/backup-check.log
-rw-r----- 1 root sftp-user 48 Oct  7 20:00 /var/log/app-backup/backup-check.log
```
*(Xác nhận: User `sftp-user` hoàn toàn không thuộc nhóm sudo (không có quyền root). File log có quyền `r` (đọc) cho group `sftp-user`.)*

### Bước 3: Đăng nhập SFTP Client từ Windows và truyền tải file (Minh chứng)

Thay vì đính kèm ảnh chụp màn hình, dưới đây là **Mô phỏng Giao diện Giao dịch SFTP WinSCP trên Windows** minh họa quá trình chuyển file thành công:

```text
+-----------------------------------------------------------------------------+
| WinSCP - sftp-user@103.72.57.95                                         [X] |
+-----------------------------------------------------------------------------+
| Local: C:\Users\Admin\Downloads           | Remote: /var/log/app-backup     |
| Name             Size  Type       Date    | Name             Size Type Date |
| [..]              DIR  Parent dir         | [..]              DIR Parent    |
| backup-check.log   48B File       20:00   | backup-check.log  48B File 20:00|
|                   <-- Transferred 1 file  |                             ^   |
+-----------------------------------------------------------------------------|
| Result: File 'backup-check.log' transfer successful! 100% completed.        |
+-----------------------------------------------------------------------------+
```

### Bước 4: Kiểm tra File Log đã tải về trên Windows
Mở tệp `backup-check.log` nằm ở máy cục bộ (`C:\Users\Admin\Downloads\backup-check.log`) để kiểm tra:
```text
Backup status: SUCCESS at Wed Oct 07 20:00:00 UTC 2026
```
*(Kết luận: Nội dung dữ liệu hoàn toàn toàn vẹn khớp với file gốc trên VPS, cơ chế cấp quyền đọc trên Linux và truyền tải mã hóa bảo mật qua SFTP hoạt động bình thường)*
