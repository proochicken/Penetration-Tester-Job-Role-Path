# IMAP / POP3 Footprinting

## Vai trò

IMAP và POP3 dùng để truy cập mailbox.

```text
SMTP  -> gửi email
IMAP  -> đọc/quản lý email trên server
POP3  -> tải/lấy email từ server
```

## Port phổ biến

|Protocol|Port|Ý nghĩa|
|---|--:|---|
|POP3|110|POP3 plaintext hoặc STLS|
|POP3S|995|POP3 over SSL/TLS|
|IMAP|143|IMAP plaintext hoặc STARTTLS|
|IMAPS|993|IMAP over SSL/TLS|

## IMAP

IMAP cho phép quản lý email trực tiếp trên server.

Đặc điểm:

- Email nằm trên server.
    
- Đồng bộ nhiều thiết bị.
    
- Hỗ trợ folder/mailbox structure.
    
- Có thể đọc, quản lý, di chuyển email online.
    

## POP3

POP3 đơn giản hơn IMAP.

Chức năng chính:

- Liệt kê email.
    
- Tải email.
    
- Xóa email.
    

POP3 không mạnh về đồng bộ nhiều thiết bị và folder structure như IMAP.

## IMAP Commands

| Command                     | Ý nghĩa                            |
| --------------------------- | ---------------------------------- |
| `1 LOGIN username password` | Đăng nhập                          |
| `1 LIST "" *`               | Liệt kê mailbox/folder             |
| `1 SELECT INBOX`            | Chọn INBOX                         |
| `1 SEARCH ALL`              | Liệt kê các thông tin trong folder |
| `1 FETCH <ID> all`          | Lấy thông tin email theo ID        |
| `1 CLOSE`                   | Xóa các mail đã đánh dấu Deleted   |
| `1 LOGOUT`                  | Thoát                              |

## POP3 Commands

|Command|Ý nghĩa|
|---|---|
|`USER username`|Nhập username|
|`PASS password`|Nhập password|
|`STAT`|Xem số lượng email|
|`LIST`|Liệt kê email và kích thước|
|`RETR id`|Đọc email theo ID|
|`DELE id`|Xóa email theo ID|
|`CAPA`|Xem capability của server|
|`RSET`|Reset trạng thái|
|`QUIT`|Thoát|

## Dangerous Settings

|Setting|Rủi ro|
|---|---|
|`auth_debug`|Log chi tiết auth|
|`auth_debug_passwords`|Có thể log password|
|`auth_verbose`|Log failed login|
|`auth_verbose_passwords`|Có thể log password failed|
|`auth_anonymous_username`|Có thể liên quan anonymous login|

## Nmap Footprinting

```bash
sudo nmap -sV -sC -p110,143,993,995 <target-ip>
```

Mục tiêu:

- Xác định POP3/IMAP có mở không.
    
- Xác định service/version, ví dụ Dovecot.
    
- Xem capability.
    
- Xem SSL certificate.
    
- Lấy hostname/commonName từ certificate.
    

## cURL IMAPS

```bash
curl -k 'imaps://<target-ip>' --user user:password
```

Dùng để đăng nhập IMAPS và liệt kê mailbox.

Verbose mode:

```bash
curl -k 'imaps://<target-ip>' --user user:password -v
```

Giúp xem TLS version, certificate, banner, capability.

## OpenSSL

Kết nối POP3S:

```bash
openssl s_client -connect <target-ip>:pop3s
```

Kết nối IMAPS:

```bash
openssl s_client -connect <target-ip>:imaps
```

Sau khi kết nối thành công, có thể nhập lệnh IMAP/POP3 thủ công.

## Tư duy lab

Nếu đã tìm được credential từ SMTP, hãy thử dùng credential đó đăng nhập IMAP/POP3.

Ví dụ:

```text
username: robin
password: robin
```

Sau khi login:

1. Liệt kê mailbox.
    
2. Chọn INBOX.
    
3. Liệt kê email.
    
4. Fetch/read email.
    
5. Tìm credential, flag, internal info.

---
# IMAP / POP3 Lab Checklist

## 1. Scan port

- [ ] Scan các port mail retrieval phổ biến:

```bash
sudo nmap -sV -sC -p110,143,993,995 <target-ip>
```

-  Ghi lại service:
    
```
Dovecot pop3d
Dovecot imapd
```

-  Ghi lại hostname/certificate CN nếu có.
    
## 2. Đọc capability

-  Xem output Nmap:
    
```
imap-capabilities
pop3-capabilities
```

-  Chú ý:
    
```
AUTH=PLAIN
STARTTLS
STLS
CAPA
SASL
LOGIN
```

## 3. Nếu có credential

-  Thử IMAPS bằng curl:
    
```
curl -k 'imaps://<target-ip>' --user <user>:<password>
```

-  Thử verbose để xem thêm thông tin:
    
```
curl -k 'imaps://<target-ip>' --user <user>:<password> -v
```

## 4. Tương tác IMAP qua TLS

-  Kết nối:
    

```
openssl s_client -connect <target-ip>:imaps
```

-  Đăng nhập:
    
```
1 LOGIN <user> <password>
```

-  Liệt kê mailbox:
    

```
1 LIST "" *
```

-  Chọn INBOX:
    

```
1 SELECT INBOX
```

- Search các file trong đó:
```
1 SEARCH ALL
```


-  Đọc email:
    

```
1 FETCH 1 all
```

hoặc:

```
1 FETCH 1 BODY[]
```

-  Thoát:
    

```
1 LOGOUT
```

## 5. Tương tác POP3 qua TLS

-  Kết nối:
    

```
openssl s_client -connect <target-ip>:pop3s
```

-  Đăng nhập:
    

```
USER <user>
PASS <password>
```

-  Xem số lượng email:
    

```
STAT
```

-  Liệt kê email:
    

```
LIST
```

-  Đọc email:
    

```
RETR 1
```

-  Thoát:
    

```
QUIT
```

## 6. Tìm thông tin nhạy cảm

-  Đọc INBOX.
    
-  Đọc folder `Important` nếu có.
    
-  Tìm:
    
```
password
credential
vpn
ssh
token
flag
backup
database
admin
```

## 7. Ghi chú report

-  Port nào mở.
    
-  Service/version.
    
-  TLS/certificate.
    
-  Login có thành công không.
    
-  Mailbox/folder nào tồn tại.
    
-  Email nào chứa thông tin nhạy cảm.