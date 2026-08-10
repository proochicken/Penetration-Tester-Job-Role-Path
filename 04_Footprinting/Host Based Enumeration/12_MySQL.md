# MySQL Enumeration - HTB Academy

## Mục tiêu

MySQL là hệ quản trị cơ sở dữ liệu quan hệ, thường dùng trong web app để lưu user, password hash, bài viết, quyền hạn, email, thông tin khách hàng và cấu hình ứng dụng.

MySQL hoạt động theo mô hình client-server:

```text
Client/App -> MySQL Server -> Database
```

MySQL thường nằm trong stack:

```text
LAMP = Linux + Apache + MySQL + PHP
LEMP = Linux + Nginx + MySQL + PHP
```

## Port mặc định

```text
MySQL TCP/3306
```

Nếu port `3306` mở ra ngoài, cần kiểm tra kỹ vì database không nên public nếu không cần thiết.

## Dangerous Settings

Các setting cần chú ý:

|Setting|Ý nghĩa|
|---|---|
|`user`|User chạy MySQL service|
|`password`|Password user MySQL, có thể bị lộ nếu config file sai quyền|
|`admin_address`|IP lắng nghe cho admin interface|
|`debug`|Có thể lộ thông tin debug|
|`sql_warnings`|Có thể lộ warning/error chi tiết|
|`secure_file_priv`|Giới hạn import/export file|

## Footprinting MySQL

Scan MySQL bằng Nmap:

```bash
sudo nmap <IP> -sV -sC -p3306 --script mysql*
```

Giải thích:

- `-sV`: detect version service
    
- `-sC`: chạy default scripts
    
- `-p3306`: scan port MySQL
    
- `--script mysql*`: chạy các NSE script liên quan MySQL
    

Không tin tuyệt đối output của Nmap. Nếu tool báo empty password hoặc valid credential, cần verify thủ công.

## Kết nối MySQL

Không có password:

```bash
mysql -u root -h <IP>
```

Có password:

```bash
mysql -u root -p<PASSWORD> -h <IP>
```

Lưu ý: không có dấu cách giữa `-p` và password.

Ví dụ đúng:

```bash
mysql -u root -pP4SSw0rd -h 10.129.14.128
```

Ví dụ nếu muốn nhập password thủ công:

```bash
mysql -u root -p -h <IP>
```

## Lệnh MySQL cơ bản

```sql
show databases;
```

Liệt kê database.

```sql
use <database>;
```

Chọn database.

```sql
show tables;
```

Liệt kê bảng trong database hiện tại.

```sql
show columns from <table>;
```

Xem các cột trong bảng.

```sql
select * from <table>;
```

Xem toàn bộ dữ liệu trong bảng.

```sql
select * from <table> where <column> = "<string>";
```

Tìm dữ liệu theo điều kiện.

## Database quan trọng

```text
information_schema
mysql
performance_schema
sys
```

- `information_schema`: metadata về database, table, column.
    
- `sys`: thông tin hệ thống và thống kê.
    
- `mysql`: user, privilege, cấu hình nội bộ.
    
- `performance_schema`: thông tin hiệu năng và runtime.
    

## Tư duy pentest

Khi thấy MySQL mở:

1. Xác định port `3306`.
    
2. Detect version.
    
3. Chạy script enumeration.
    
4. Kiểm tra anonymous/empty password.
    
5. Thử credential đã thu thập được.
    
6. Sau khi login, liệt kê database.
    
7. Tìm database custom của app.
    
8. Kiểm tra bảng user, admin, config, token, session.
    
9. Không tin hoàn toàn output tool, luôn verify thủ công.

---

# MySQL Lab Checklist

## 1. Xác định service

- [ ] Scan port MySQL:

```bash
sudo nmap <IP> -p3306 -sV
```

-  Nếu port mở, ghi lại version.
    
-  Kiểm tra service có thật sự là MySQL/MariaDB không.
    
## 2. Chạy script enumeration
-  Chạy NSE script MySQL:
    
```
sudo nmap <IP> -sV -sC -p3306 --script mysql*
```

-  Kiểm tra output:
    
    -  `mysql-info`
        
    -  `mysql-empty-password`
        
    -  `mysql-brute`
        
    -  `mysql-users`
        
    -  `mysql-databases`
        
    -  `mysql-variables`
        
    -  version
        
    -  auth plugin
        

## 3. Verify kết quả thủ công

-  Nếu Nmap báo empty password, test lại:
    
```
mysql -u root -h <IP>
```

-  Nếu có password:
    
```
mysql -u <user> -p<PASSWORD> -h <IP>
```

-  Nếu muốn tránh lộ password trong shell history:
    

```
mysql -u <user> -p -h <IP>
```

## 4. Sau khi login thành công

-  Liệt kê database:
    

```
show databases;
```

-  Chọn database nghi ngờ:
    

```
use <database>;
```

-  Liệt kê bảng:
    

```
show tables;
```

-  Xem cấu trúc bảng:
    

```
show columns from <table>;
```

-  Dump bảng quan trọng:
    

```
select * from <table>;
```

-  Tìm dữ liệu theo điều kiện:
    

```
select * from <table> where <column> = "<string>";
```

## 5. Database nên kiểm tra

-  `mysql`
    
-  `information_schema`
    
-  `sys`
    
-  database custom của web app
    
-  bảng chứa user/admin
    
-  bảng chứa password/hash/token/API key/session
    

## 6. Kiểm tra cấu hình nguy hiểm nếu có quyền

-  Xem biến MySQL:
    

```
show variables;
```

-  Kiểm tra `secure_file_priv`:
    

```
show variables like 'secure_file_priv';
```

-  Kiểm tra user hiện tại:
    

```
select user();
```

-  Kiểm tra quyền hiện tại:
    

```
show grants;
```

## 7. Ghi chú report

-  MySQL có expose ra ngoài không?
    
-  Có dùng password yếu/empty password không?
    
-  Có lộ version cũ không?
    
-  Có user thừa không?
    
-  Có quyền quá cao không?
    
-  Có dữ liệu nhạy cảm không?
    
-  Có thể đọc/ghi file qua MySQL không?