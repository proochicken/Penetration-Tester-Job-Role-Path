# HTB Academy - Information Gathering

## Ý chính
Information Gathering là giai đoạn thu thập thông tin về target sau khi Pre-Engagement đã hoàn tất và có đầy đủ permission/scope. Đây là nền móng của pentest vì mọi bước khai thác sau đó đều dựa trên thông tin ta enumerate được.

> Pentest tốt không bắt đầu bằng exploit, mà bắt đầu bằng việc hiểu target.

## 4 nhóm chính
1. OSINT
2. Infrastructure Enumeration
3. Service Enumeration
4. Host Enumeration

## OSINT
OSINT là thu thập thông tin công khai từ Internet:
- Website
- GitHub/GitLab
- StackOverflow
- LinkedIn
- Job postings
- Social media
- Public documents
- Search engine

Có thể tìm thấy:
- Password
- Hash
- SSH key
- API token
- Internal URL
- Email nhân viên
- Code/config bị public

Nếu phát hiện password/key/token public thì cần báo theo quy trình Incident Handling trong RoE.

## Infrastructure Enumeration
Mục tiêu là lập bản đồ hạ tầng:
- Domain/subdomain
- IP range
- DNS server
- Mail server
- Web server
- Cloud instance
- VPN
- Firewall/WAF

Luôn so sánh asset tìm được với scope trước khi test sâu.

## Service Enumeration
Mục tiêu là hiểu service trên từng port:
- Port nào mở?
- Service gì?
- Version bao nhiêu?
- Banner trả về gì?
- Có anonymous/default access không?
- Có CVE hoặc misconfiguration không?

Version rất quan trọng vì service cũ thường có lỗ hổng đã biết.

## Host Enumeration
Mục tiêu là hiểu từng host:
- OS là gì?
- Host đóng vai trò gì?
- Service nào đang chạy?
- Host giao tiếp với máy nào?
- Có service nội bộ nào không?
- Có file/config/credential nào nhạy cảm không?

Internal enumeration thường phát hiện nhiều misconfiguration vì admin hay nghĩ “nội bộ là an toàn”.

## Pillaging
Pillaging là thu thập thông tin nhạy cảm sau khi đã có quyền truy cập vào host:
- Config file
- Credential
- SSH key
- API token
- Backup file
- Internal document
- Customer/employee data
- Script
- Local services

Thông tin này dùng để chứng minh impact, leo quyền hoặc lateral movement.

## Takeaway
Information Gathering không chỉ làm một lần ở đầu pentest. Nó lặp lại liên tục trong toàn bộ quá trình: trước exploit, trong exploit, sau exploit và khi leo quyền/di chuyển ngang.

---
# Checklist - Information Gathering

## 1. Xác nhận scope
- [ ] Đọc lại scope/RoE trước khi enumerate.
- [ ] Xác định domain/IP/URL nào được phép test.
- [ ] Ghi riêng các asset phát hiện ngoài scope, không test sâu nếu chưa được phép.

## 2. OSINT
- [ ] Tìm domain chính của target.
- [ ] Tìm subdomain public.
- [ ] Tìm email nhân viên.
- [ ] Kiểm tra GitHub/GitLab public.
- [ ] Tìm token/key/password/config bị lộ.
- [ ] Tìm thông tin công nghệ qua job postings.
- [ ] Tìm tài liệu public có metadata.
- [ ] Ghi lại nguồn tìm thấy thông tin.

## 3. Infrastructure Enumeration
- [ ] Thu thập DNS records: A, AAAA, MX, NS, TXT, CNAME.
- [ ] Map hostname sang IP.
- [ ] Xác định mail server.
- [ ] Xác định name server.
- [ ] Xác định cloud/CDN/WAF nếu có.
- [ ] Tạo danh sách asset.
- [ ] So sánh asset với scope.

## 4. Service Enumeration
- [ ] Scan port trên host trong scope.
- [ ] Xác định service trên từng port.
- [ ] Xác định version service.
- [ ] Lấy banner nếu có.
- [ ] Kiểm tra anonymous/default access.
- [ ] Tìm CVE/misconfiguration liên quan version.
- [ ] Ghi chú service đáng quan tâm.

## 5. Host Enumeration
- [ ] Xác định hệ điều hành.
- [ ] Xác định role của host.
- [ ] Xác định service nội bộ/public.
- [ ] Xác định host giao tiếp với thành phần nào.
- [ ] Nếu đã có shell, kiểm tra file/config/script/local service.
- [ ] Tìm credential/token/key nhưng xử lý theo đúng RoE.

## 6. Pillaging sau khai thác
- [ ] Tìm file cấu hình.
- [ ] Tìm credential trong file.
- [ ] Tìm SSH key/API key/token.
- [ ] Kiểm tra lịch sử command.
- [ ] Kiểm tra backup file.
- [ ] Kiểm tra user/home directory.
- [ ] Kiểm tra network share/internal docs.
- [ ] Ghi bằng chứng chứng minh impact.
- [ ] Không lấy dữ liệu quá mức cần thiết.

## 7. Ghi chú kết quả
- [ ] Lưu asset inventory.
- [ ] Lưu command đã chạy.
- [ ] Lưu output quan trọng.
- [ ] Lưu screenshot/bằng chứng.
- [ ] Phân loại phát hiện: OSINT / Infrastructure / Service / Host / Pillaging.

---
1. Tôi đang cần biết điều gì về target?
2. Thông tin đó lấy từ nguồn public, network scan, service hay local host?
3. Thông tin này có nằm trong scope không?
4. Thông tin này giúp mở ra hướng tấn công nào?
5. Có cần báo ngay cho khách hàng nếu phát hiện secret public không?
6. Có bằng chứng đủ để chứng minh impact chưa?
