# Crawling

## Ý chính

Crawling, còn gọi là spidering, là quá trình tự động duyệt website bằng cách đi theo các link có sẵn.

Crawler bắt đầu từ một seed URL, tải nội dung trang, parse HTML, trích xuất link, đưa link mới vào queue và tiếp tục crawl.

## Crawling khác Fuzzing

| Kỹ thuật | Cách hoạt động |
|---|---|
| Crawling | Đi theo link thật đã tồn tại trên website |
| Fuzzing | Đoán/brute-force path bằng wordlist |

Ví dụ:

```text
Crawling: thấy link /admin trong HTML rồi truy cập
Fuzzing: đoán /admin dù website không link đến
```
## Quy trình Crawling

```
Seed URL
  ↓
Fetch page
  ↓
Parse content
  ↓
Extract links
  ↓
Add links to queue
  ↓
Repeat
```

## Chiến lược Crawling

### Breadth-First Crawling

Đi rộng trước, sâu sau.

- Crawl hết link cấp 1 trước
- Sau đó mới crawl link bên trong
- Phù hợp để có overview cấu trúc website

### Depth-First Crawling

Đi sâu trước, rộng sau.

- Theo một nhánh càng sâu càng tốt
- Sau đó quay lại nhánh khác
- Phù hợp để tìm nội dung nằm sâu

## Dữ liệu crawler có thể thu thập

- Internal links
- External links
- Comments
- Metadata
- Sensitive files
- Backup files
- Config files
- Log files
- API endpoints
- Forms
- JavaScript files

## File nhạy cảm cần chú ý

```
.env
config.php
settings.php
web.config
backup.zip
database.sql
error_log
access_log
*.bak
*.old
```

Các file này có thể chứa:

- Database credentials
- API keys
- Encryption keys
- Source code
- Internal paths
- Debug information

## Tầm quan trọng của context

Một mẩu thông tin riêng lẻ có thể chưa nguy hiểm, nhưng khi kết hợp với thông tin khác có thể thành finding quan trọng.

Ví dụ:

```
Comment nhắc tới "file server"
+
Crawler tìm thấy /files/
+
/files/ bật directory browsing
=
Có khả năng lộ dữ liệu nhạy cảm
```

## Ghi nhớ

Crawling giúp map website dựa trên link thật. Nhưng crawling không tìm được những path không được link ra. Vì vậy nên kết hợp:

1. Crawling
2. Fuzzing
3. JavaScript analysis
4. Manual review
5. Contextual analysis

---
# Crawling Checklist  
  
## 1. Chuẩn bị  
  
- [ ] Xác định target trong scope.  
- [ ] Xác định seed URL.  
- [ ] Xác định có cần login/session không.  
- [ ] Kiểm tra robots.txt.  
- [ ] Lưu output crawling.  
  
Ví dụ seed URL:  
  
```text  
http://target.htb/  
https://app.target.htb/
```
## 2. Crawl bằng browser/proxy

- [ ]  Mở Burp Suite hoặc OWASP ZAP.
- [ ]  Cấu hình browser proxy.
- [ ]  Duyệt website thủ công.
- [ ]  Đăng nhập nếu có tài khoản test.
- [ ]  Click các chức năng chính.
- [ ]  Ghi lại sitemap/HTTP history.

## 3. Crawl tự động

- [ ]  Dùng crawler nếu scope cho phép.
- [ ]  Giới hạn depth để tránh scan quá sâu.
- [ ]  Giới hạn domain để không crawl ra ngoài scope.
- [ ]  Lưu URL tìm được.

Ví dụ tools có thể dùng:

```
Burp Crawler
OWASP ZAP Spider
katana
hakrawler
gospider
```

## 4. Thu thập link

- [ ]  Internal links.
- [ ]  External links.
- [ ]  API endpoints.
- [ ]  Form actions.
- [ ]  JavaScript files.
- [ ]  File download links.
- [ ]  Hidden links trong HTML comments.

## 5. Kiểm tra JavaScript

- [ ]  Tải các file `.js`.
- [ ]  Tìm endpoint API.
- [ ]  Tìm route ẩn.
- [ ]  Tìm token/key hardcoded.
- [ ]  Tìm URL nội bộ.

Từ khóa nên grep:

```
api
token
key
secret
admin
debug
internal
localhost
staging
dev
```

## 6. Tìm file nhạy cảm

- [ ]  Backup files.

```
.bak
.old
.zip
.tar.gz
.sql
```

- [ ]  Config files.

```
.env
config.php
settings.php
web.config
```

- [ ]  Logs.

```
error_log
access_log
debug.log.log
```

## 7. Kiểm tra directory browsing

- [ ]  Truy cập các thư mục đáng chú ý.

```
/files/
/uploads/
/backup/
/docs/
/download/
/assets/
```

- [ ]  Nếu thấy `Index of /...`, kiểm tra có file nhạy cảm không.
- [ ]  Không tải dữ liệu ngoài scope hoặc dữ liệu cá nhân không cần thiết.

## 8. Phân tích metadata/comments

- [ ]  Page title.
- [ ]  Meta generator.
- [ ]  Author.
- [ ]  Version.
- [ ]  HTML comments.
- [ ]  Date/last modified.

## 9. Kết hợp context

- [ ]  Nối các thông tin rời rạc.
- [ ]  Link comment với directory/file tìm được.
- [ ]  Link metadata version với CVE/misconfig.
- [ ]  Link endpoint API với chức năng web.
- [ ]  Link external domains với third-party services.

## 10. Ghi chú report

- [ ]  URL tìm được.
- [ ]  Nguồn phát hiện: crawling/manual/JS.
- [ ]  Bằng chứng.
- [ ]  Rủi ro thực tế.
- [ ]  Cách reproduce.
- [ ]  Không report chỉ vì URL tồn tại; cần impact.