# Creepy Crawlies - Web Crawling Tools

## Ý chính

Web crawling giúp tự động duyệt website và thu thập dữ liệu recon. Thay vì click thủ công từng page, crawler sẽ đi theo link, parse nội dung và lưu lại các thông tin hữu ích.

## Tools phổ biến

| Tool | Công dụng |
|---|---|
| Burp Suite Crawler | Crawl/map web app, kết hợp proxy và testing |
| OWASP ZAP Spider | Crawler miễn phí, open-source |
| Scrapy | Python framework để viết custom crawler |
| Apache Nutch | Scalable crawler cho crawl quy mô lớn |

## Ethical Crawling

Khi crawl cần:

- Có permission
- Giới hạn scope
- Không gửi request quá mức
- Không làm quá tải server
- Không thu thập dữ liệu ngoài phạm vi được phép

## Scrapy

Cài Scrapy:

```bash
pip3 install scrapy
```
Tải ReconSpider:

```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
```

Chạy ReconSpider:

```
python3 ReconSpider.py http://inlanefreight.com
```

## Output

Kết quả được lưu vào:

```
results.json
```

Cấu trúc chính:

```
{
  "emails": [],
  "links": [],
  "external_files": [],
  "js_files": [],
  "form_fields": [],
  "images": [],
  "videos": [],
  "audio": [],
  "comments": []
}
```

## Ý nghĩa các field

|Key|Ý nghĩa|
|---|---|
|`emails`|Email tìm thấy trên website|
|`links`|URL/link crawl được|
|`external_files`|File như PDF/DOCX/ZIP|
|`js_files`|JavaScript files|
|`form_fields`|Input fields trong form|
|`images`|Image URLs|
|`videos`|Video URLs|
|`audio`|Audio URLs|
|`comments`|HTML comments|

## Dữ liệu cần ưu tiên phân tích

1. `links`
2. `js_files`
3. `form_fields`
4. `external_files`
5. `comments`
6. `emails`

## Ghi nhớ

Crawler chỉ thu thập dữ liệu. Việc quan trọng là phân tích output:

- Link nào là login/admin/api?
- JS có lộ endpoint/token không?
- Form nào có input user-controlled?
- File nào có thể chứa metadata/secret?
- Comment nào lộ thông tin nội bộ?

---
# Creepy Crawlies Lab Checklist  
  
## 1. Chuẩn bị target  
  
- [ ] Xác định domain/IP trong scope.  
- [ ] Kiểm tra website truy cập được.  
- [ ] Nếu là vhost/lab, thêm `/etc/hosts` nếu cần.  
- [ ] Xác định HTTP hay HTTPS.  
- [ ] Tạo thư mục lưu output.  
  
```bash  
mkdir crawl-recon  
cd crawl-recon
```
## 2. Cài Scrapy

```
pip3 install scrapy
```

Kiểm tra:

```
	python3 -c "import scrapy; print(scrapy.__version__)"
```

## 3. Tải ReconSpider

```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip
```

Kiểm tra file:

```
ls -la
```

## 4. Chạy ReconSpider

```
python3 ReconSpider.py http://target.com
```

Ví dụ lab:

```
python3 ReconSpider.py http://inlanefreight.com
```

## 5. Kiểm tra output

```
ls -la results.json
cat results.json
```

Format đẹp hơn:

```
jq . results.json
```

## 6. Trích xuất dữ liệu quan trọng

### Links

```
jq -r '.links[]?' results.json | sort -u
```

### JavaScript files

```
jq -r '.js_files[]?' results.json | sort -u
```

### Emails

```
jq -r '.emails[]?' results.json | sort -u
```

### External files

```
jq -r '.external_files[]?' results.json | sort -u
```

### Form fields

```
jq -r '.form_fields[]?' results.json
```

### Comments

```
jq -r '.comments[]?' results.json
```

## 7. Phân tích links

- [ ]  Tìm admin/login.

```
jq -r '.links[]?' results.json | grep -Ei "admin|login|signin|auth|portal"
```

- [ ]  Tìm API.

```
jq -r '.links[]?' results.json | grep -Ei "/api/|graphql|rest"
```

- [ ]  Tìm upload/download/file.

```
jq -r '.links[]?' results.json | grep -Ei "upload|download|file|files|docs"
```

## 8. Phân tích JS files

- [ ]  Tải JS đáng chú ý.
- [ ]  Tìm endpoint.
- [ ]  Tìm secret/token/key.
- [ ]  Tìm route ẩn.

Từ khóa grep:

```
api|token|secret|key|debug|admin|internal|localhost|staging|dev
```

## 9. Phân tích external files

- [ ]  Kiểm tra PDF/DOCX/XLSX.
- [ ]  Kiểm tra metadata nếu scope cho phép.
- [ ]  Tìm file backup/archive.
- [ ]  Không tải dữ liệu nhạy cảm ngoài phạm vi cần thiết.

## 10. Phân tích form fields

- [ ]  Xác định chức năng form.
- [ ]  Field nào user-controlled?
- [ ]  Có CSRF token không?
- [ ]  Có upload field không?
- [ ]  Có search/filter/id parameter không?

## 11. Phân tích comments

- [ ]  Tìm endpoint ẩn.
- [ ]  Tìm TODO/debug/internal note.
- [ ]  Tìm version/công nghệ.
- [ ]  Tìm path/file bị comment lại.

## 12. Ghi chú kết quả

- [ ]  URL quan trọng.
- [ ]  JS file đáng chú ý.
- [ ]  Form/input cần test.
- [ ]  File/tài liệu cần kiểm tra.
- [ ]  Comment có giá trị.
- [ ]  Hướng kiểm thử tiếp theo.