# FTP Enumeration

## Tổng quan

FTP là giao thức truyền file giữa client và server. FTP hoạt động ở tầng Application Layer và thường dùng TCP port `21` cho control channel.

FTP sử dụng 2 kênh:

```text
Control Channel: gửi lệnh, nhận response code
Data Channel: truyền file hoặc directory listing
```

FTP mặc định là clear-text protocol, nghĩa là username/password và dữ liệu có thể bị sniff nếu traffic không được mã hóa.

## Active FTP vs Passive FTP

### Active FTP

Client kết nối tới server port `21`, sau đó server kết nối ngược lại client để truyền data.

Vấn đề: dễ bị firewall/NAT phía client chặn.

### Passive FTP

Server mở một port data và báo cho client. Client chủ động kết nối vào port đó.

Passive mode thường dễ hoạt động hơn khi có firewall.

## Anonymous FTP

Anonymous FTP cho phép đăng nhập không cần tài khoản thật.

Thông tin đăng nhập thường dùng:

```text
username: anonymous
password: anonymous
```

Cần kiểm tra:

```text
Có login được không?
Có list được file không?
Có download được file không?
Có upload được file không?
Có tạo thư mục được không?
FTP folder có liên quan tới web root không?
```

## TFTP

TFTP là phiên bản đơn giản hơn FTP.

Khác biệt chính:

```text
FTP dùng TCP, TFTP dùng UDP
FTP có authentication, TFTP không có authentication
FTP hỗ trợ directory listing, TFTP không hỗ trợ directory listing
```

TFTP thường chỉ nên dùng trong mạng nội bộ.

## vsFTPd

vsFTPd là FTP server phổ biến trên Linux.

File cấu hình chính:

```bash
/etc/vsftpd.conf
```

Một số setting quan trọng:

```text
anonymous_enable=YES/NO
local_enable=YES/NO
write_enable=YES/NO
anon_upload_enable=YES/NO
anon_mkdir_write_enable=YES/NO
chroot_local_user=YES/NO
hide_ids=YES/NO
ls_recurse_enable=YES/NO
ssl_enable=YES/NO
```

File deny user FTP:

```bash
/etc/ftpusers
```

User nằm trong file này sẽ không được login FTP.

## Dangerous Settings

Các setting nguy hiểm:

```text
anonymous_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
no_anon_password=YES
write_enable=YES
```

Rủi ro:

```text
Lộ file nội bộ
Anonymous user upload được file
Tạo được thư mục
Ghi/xóa/rename file
Có thể dẫn tới RCE nếu FTP folder liên kết với web server
```

## FTP Enumeration Commands

Kết nối FTP:

```bash
ftp <target-ip>
```

Đăng nhập anonymous:

```text
Name: anonymous
Password: anonymous
```

Liệt kê file:

```bash
ls
```

Liệt kê đệ quy:

```bash
ls -R
```

Xem trạng thái FTP session:

```bash
status
```

Bật debug/trace:

```bash
debug
trace
```

Tải file:

```bash
get "Important Notes.txt"
```

Upload file test:

```bash
touch testupload.txt
put testupload.txt
```

Tải toàn bộ file FTP bằng wget:

```bash
wget -m --no-passive ftp://anonymous:anonymous@<target-ip>
```

Scan FTP bằng Nmap:

```bash
sudo nmap -sV -p21 -sC -A <target-ip>
```

Update NSE script database:

```bash
sudo nmap --script-updatedb
```

Tìm FTP NSE scripts:

```bash
find / -type f -name "ftp*" 2>/dev/null | grep scripts
```

Trace NSE script:

```bash
sudo nmap -sV -p21 -sC -A <target-ip> --script-trace
```

Banner grabbing bằng nc/telnet:

```bash
nc -nv <target-ip> 21
telnet <target-ip> 21
```

Kiểm tra FTP có TLS/SSL:

```bash
openssl s_client -connect <target-ip>:21 -starttls ftp
```

## Tư duy pentest

