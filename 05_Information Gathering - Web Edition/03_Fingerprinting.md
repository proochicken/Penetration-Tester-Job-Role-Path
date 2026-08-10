# Fingerprinting

## Ý chính

Fingerprinting là quá trình thu thập dấu hiệu kỹ thuật của website/web app để xác định target đang dùng công nghệ gì.

Có thể fingerprint:

- Web server: Apache, Nginx, IIS
- OS: Ubuntu, Debian, Windows Server
- CMS: WordPress, Drupal, Joomla
- Framework: Laravel, Django, Express, ASP.NET
- WAF: Wordfence, Cloudflare, Akamai, ModSecurity
- Version phần mềm
- HTTP headers
- File/endpoint đặc trưng

## Vì sao quan trọng?

Fingerprinting giúp pentester:

- Tấn công/kiểm thử có định hướng
- Tìm công nghệ có CVE hoặc misconfig
- Ưu tiên target rủi ro cao
- Xây dựng profile tổng quan về hạ tầng target

## Kỹ thuật fingerprinting

### Banner Grabbing

Lấy banner từ service/web server.

```bash
curl -I inlanefreight.com
```
Ví dụ:

```
Server: Apache/2.4.41 (Ubuntu)
```

### HTTP Headers Analysis

Phân tích header để tìm công nghệ.

```
X-Redirect-By: WordPress
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
```

Dấu hiệu này cho thấy website dùng WordPress.

### Probing Specific Responses

Gửi request tới endpoint/file đặc trưng:

```
/wp-login.php
/wp-json/
/xmlrpc.php
/license.txt
/server-status
```

### Page Content Analysis

Xem HTML/source code để tìm:

```
wp-content
wp-includes
generator meta tag
framework assets
comments
copyright
```

## Tools

| Tool       | Công dụng                                   |
| ---------- | ------------------------------------------- |
| Wappalyzer | Browser extension nhận diện công nghệ web   |
| BuiltWith  | Báo cáo technology stack                    |
| WhatWeb    | CLI fingerprinting                          |
| Nmap       | Service/version/OS detection                |
| Netcraft   | Website/hosting profiling                   |
| wafw00f    | Phát hiện WAF                               |
| Nikto      | Web server scanner/fingerprinting/misconfig |

## curl fingerprinting

```
curl -I inlanefreight.com
curl -I https://inlanefreight.com
curl -I https://www.inlanefreight.com
```

Kết quả đáng chú ý:

```
Apache/2.4.41 (Ubuntu)
X-Redirect-By: WordPress
/wp-json/
```

## WAF detection

Cài wafw00f:

```
pip3 install git+https://github.com/EnableSecurity/wafw00f
```

Chạy:

```
wafw00f inlanefreight.com
```

Kết quả:

```
Wordfence (Defiant) WAF
```

## Nikto fingerprinting

```
nikto -h inlanefreight.com -Tuning b
```

Ý nghĩa:

- `-h`: target host
- `-Tuning b`: chỉ chạy software identification modules

Finding chính:

- Apache/2.4.41 Ubuntu
- WordPress installation
- `/wp-login.php`
- `/license.txt`
- Missing HSTS
- Missing X-Content-Type-Options
- Cookie thiếu HttpOnly
- Apache version có vẻ outdated

## Ghi nhớ

Fingerprinting không phải exploit trực tiếp. Nó giúp biết target đang dùng gì để chọn hướng kiểm thử tiếp theo.

Quy trình tốt:

1. Check headers bằng `curl -I`
2. Follow redirect HTTP -> HTTPS -> www
3. Detect CMS/framework
4. Detect WAF bằng `wafw00f`
5. Scan fingerprint/misconfig bằng `Nikto`
6. Validate finding thủ công
7. Ưu tiên công nghệ/version/endpoint có rủi ro cao

---
# Fingerprinting Checklist  
  
## 1. Xác định target  
  
- [ ] Có domain/IP trong scope.  
- [ ] Xác định port web đang mở.  
  
```bash  
nmap -sC -sV -p80,443,8080,8000,8443 <target>
```
## 2. Lấy HTTP headers

- [ ]  Kiểm tra HTTP.

```
curl -I http://<target>
```

- [ ]  Kiểm tra HTTPS.

```
curl -Ik https://<target>
```

- [ ]  Follow redirect nếu cần.

```
curl -IL http://<target>
```

Ghi lại:

- [ ]  `Server`
- [ ]  `X-Powered-By`
- [ ]  `X-Redirect-By`
- [ ]  `Location`
- [ ]  `Set-Cookie`
- [ ]  `Link`
- [ ]  `Content-Type`
- [ ]  Security headers

## 3. Kiểm tra công nghệ web

- [ ]  Xem source HTML.

```
curl -s https://<target> | head
```

- [ ]  Tìm dấu hiệu CMS/framework.

```
curl -s https://<target> | grep -Ei "wp-content|wp-includes|drupal|joomla|laravel|django|express|asp.net"
```

- [ ]  Kiểm tra endpoint đặc trưng.

```
/wp-json/
/wp-login.php
/xmlrpc.php
/robots.txt
/license.txt
```

## 4. Detect WAF

```
wafw00f <target>
```

Ghi lại:

- [ ]  Có WAF không?
- [ ]  WAF loại gì?
- [ ]  Có thể ảnh hưởng scan không?

## 5. Nikto software identification

```
nikto -h <target> -Tuning b
```

Ghi lại:

- [ ]  Server/version
- [ ]  CMS
- [ ]  File mặc định
- [ ]  Header thiếu
- [ ]  Cookie flags
- [ ]  Login page
- [ ]  Finding cần verify

## 6. Dùng tool bổ sung nếu cần

- [ ]  Wappalyzer
- [ ]  WhatWeb

```
whatweb <target>
```

- [ ]  Nmap NSE scripts nếu cần.

```
nmap -sV --script=http-title,http-server-header <target>
```

## 7. Validate finding thủ công

- [ ]  Không tin tuyệt đối output tool.
- [ ]  Kiểm tra lại bằng curl/browser.
- [ ]  Phân biệt info disclosure với vulnerability thật.
- [ ]  Kiểm tra version có thực sự vulnerable không.
- [ ]  Tìm exploit/CVE chỉ sau khi xác nhận version và context.

## 8. Ưu tiên bước tiếp theo

Nếu thấy WordPress:

- [ ]  Kiểm tra `/wp-login.php`
- [ ]  Kiểm tra `/wp-json/`
- [ ]  Enumerate plugins/themes nếu scope cho phép
- [ ]  Check XML-RPC
- [ ]  Check users leak

Nếu thấy Apache/Nginx version cũ:

- [ ]  Kiểm tra CVE liên quan
- [ ]  Kiểm tra config issue
- [ ]  Kiểm tra default files

Nếu thiếu security headers:

- [ ]  Ghi nhận mức độ phù hợp
- [ ]  Không overclaim impact