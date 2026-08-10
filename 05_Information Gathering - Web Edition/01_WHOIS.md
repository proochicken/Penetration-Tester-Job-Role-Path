# WHOIS

## Mục tiêu

WHOIS là giao thức dùng để tra cứu thông tin đăng ký của tài nguyên Internet như domain name, IP address block và Autonomous System.

Có thể hiểu WHOIS như một danh bạ Internet: giúp biết domain/IP thuộc về ai, đăng ký ở đâu, dùng name server nào, được tạo khi nào và hết hạn khi nào.

## Thông tin thường có trong WHOIS

- `Domain Name`: tên miền
- `Registrar`: nhà đăng ký domain
- `Registrant Contact`: người/tổ chức đăng ký domain
- `Administrative Contact`: người quản trị domain
- `Technical Contact`: người phụ trách kỹ thuật
- `Creation Date`: ngày tạo domain
- `Expiration Date`: ngày hết hạn domain
- `Updated Date`: lần cập nhật gần nhất
- `Name Servers`: DNS server chịu trách nhiệm phân giải domain

## Command cơ bản

```bash
whois inlanefreight.com
```
Ý nghĩa:

- Gửi truy vấn WHOIS cho domain `inlanefreight.com`
- Trả về thông tin đăng ký domain
- Dùng trong giai đoạn recon/footprinting

## Vì sao WHOIS quan trọng trong Web Recon?

WHOIS giúp pentester:

- Xác định tổ chức sở hữu domain
- Biết registrar/domain provider
- Tìm name server để tiếp tục DNS enumeration
- Tìm email/contact kỹ thuật nếu chưa bị ẩn
- Phân tích ngày tạo/ngày cập nhật/ngày hết hạn domain
- Pivot sang historical WHOIS để xem thông tin cũ
- Tìm manh mối về hạ tầng mạng và nhà cung cấp dịch vụ

## Thuật ngữ quan trọng

- `Registrar`: nơi domain được đăng ký, ví dụ Namecheap, GoDaddy, Amazon Registrar
- `Registry`: đơn vị quản lý TLD như `.com`, `.net`, `.org`
- `Registrant`: người/tổ chức sở hữu domain
- `Name Server`: DNS server của domain
- `RIR`: tổ chức quản lý IP/ASN theo khu vực
- `ASN`: mã số đại diện cho một hệ thống mạng lớn trên Internet
- `ICANN`: tổ chức quản lý chính sách domain/DNS toàn cầu
- `GDPR`: luật bảo vệ dữ liệu cá nhân, khiến nhiều thông tin WHOIS bị ẩn
- `RDAP`: giao thức hiện đại hơn WHOIS, có cấu trúc tốt hơn và hỗ trợ privacy tốt hơn

## Ghi nhớ

WHOIS là bước recon thụ động quan trọng. Nó không khai thác lỗ hổng trực tiếp, nhưng giúp thu thập manh mối để tiếp tục enumeration.

Không nên chỉ nhìn mỗi domain. Hãy chú ý thêm:

- Registrar
- Name server
- Contact email
- Creation/Updated/Expiration date
- Organization name
- Historical WHOIS
- IP block / ASN nếu có

---
# Utilizing WHOIS

## Ý chính

WHOIS không chỉ dùng để xem domain thuộc về ai, mà còn rất hữu ích trong các tình huống security thực tế như:

- Điều tra phishing
- Phân tích malware/C2
- Viết threat intelligence report
- Tìm pattern hạ tầng của threat actor

## Scenario 1: Phishing Investigation

Khi điều tra email phishing, tra WHOIS domain trong link độc hại có thể giúp phát hiện red flags:

- Domain mới được đăng ký vài ngày trước
- Registrant bị ẩn sau privacy service
- Name server/hosting liên quan đến bulletproof hosting

Kết hợp các dấu hiệu này có thể cho thấy domain đang được dùng cho phishing campaign.

Hành động:

- Block domain
- Cảnh báo người dùng
- Điều tra thêm IP/hosting provider
- Tìm domain liên quan

## Scenario 2: Malware Analysis

Malware thường kết nối về C2 server để nhận lệnh hoặc exfiltrate dữ liệu.

Tra WHOIS domain C2 có thể giúp biết:

- Registrant dùng email ẩn danh
- Location đáng ngờ
- Registrar có chính sách abuse yếu
- Hosting provider liên quan

Từ đó có thể notify hosting provider hoặc bổ sung IOC.

## Scenario 3: Threat Intelligence

Khi phân tích nhiều domain của cùng threat actor, WHOIS có thể giúp tìm pattern:

- Domain được đăng ký theo cụm
- Đăng ký ngay trước đợt tấn công
- Dùng alias/fake identity
- Dùng chung name server
- Có lịch sử takedown

Những pattern này giúp tạo threat profile, TTPs và IOC.

## Cài đặt WHOIS

```bash
sudo apt update
sudo apt install whois -y
```
## Sử dụng WHOIS

```
whois facebook.com
```

## Thông tin cần chú ý trong output

- `Domain Name`: tên miền
- `Registrar`: nhà đăng ký domain
- `Creation Date`: ngày tạo domain
- `Registry Expiry Date`: ngày hết hạn
- `Registrant Organization`: tổ chức sở hữu domain
- `Domain Status`: trạng thái bảo vệ domain
- `Name Server`: DNS server
- `DNSSEC`: trạng thái DNSSEC

## Ví dụ facebook.com

WHOIS cho thấy:
- Domain tạo từ năm 1997
- Đăng ký qua RegistrarSafe, LLC
- Chủ sở hữu là Meta Platforms, Inc.
- Có nhiều domain status dạng `Prohibited` để chống xóa/chuyển/cập nhật trái phép
- Name server thuộc `facebook.com`, cho thấy Meta tự quản lý DNS infrastructure
## Ghi nhớ

WHOIS thường không chỉ ra lỗ hổng trực tiếp. Nó là nguồn OSINT/recon giúp hiểu digital footprint của mục tiêu.

WHOIS nên được kết hợp với:

- DNS enumeration
- Subdomain enumeration
- Certificate Transparency
- ASN/IP lookup
- Historical WHOIS
- Threat intelligence feeds