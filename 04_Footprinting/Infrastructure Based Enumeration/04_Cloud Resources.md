# Cloud Resources

## Ý chính

Nhiều công ty sử dụng cloud như AWS, GCP, Azure để lưu trữ và vận hành dịch vụ.

Cloud provider có thể bảo mật hạ tầng tốt, nhưng tài nguyên cloud vẫn có thể bị lộ do lỗi cấu hình của admin.

Các tài nguyên thường gặp:

- AWS S3 Bucket
- Azure Blob Storage
- GCP Cloud Storage

Nếu cấu hình sai, cloud storage có thể bị public và cho phép người ngoài truy cập file mà không cần xác thực.

## Mục tiêu recon

- Tìm cloud storage liên quan đến công ty
- Xác định bucket/container public
- Tìm file bị lộ
- Phân tích file nhạy cảm
- Ghi nhận cloud provider đang được sử dụng
- Phân biệt tài nguyên nằm trong scope và third-party

## Nguồn tìm cloud resources

### 1. DNS/Subdomain

Khi resolve subdomain, có thể thấy host trỏ tới cloud provider.

Ví dụ:

```text
s3-website-us-west-2.amazonaws.com
blob.core.windows.net
storage.googleapis.com
```

Command mẫu:

```bash
for i in $(cat subdomainlist); do  host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4done
```

### 2. Google Dorks

Tìm AWS S3:

```
intext:<company_name> inurl:amazonaws.com
```

Tìm Azure Blob:

```
intext:<company_name> inurl:blob.core.windows.net
```

Có thể kết hợp thêm:

```
filetype:pdffiletype:xlsxfiletype:docxfiletype:zipfiletype:sqlfiletype:env
```

### 3. Website Source Code

Kiểm tra HTML source để tìm URL cloud storage.

Ví dụ:

```html
<script src="https://example.blob.core.windows.net/app.js"></script><img src="https://bucket.s3.amazonaws.com/logo.png">
```

Các file như JS, CSS, image có thể được load từ cloud storage.

### 4. domain.glass

Dùng để tra cứu:

- IP information
- SSL certificate
- DNS info
- Cloudflare/security assessment
- Related infrastructure

Nếu thấy Cloudflare, ghi chú vào layer `Gateway`.

### 5. GrayHatWarfare

Dùng để tìm public bucket trên:

- AWS
- Azure
- GCP

Có thể lọc theo file format:

- pdf
- txt
- zip
- sql
- env
- pem
- key
- id_rsa

## File nhạy cảm cần chú ý

- `.env`
- `config.php`
- `database.sql`
- `backup.zip`
- `credentials.txt`
- `id_rsa`
- `id_rsa.pub`
- `.pem`
- `.key`
- source code
- internal documents
- API keys

## Rủi ro lớn

Nếu SSH private key bị lộ, attacker có thể dùng key đó để đăng nhập vào server mà không cần password.

Ví dụ file nguy hiểm:

```
id_rsa-----BEGIN RSA PRIVATE KEY-----...-----END RSA PRIVATE KEY-----
```

## Ghi nhớ

- Cloud không tự động an toàn.
- Lỗi thường nằm ở cấu hình của công ty, không phải cloud provider.
- Public bucket có thể làm lộ dữ liệu nghiêm trọng.
- Không kiểm thử/tải/xác minh tài nguyên ngoài scope.
- Không dùng leaked key để đăng nhập nếu không được phép.