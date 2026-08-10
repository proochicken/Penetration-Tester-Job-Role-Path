# Domain Information

## Mục tiêu

Domain Information là bước thu thập thông tin về sự hiện diện của công ty trên Internet.

Không chỉ tìm subdomain, mà cần hiểu:

- Công ty cung cấp dịch vụ gì
- Họ cần hạ tầng/công nghệ nào để vận hành dịch vụ đó
- Domain/subdomain nào đang tồn tại
- IP nào liên quan
- DNS record tiết lộ gì
- Công ty dùng third-party provider nào
- Có cloud/CDN/mail/remote access/API service nào không

## Passive Recon

Ở giai đoạn này ưu tiên passive recon:

- Đọc website chính như một khách truy cập bình thường
- Xem SSL certificate
- Tra cứu Certificate Transparency logs bằng crt.sh
- Resolve subdomain thành IP
- Dùng Shodan để xem service đã được index
- Kiểm tra DNS records bằng dig

Mục tiêu là hạn chế tương tác trực tiếp với hạ tầng target để tránh gây chú ý.

## Tư duy chính

Quan sát cả:

- Những gì thấy được
- Những gì chưa thấy nhưng có thể suy luận

Ví dụ:

Công ty nói làm IoT  
=> Có thể có API, device management, MQTT broker, cloud dashboard

Công ty dùng Mailgun  
=> Có thể có email API, SMTP relay, webhook

Công ty dùng Office 365  
=> Có thể có OneDrive, SharePoint, Azure storage

## Nguồn tìm subdomain

### SSL Certificate

Certificate có thể chứa nhiều domain/subdomain trong SAN.

Ví dụ:

```text
inlanefreight.com
www.inlanefreight.com
support.inlanefreight.com
```
### crt.sh
crt.sh dùng Certificate Transparency logs để tìm certificate đã cấp cho domain.

```
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq .
```

Lọc subdomain unique:

```
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" \| jq . \| grep name \| cut -d":" -f2 \| grep -v "CN=" \| cut -d'"' -f2 \| awk '{gsub(/\\n/,"\n");}1;' \| sort -u
```
## Resolve subdomain thành IP

```
for i in $(cat subdomainlist); do  host $i | grep "has address" | cut -d" " -f1,4done
```

Mục tiêu:
- Xác định subdomain nào resolve được
- Lấy IP tương ứng
- Phân biệt host thuộc công ty và host thuộc third-party
## Shodan
Tạo danh sách IP:

```
for i in $(cat subdomainlist); do  host $i | grep "has address" | cut -d" " -f4 >> ip-addresses.txtdone
```

Kiểm tra từng IP trên Shodan:

```
for i in $(cat ip-addresses.txt); do  shodan host $idone
```

Shodan có thể cho biết:
- Open ports
- Service name
- Version
- Organization
- Location
- SSL/TLS info
- Banner
## DNS Records
```
dig any inlanefreight.com
```
Các record quan trọng:

| Record | Ý nghĩa                                                |
| ------ | ------------------------------------------------------ |
| A      | Domain/subdomain trỏ tới IPv4                          |
| AAAA   | Domain/subdomain trỏ tới IPv6                          |
| CNAME  | Alias sang domain khác                                 |
| MX     | Mail server                                            |
| NS     | Name server                                            |
| TXT    | Verification token, SPF, DKIM, DMARC, third-party info |
| SOA    | Thông tin DNS zone chính                               |

## Thông tin có thể suy ra từ TXT record
Ví dụ TXT record có thể tiết lộ:
- Google Gmail / Google Workspace
- Outlook / Microsoft 365
- Atlassian
- Mailgun
- LogMeIn
- Hosting provider
- IP được phép gửi mail
- Domain verification token
## Ghi nhớ
- Không scan third-party provider nếu không nằm trong scope.
- DNS record nhìn qua có vẻ nhàm chán nhưng thường chứa nhiều thông tin có giá trị.
- Passive recon tốt giúp định hướng active recon tốt hơn.
- Đừng chỉ thu thập subdomain, hãy suy luận công nghệ và cấu trúc hệ thống phía sau.


---
# Domain Information Lab Checklist

## 1. Hiểu target

- [ ] Xác định domain chính.
- [ ] Đọc website chính.
- [ ] Ghi lại công ty cung cấp dịch vụ gì.
- [ ] Suy luận các hệ thống kỹ thuật có thể tồn tại phía sau.

