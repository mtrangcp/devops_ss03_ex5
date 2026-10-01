# Báo cáo Bài 5: Thiết lập thư mục Web an toàn tại phân vùng hệ thống (/opt/)

## 1. Các bước thực hiện cấu hình và phân quyền

Đăng nhập tài khoản có quyền `sudo` và thực hiện lần lượt các bước sau:

### Bước 1: Tạo cấu trúc thư mục tại phân vùng `/opt/`

```bash
sudo mkdir -p /opt/my-app/html
sudo mkdir -p /opt/my-app/logs
```

### Bước 2: Phân quyền sở hữu và truy cập

```bash
# Phân quyền sở hữu toàn bộ thư mục cho devops (owner) và www-data (group)
sudo chown -R devops:www-data /opt/my-app/

# Phân quyền thư mục: 750 (devops có rwx, www-data có rx, người khác không có quyền)
sudo chmod -R 750 /opt/my-app/

# Riêng thư mục logs, cấp quyền ghi cho group www-data (775) để Nginx ghi nhận logs
sudo chmod -R 775 /opt/my-app/logs/
```

### Bước 3: Cấu hình Nginx Server Block

Tạo file cấu hình cho website tại `/etc/nginx/sites-available/my-app`:

```nginx
server {
    listen 80;
    server_name localhost;

    root /opt/my-app/html;
    index index.html index.htm;

    access_log /opt/my-app/logs/access.log;
    error_log /opt/my-app/logs/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Kích hoạt file cấu hình thông qua liên kết tượng trưng (symlink) và khởi động lại Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

```

## 2. Kiểm tra kết quả đầu ra

### Kiểm tra phân quyền thư mục

```bash
ls -la /opt/my-app/

```

**Kết quả thực tế:**

```text
total 16
drwxr-x--- 4 devops www-data 4096 Oct  2 08:00 .
drwxr-xr-x 3 root   root     4096 Oct  2 08:00 ..
drwxr-x--- 2 devops www-data 4096 Oct  2 08:00 html
drwxrwxr-x 2 devops www-data 4096 Oct  2 08:00 logs

```

### Kiểm tra ghi file từ user `devops` (Không dùng `sudo`)

```bash
su - devops
echo "Update Test" >> /opt/my-app/html/index.html

```

- **Kết quả:** Thao tác ghi file thành công hoàn toàn mà không gặp lỗi từ chối quyền (`Permission denied`).

### Kiểm tra nội dung logs phát sinh

```bash
tail -f /opt/my-app/logs/access.log

```

- **Kết quả:** Khi tiến hành truy cập vào website qua trình duyệt hoặc lệnh `curl`, các yêu cầu được ghi nhận thành công vào file log tại đường dẫn mới mà không gây ra lỗi 403.
