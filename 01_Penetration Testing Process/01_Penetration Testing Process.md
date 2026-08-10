# Process
## Ý chính
Pentest process là chuỗi các bước và sự kiện mà pentester thực hiện để đạt được mục tiêu đã định nghĩa trước. Quy trình này không phải là công thức cứng, mà là một framework linh hoạt. Mỗi bước phụ thuộc vào thông tin và kết quả thu được từ bước trước.
![01_Penetration Testing Process-20260524024704161.png](01_Penetration%20Testing%20Process/01_Penetration%20Testing%20Process-20260524024704161.png)
## Deterministic vs Stochastic
- **Deterministic process**: bước sau phụ thuộc vào bước trước. Trong pentest, hành động tiếp theo thường dựa trên kết quả vừa tìm được.
- **Stochastic process**: kết quả mang tính xác suất, không chắc chắn tuyệt đối. Ví dụ fuzzing/brute-force.
## Vì sao pentest không phải checklist cứng?
Mỗi target có hạ tầng, mục tiêu, scope và ràng buộc khác nhau. Vì vậy pentester cần có playbook riêng, nhưng phải biết điều chỉnh theo tình huống thực tế.
## 8 giai đoạn pentest
### 1. Pre-Engagement
- Thống nhất với khách hàng trước khi test.
- Gồm: NDA, goals, scope, time estimation, rules of engagement.
### 2. Information Gathering
- Thu thập thông tin về target.
- Ví dụ: domain, IP, service, technology, software/hardware, user, public information.
### 3. Vulnerability Assessment
- Phân tích thông tin đã thu thập.
- Tìm lỗ hổng đã biết hoặc hành vi đáng nghi.
- Có thể dùng tool tự động và kiểm tra thủ công.
### 4. Exploitation
- Kiểm tra và khai thác lỗ hổng để có initial access hoặc chứng minh impact.
### 5. Post-Exploitation
- Sau khi vào được hệ thống, tiếp tục enum từ bên trong.
- Có thể leo quyền, tìm credential, dữ liệu nhạy cảm, config quan trọng.
### 6. Lateral Movement
- Di chuyển từ máy đã compromise sang các host khác trong mạng nội bộ.
- Thường kết hợp với post-exploitation và pillaging.
### 7. Proof-of-Concept
- Ghi lại từng bước khai thác.
- Mục tiêu là chứng minh lỗ hổng tồn tại và cho khách hàng thấy chuỗi tấn công hoạt động như thế nào.
### 8. Post-Engagement
- Viết báo cáo, trình bày kết quả, cleanup, hỗ trợ khách hàng hiểu và sửa lỗi.
- Có thể có retest sau khi khách hàng vá lỗi.
## Takeaway
Pentest là một quy trình lặp và linh hoạt. Không nên học theo kiểu nhớ checklist máy móc. Cần hiểu mỗi giai đoạn dùng để làm gì, đầu vào là gì, đầu ra là gì, và khi nào cần quay lại giai đoạn trước.

---
# Pentest Process Checklist
## 1. Pre-Engagement
- [ ] Xác định target được phép test
- [ ] Xác định scope
- [ ] Xác định mục tiêu
- [ ] Xác định kỹ thuật nào được phép/không được phép
- [ ] Xác định thời gian test
## 2. Information Gathering
- [ ] Xác định domain/IP/subdomain
- [ ] Scan port/service
- [ ] Fingerprint technology
- [ ] Tìm version của service/app/plugin
- [ ] Thu thập endpoint, directory, parameter
- [ ] Tìm thông tin public: GitHub leak, docs, metadata

## 3. Vulnerability Assessment
- [ ] Map service/version với CVE hoặc known issue
- [ ] Kiểm tra misconfiguration
- [ ] Kiểm tra input đáng nghi
- [ ] Xác định attack vector khả thi
- [ ] Ưu tiên lỗi có impact cao

## 4. Exploitation
- [ ] Chuẩn bị payload/tool
- [ ] Test an toàn, tránh phá hệ thống
- [ ] Xác nhận lỗ hổng có thật
- [ ] Ghi lại request/response/screenshot
- [ ] Xác định có đạt được initial access không

## 5. Post-Exploitation
- [ ] Xác định user hiện tại
- [ ] Kiểm tra quyền hạn
- [ ] Tìm credential/config/token
- [ ] Tìm dữ liệu nhạy cảm
- [ ] Kiểm tra khả năng privilege escalation

## 6. Lateral Movement
- [ ] Kiểm tra network nội bộ
- [ ] Tìm host khác có thể truy cập
- [ ] Kiểm tra credential reuse
- [ ] Thử truy cập service nội bộ nếu trong scope

## 7. Proof-of-Concept
- [ ] Viết lại chuỗi khai thác từng bước
- [ ] Ghi rõ điều kiện cần có
- [ ] Ghi rõ impact
- [ ] Tạo script/payload tái hiện nếu cần

## 8. Post-Engagement
- [ ] Cleanup artifact nếu có
- [ ] Tổng hợp bằng chứng
- [ ] Viết report
- [ ] Đưa remediation rõ ràng
- [ ] Chuẩn bị retest nếu cần

---
# Flow Pentest 
Information Gathering
→ Tìm domain, subdomain, port, technology, endpoint

Vulnerability Assessment
→ Xác định điểm nghi ngờ: version cũ, form input, file upload, auth logic, exposed API

Exploitation
→ Thử khai thác có kiểm soát

Post-Exploitation
→ Nếu có access, enum từ bên trong

Proof-of-Concept
→ Ghi lại request, response, payload, impact

Post-Engagement
→ Viết report và remediation