Câu hỏi:

- [ ] Công ty làm web/app/IoT/cloud/security/data/hosting không?
- [ ] Có customer portal không?
- [ ] Có support portal không?
- [ ] Có API/documentation không?
- [ ] Có login page không?

---

## 2. Kiểm tra SSL Certificate

- [ ] Mở website chính bằng HTTPS.
- [ ] Xem certificate.
- [ ] Kiểm tra SAN/Common Name.
- [ ] Ghi lại domain/subdomain xuất hiện trong certificate.

Câu hỏi:

- [ ] Certificate có nhiều subdomain không?
- [ ] Có `support`, `api`, `dev`, `staging`, `admin` không?
- [ ] Certificate do CA nào cấp?
- [ ] Certificate còn hạn không?

---

## 3. Tìm subdomain qua crt.sh

- [ ] Query domain trên crt.sh.
- [ ] Xuất kết quả JSON nếu cần.
- [ ] Lọc danh sách subdomain unique.
- [ ] Lưu vào file `subdomainlist`.

Command mẫu:

```bash
curl -s "https://crt.sh/?q=example.com&output=json" \
| jq . \
| grep name \
| cut -d":" -f2 \
| grep -v "CN=" \
| cut -d'"' -f2 \
| awk '{gsub(/\\n/,"\n");}1;' \
| sort -u > subdomainlist
```
## 4. Resolve subdomain thành IP

- [ ]  Resolve từng subdomain.
- [ ]  Lưu domain và IP.
- [ ]  Loại bỏ kết quả trùng.
- [ ]  Đánh dấu host có vẻ thuộc công ty.
- [ ]  Đánh dấu host thuộc third-party/CDN/cloud.

Command mẫu:

```
for i in $(cat subdomainlist); do  host $i | grep "has address" | cut -d" " -f1,4done
```

---

## 5. Tạo danh sách IP

- [ ]  Tạo file `ip-addresses.txt`.
- [ ]  Chỉ thêm IP nằm trong scope.
- [ ]  Loại bỏ IP trùng.
- [ ]  Không scan IP third-party nếu chưa được phép.

Command mẫu:

```
for i in $(cat subdomainlist); do  host $i | grep "has address" | cut -d" " -f4 >> ip-addresses.txtdonesort -u ip-addresses.txt -o ip-addresses.txt
```

---

## 6. Kiểm tra IP bằng Shodan

- [ ]  Chạy `shodan host` cho từng IP.
- [ ]  Ghi lại open ports.
- [ ]  Ghi lại service/version.
- [ ]  Ghi lại organization/location.
- [ ]  Ghi lại SSL/TLS information.
- [ ]  Ưu tiên IP có nhiều service đáng chú ý.

Command mẫu:

```
for i in $(cat ip-addresses.txt); do  shodan host $idone
```

---

## 7. Kiểm tra DNS Records

- [ ]  Chạy `dig any`.
- [ ]  Ghi lại A record.
- [ ]  Ghi lại MX record.
- [ ]  Ghi lại NS record.
- [ ]  Ghi lại TXT record.
- [ ]  Ghi lại SOA record.
- [ ]  Phân tích third-party provider từ TXT/MX/NS.

Command mẫu:

```
dig any example.com
```

---

## 8. Phân tích TXT Record

- [ ]  Tìm Google verification.
- [ ]  Tìm Microsoft verification.
- [ ]  Tìm Atlassian verification.
- [ ]  Tìm Mailgun/SPF include.
- [ ]  Tìm LogMeIn token.
- [ ]  Tìm IP trong SPF.
- [ ]  Ghi lại dịch vụ bên thứ ba công ty đang dùng.

Câu hỏi:

- [ ]  Công ty dùng Google Workspace hay Microsoft 365?
- [ ]  Có Mailgun/SendGrid/Amazon SES không?
- [ ]  Có Atlassian/Jira/Confluence không?
- [ ]  Có remote access platform không?
- [ ]  Có IP nào mới lộ ra từ SPF không?

---

## 9. Đưa ra hướng đi tiếp

- [ ]  Subdomain nào đáng kiểm tra tiếp?
- [ ]  IP nào nằm trong scope?
- [ ]  Service nào có bề mặt tấn công lớn?
- [ ]  Có cloud storage/document platform nào đáng kiểm tra không?
- [ ]  Có API/email/webhook/remote access portal nào không?