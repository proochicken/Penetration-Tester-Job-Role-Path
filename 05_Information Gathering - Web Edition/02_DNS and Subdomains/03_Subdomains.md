# Subdomains

## Ý chính

Subdomain là domain con của domain chính.

Ví dụ:

```text
example.com
www.example.com
api.example.com
admin.example.com
dev.example.com
staging.example.com
```
Trong web recon, subdomain rất quan trọng vì chúng mở rộng attack surface và có thể chứa hệ thống không được link từ website chính.
## Vì sao subdomain quan trọng?

Subdomain có thể chứa:

- Development/Staging environments
- Hidden login portals
- Legacy applications
- Sensitive information
- Admin panels
- APIs
- VPN/SSO portals
- Old services

Các môi trường như `dev`, `staging`, `test`, `old` thường có rủi ro cao vì có thể bảo mật yếu hơn production.

## Subdomain Enumeration

Subdomain enumeration là quá trình tìm và liệt kê subdomain của một domain.

Về DNS, subdomain thường được biểu diễn qua:

- `A record`: hostname -> IPv4
- `AAAA record`: hostname -> IPv6
- `CNAME record`: hostname alias -> hostname khác

## Active Enumeration

Active enumeration tương tác trực tiếp với target/DNS server.

Ví dụ:

- DNS zone transfer
- DNS brute-force
- Query DNS server

Ưu điểm:

- Chủ động
- Có thể tìm được subdomain mới

Nhược điểm:

- Có thể bị log/detect
- Có thể gây noise
- Cần đảm bảo scope cho phép

## Passive Enumeration

Passive enumeration dùng nguồn public/bên thứ ba.

Ví dụ:

- Certificate Transparency logs
- Search engines
- Public DNS databases
- Online OSINT tools

Ưu điểm:

- Stealthier
- Ít tương tác trực tiếp với target

Nhược điểm:

- Dữ liệu có thể cũ
- Có thể thiếu subdomain
- Cần validate lại

## Subdomain Brute-force Process

1. Chọn wordlist
2. Ghép từng từ với domain chính
3. DNS lookup từng subdomain
4. Lọc và validate kết quả

Ví dụ wordlist:

```
www
mail
admin
dev
test
staging
api
portal
vpn
```

Target:

```
example.com
```

Tool sẽ thử:

```
www.example.com
mail.example.com
admin.example.com
dev.example.com
```

## Tools

- `dnsenum`
- `fierce`
- `dnsrecon`
- `amass`
- `assetfinder`
- `puredns`
- `ffuf`
- `gobuster`

## DNSEnum

`dnsenum` là tool DNS reconnaissance viết bằng Perl.

Nó hỗ trợ:

- DNS record enumeration
- Zone transfer attempts
- Subdomain brute-forcing
- Google scraping
- Reverse lookup
- WHOIS lookups

Command ví dụ:

```
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

Ý nghĩa:

- `--enum`: bật chế độ enum tổng hợp
- `inlanefreight.com`: target domain
- `-f`: chỉ định wordlist
- `-r`: recursive brute-force

## Ghi nhớ

Subdomain enumeration là bước rất quan trọng trong recon vì nhiều lỗi không nằm ở domain chính mà nằm ở subdomain phụ như `dev`, `staging`, `old`, `admin`, `api`.

Luôn kết hợp passive + active enumeration và validate lại kết quả để tránh false positive.

---

# Subdomain Enumeration Checklist  
  
## 1. Chuẩn bị  
  
- [ ] Xác định domain trong scope.  
- [ ] Tạo thư mục lưu kết quả.  
- [ ] Kiểm tra tool cần dùng.  
- [ ] Kiểm tra wordlist có sẵn không.  
  
```bash  
ls /usr/share/seclists/Discovery/DNS/
```
## 2. Passive Enumeration trước

- [ ]  Tìm subdomain qua Certificate Transparency logs.
- [ ]  Tìm bằng search engine.

```
site:example.com
site:example.com -www
```

- [ ]  Tìm bằng các nguồn public như crt.sh, SecurityTrails, VirusTotal nếu được phép.
- [ ]  Lưu danh sách subdomain tìm được.

## 3. Active Enumeration

- [ ]  Kiểm tra NS record.

```
dig NS example.com
```

- [ ]  Thử zone transfer nếu trong scope.

```
dig axfr example.com @ns1.example.com
```

- [ ]  Brute-force subdomain bằng wordlist.

```
dnsenum --enum example.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

## 4. Validate kết quả

- [ ]  Resolve subdomain để lấy IP.

```
dig A sub.example.com
dig AAAA sub.example.com
```

- [ ]  Kiểm tra CNAME.

```
dig CNAME sub.example.com
```

- [ ]  Lọc trùng.
- [ ]  Kiểm tra wildcard DNS.
- [ ]  Kiểm tra subdomain còn sống không.
- [ ]  Kiểm tra HTTP/HTTPS service.

## 5. Phân loại subdomain theo rủi ro

Ưu tiên kiểm tra các subdomain như:

- [ ]  `admin`
- [ ]  `dev`
- [ ]  `staging`
- [ ]  `test`
- [ ]  `old`
- [ ]  `backup`
- [ ]  `api`
- [ ]  `portal`
- [ ]  `vpn`
- [ ]  `sso`
- [ ]  `git`
- [ ]  `jenkins`
- [ ]  `jira`
- [ ]  `grafana`
- [ ]  `kibana`

## 6. Kiểm tra công nghệ và service

- [ ]  HTTP title.
- [ ]  Status code.
- [ ]  Redirect.
- [ ]  Login page.
- [ ]  Framework.
- [ ]  Server header.
- [ ]  Exposed files.
- [ ]  API docs.
- [ ]  Debug page.

## 7. Ghi chú phục vụ report

- [ ]  Lưu raw output của tool.
- [ ]  Lưu danh sách subdomain valid.
- [ ]  Map subdomain -> IP.
- [ ]  Map subdomain -> technology.
- [ ]  Ghi lại subdomain có rủi ro cao.
- [ ]  Không report subdomain chỉ vì nó tồn tại; phải gắn với rủi ro thực tế.