Khi thấy FTP mở port 21:
1. Lấy banner/version.
2. Kiểm tra anonymous login.
3. List file/thư mục.
4. Tìm file nhạy cảm.
5. Download file quan trọng.
6. Kiểm tra quyền upload/write.
7. Nếu upload được, kiểm tra FTP có liên kết với web root không.
8. Dùng Nmap NSE để tự động phát hiện anonymous, writable, STAT info.
9. Nếu FTP dùng SSL/TLS, xem certificate để lấy hostname/domain/email.
10. Ghi chú mọi thông tin có thể dùng cho bước khai thác tiếp theo.

---

# FTP Lab Checklist

## 1. Xác định service

- [ ] Scan port FTP:

```bash
sudo nmap -sV -p21 <target-ip>
```
-  Ghi lại banner nếu có.
## 2. Chạy default script scan
-  Dùng Nmap `-sC`:
```
sudo nmap -sV -sC -p21 <target-ip>
```
-  Kiểm tra có anonymous login không:
```
ftp-anon: Anonymous FTP login allowed
```
-  Kiểm tra có file/thư mục writable không:
```
[NSE: writeable]
```
## 3. Kết nối thủ công bằng ftp client
-  Kết nối:
```
ftp <target-ip>
```
-  Thử anonymous login:
```
Name: anonymous
Password: anonymous
```
-  Nếu login thành công, chạy:
```
ls
pwd
status
```
## 4. Enumeration thư mục/file
-  Liệt kê thư mục hiện tại:
```
ls
```
-  Nếu được phép, liệt kê đệ quy:
```
ls -R
```
-  Chú ý các file có tên đáng ngờ:
```
backup
config
password
credential
user
admin
private
key
note
important
```
-  Ghi lại owner/permission/file size/timestamp.
## 5. Download file
-  Download từng file quan trọng:
```
get filename
```
-  Với file có dấu cách:
    
```
get "Important Notes.txt"
```
hoặc:
```
get Important\ Notes.txt
```
-  Kiểm tra file local:
```
ls -la
file <filename>
cat <filename>
strings <filename>
```

## 6. Mirror toàn bộ FTP nếu phù hợp
-  Nếu số lượng file vừa phải và scope cho phép:
    
```
wget -m --no-passive ftp://anonymous:anonymous@<target-ip>
```

-  Xem cây thư mục sau khi tải:
    
```
tree <target-ip>
```

-  Cẩn thận: tải toàn bộ có thể gây ồn trong môi trường thật.
    
## 7. Kiểm tra upload/write permission
-  Tạo file test vô hại:
    
```
touch testupload.txt
```

-  Upload:
    
```
put testupload.txt
```

-  List lại để xác nhận:
    
```
ls
```

-  Nếu upload được, kiểm tra permission của file vừa upload.
## 8. Kiểm tra liên kết với web server
-  Nếu cùng target có HTTP/HTTPS, kiểm tra xem file upload có truy cập được qua web không.
-  Ví dụ tư duy:
```
FTP upload: testupload.txt
Web check: http://<target>/testupload.txt
```

-  Nếu truy cập được qua web, đây là hướng khai thác rất quan trọng.
## 9. Tương tác service thủ công

-  Banner grabbing bằng netcat:
    
```
nc -nv <target-ip> 21
```

-  Hoặc telnet:
    
```
telnet <target-ip> 21
```

## 10. Nếu FTP dùng TLS/SSL

-  Kiểm tra certificate:
    
```
openssl s_client -connect <target-ip>:21 -starttls ftp
```

-  Ghi lại:
    
```
CN
hostname
domain
email
organization
location
```

## 11. Tìm script NSE liên quan FTP
-  Update NSE database:
    
```
sudo nmap --script-updatedb
```

-  Tìm FTP scripts:
    
```
find / -type f -name "ftp*" 2>/dev/null | grep scripts
```

-  Có thể chú ý:
    
```
ftp-anon
ftp-syst
ftp-brute
ftp-vsftpd-backdoor
ftp-proftpd-backdoor
ftp-bounce
```

## 12. Kết luận phát hiện
-  FTP có clear-text không?
-  Có anonymous login không?
-  Có download được file không?
-  Có upload được file không?
-  Có writable directory không?
-  Có version vulnerable không?
-  Có lộ hostname/domain/email qua banner/certificate không?
-  Có thể chain với web service không?