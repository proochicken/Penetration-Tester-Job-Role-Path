# Web Enumeration

## Mục tiêu
Sau khi phát hiện web server trên port 80/443 hoặc port web khác, cần enum web app để tìm:
- Hidden directories/files
- Admin panels
- CMS/framework/version
- Sensitive files
- Subdomains/vhosts
- Headers/certificates
- robots.txt
- Source code comments
- Possible credentials or misconfigurations

## Directory/File Enumeration với Gobuster

```bash
gobuster dir -u http://<target>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Ý nghĩa:
- `dir`: brute-force directory/file
- `-u`: target URL
- `-w`: wordlist

Status code cần chú ý:
- `200`: tồn tại và truy cập được
- `301/302`: redirect, tài nguyên có thể tồn tại
- `401`: cần xác thực
- `403`: tồn tại nhưng bị cấm truy cập
- `404`: không tồn tại
- `500`: server error

## DNS Subdomain Enumeration

```
gobuster dns -d <domain> -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

Dùng khi target có domain. Mục tiêu là tìm subdomain như:
- admin.example.com
- dev.example.com
- test.example.com
- api.example.com

## SecLists

Cài SecLists:
```
sudo apt install seclists -y
```

Hoặc:
```
git clone https://github.com/danielmiessler/SecLists
```

## Web Headers

```
curl -IL http://<target>/
```

Dùng để xem:

- Server
- Framework/CMS dấu hiệu
- Redirect
- Content-Type
- Security headers
- WordPress REST API link

## WhatWeb

```
whatweb http://<target>/
```

Scan cả subnet:

```
whatweb --no-errors 10.10.10.0/24
```

Dùng để fingerprint:

- Web server
- OS hint
- Framework
- CMS
- Page title
- Email leak
- PHP version

## Certificates

Nếu có HTTPS, kiểm tra certificate để tìm:

- Domain/subdomain
- Organization
- Email
- Common Name
- Subject Alternative Names

## robots.txt

Kiểm tra:

```
http://<target>/robots.txt
```

Có thể lộ:

- /admin
- /private
- /uploaded_files
- /backup

## Source Code

Kiểm tra source bằng:

- Browser: `Ctrl + U`
- CLI: `curl http://<target>/`

Tìm:

- Developer comments
- Test credentials
- Hidden endpoints
- API paths
- JS files
- TODO/FIXME

---

# Checklist làm lab
## A. Khi thấy port web mởVí dụ từ Nmap:
```text
80/tcp open http
443/tcp open https
8080/tcp open http-proxy
8000/tcp open http
```

Truy cập bằng browser:

```
http://<target>/https://<target>/http://<target>:8080/
```

## B. Lấy header

```
curl -IL http://<target>/
```

Ghi lại:

- Server/version.
- Redirect.
- Cookie.
- Framework hint.
- CMS hint.
- Security headers thiếu hay có.
- `X-Powered-By`.
- `Link: ... wp-json` nếu là WordPress.

---

## C. Fingerprint công nghệ

```
whatweb http://<target>/
```

Nếu có HTTPS:

```
whatweb https://<target>/
```

Cần ghi lại:

- Apache/Nginx/IIS.
- PHP/Python/Node/ASP.NET.
- CMS.
- Title.
- Email.
- Framework.
- Version.

---

## D. Kiểm tra các file mặc định

Truy cập thủ công:

```
/robots.txt/sitemap.xml/favicon.ico
```

Các path nên thử nhanh:

```
/admin
/login
/dashboard
/wp-admin
/wp-login.php
/phpinfo.php
/info.php
/server-status
/backup
/backups
/uploads
/upload
/private
```

---

## E. Directory brute-force

Chạy Gobuster cơ bản:

```
gobuster dir -u http://<target>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Nếu muốn thêm extension PHP/TXT/BAK:

```
gobuster dir -u http://<target>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,bak,old,zip
```

Nếu target trả response lạ, thêm status code cần lọc:

```
gobuster dir -u http://<target>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,bak -t 30
```

---

## F. Nếu tìm thấy CMS WordPress

Kiểm tra:

```
/wp-admin
/wp-login.php
/wp-content/
/wp-content/plugins/
/wp-content/themes/
/wp-json/
```

Dùng curl:

```
curl http://<target>/wp-json/
```

Hoặc dùng whatweb:

```
whatweb http://<target>/wordpress/
```

---

## G. Kiểm tra source code

Trên browser:

```
Ctrl + U
```

Hoặc CLI:

```
curl -s http://<target>/ | less
```

Tìm keyword:

```
password
user
admin
test
dev
todo
fixme
api
token
key
secret
```

Có thể dùng:

```
curl -s http://<target>/ | grep -iE 'password|user|admin|test|dev|todo|token|secret'
```

---

## H. HTTPS certificate

Nếu có HTTPS, kiểm tra certificate bằng browser hoặc openssl:

```
openssl s_client -connect <target>:443 -showcerts
```

Tìm:

- Common Name.
- Organization.
- Email.
- Subject Alternative Names.
- Domain/subdomain.

---

## I. DNS/Subdomain enumeration

Chỉ áp dụng khi có domain, ví dụ:

```
inlanefreight.com
```

Chạy:

```
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

Nếu trong HTB cần map domain về IP:

```
sudo nano /etc/hosts
```

Thêm:

```
10.10.10.121 inlanefreight.htb
```

Sau đó enum tiếp.

---
# Mindset 
1. Web server là gì?
2. Công nghệ/framework/CMS nào?
3. Header có lộ version không?
4. robots.txt có path nhạy cảm không?
5. Source code có comment/credential/endpoint không?
6. Gobuster tìm được path nào?
7. Có admin panel/login/upload không?
8. Có HTTPS certificate tiết lộ domain/subdomain không?
9. Có WordPress/CMS setup mode không?
10. Có file backup/config/phpinfo bị lộ không?