# Virtual Hosts

## Ý chính

Virtual Hosting cho phép một web server như Apache, Nginx hoặc IIS host nhiều website/app trên cùng một server/IP.

Web server phân biệt website bằng `HTTP Host header`.

Ví dụ request:

```http
GET / HTTP/1.1
Host: forum.inlanefreight.htb
```
Web server đọc `Host` header rồi chọn VirtualHost tương ứng để trả content.
## Subdomain vs VHost

### Subdomain

Subdomain là domain con trong DNS.

Ví dụ:

```
blog.example.com
admin.example.com
dev.example.com
```

Subdomain thường có DNS record như `A`, `AAAA`, hoặc `CNAME`.

### Virtual Host

VHost là cấu hình trong web server.

Một VHost có thể tồn tại dù không có DNS record public.

Nếu biết hostname, có thể map thủ công trong `/etc/hosts`:

```
10.10.10.10 dev.example.com
```

## Server xử lý VHost như thế nào?

1. Browser gửi request đến IP.
2. Request chứa `Host` header.
3. Web server đọc `Host` header.
4. Web server tìm VirtualHost config khớp.
5. Web server trả nội dung từ `DocumentRoot` tương ứng.

## Các loại Virtual Hosting

|Loại|Cách hoạt động|Ghi chú|
|---|---|---|
|Name-Based|Dựa vào `Host header`|Phổ biến nhất|
|IP-Based|Mỗi website một IP|Tốn IP, tách biệt hơn|
|Port-Based|Mỗi website một port|Hay gặp ở dev/test|

## VHost Fuzzing

VHost fuzzing là kỹ thuật brute-force `Host header` để tìm virtual host ẩn.

Ví dụ thử:

```
admin.example.com
dev.example.com
staging.example.com
forum.example.com
```

## Tools

- `gobuster`
- `ffuf`
- `feroxbuster`

## Gobuster VHost

Command mẫu:

```
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

Ý nghĩa:

- `vhost`: chế độ tìm virtual host
- `-u`: target URL/IP
- `-w`: wordlist
- `--append-domain`: ghép domain vào từng word trong wordlist

Ví dụ tìm được:

```
Found: forum.inlanefreight.htb:81 Status: 200 [Size: 100]
```

## Ghi nhớ

DNS trả domain về IP, còn web server dùng `Host header` để quyết định trả website nào.

Trong HTB, nếu truy cập IP không thấy gì thú vị, hãy nghĩ đến:

- Subdomain enumeration
- VHost fuzzing
- Thêm hostname vào `/etc/hosts`
- Kiểm tra response khác biệt theo `Host header`

---
# Virtual Host Discovery Checklist  
  
## 1. Xác định target  
  
- [ ] Xác định IP target.  
- [ ] Xác định domain chính nếu có.  
- [ ] Kiểm tra port web đang mở.  
  
```bash  
nmap -sC -sV -p80,81,443,8080 <target-ip>
```
## 2. Thêm domain chính vào `/etc/hosts` nếu cần

```
sudo nano /etc/hosts
```

Thêm:

```
<target-ip> inlanefreight.htb
```

Ví dụ:

```
10.129.75.88 inlanefreight.htb
```

## 3. Kiểm tra response mặc định

- [ ]  Truy cập bằng IP.

```
curl -i http://<target-ip>
```

- [ ]  Truy cập bằng domain.

```
curl -i http://inlanefreight.htb
```

- [ ]  So sánh status code, size, title, redirect.

## 4. Thử Host header thủ công

```
curl -i http://<target-ip> -H "Host: test.inlanefreight.htb"
```

Nếu response khác với default, có thể có VHost.

## 5. Fuzz VHost bằng Gobuster

```
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

Hoặc dùng IP:

```
gobuster vhost -u http://<target-ip>:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --domain inlanefreight.htb --append-domain
```

Tùy version Gobuster, option có thể khác. Nếu `--domain` không hỗ trợ, dùng URL có domain và đảm bảo domain đã nằm trong `/etc/hosts`.

## 6. Tăng tốc nếu cần

```
gobuster vhost -u http://inlanefreight.htb:81 -w <wordlist> --append-domain -t 50
```

- [ ]  `-t 50`: tăng threads lên 50.

## 7. Bỏ qua lỗi TLS nếu HTTPS

```
gobuster vhost -u https://inlanefreight.htb -w <wordlist> --append-domain -k
```

- [ ]  `-k`: ignore SSL/TLS certificate errors.

## 8. Lưu output

```
gobuster vhost -u http://inlanefreight.htb:81 -w <wordlist> --append-domain -o vhosts.txt
```

## 9. Lọc false positive

- [ ]  Quan sát response size.
- [ ]  Nếu mọi kết quả có cùng size, có thể là false positive.
- [ ]  Dùng filter status/length nếu tool hỗ trợ.
- [ ]  Kiểm tra lại kết quả bằng `curl`.

Ví dụ:

```
curl -i http://inlanefreight.htb:81 -H "Host: forum.inlanefreight.htb"
```

## 10. Thêm VHost tìm được vào `/etc/hosts`

Nếu tìm được:

```
forum.inlanefreight.htb
```

thêm:

```
<target-ip> inlanefreight.htb forum.inlanefreight.htb
```

Sau đó truy cập browser:

```
http://forum.inlanefreight.htb:81
```

## 11. Kiểm tra sâu VHost tìm được

- [ ]  Xem title.
- [ ]  Xem công nghệ.
- [ ]  Kiểm tra login page.
- [ ]  Directory brute-force.
- [ ]  Check robots.txt.
- [ ]  Check source code comments.
- [ ]  Check exposed files.
- [ ]  Check default credentials nếu scope cho phép.

## 12. Ghi chú report/lab

- [ ]  VHost tìm được.
- [ ]  Port.
- [ ]  Status code.
- [ ]  Response size.
- [ ]  Screenshot nếu cần.
- [ ]  Công nghệ/service.
- [ ]  Rủi ro nếu có.