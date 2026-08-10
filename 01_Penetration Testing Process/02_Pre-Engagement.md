# Pre-Engagement
## Ý chính
Pre-Engagement là giai đoạn chuẩn bị trước khi penetration test bắt đầu. Mục tiêu là thống nhất rõ: khách hàng muốn test gì, phạm vi ở đâu, ai có thẩm quyền, luật chơi ra sao, rủi ro thế nào, và tài liệu pháp lý nào cần ký.

> Không có scope + permission rõ ràng thì pentest có thể trở thành hành vi trái phép.

## 3 thành phần chính
1. Scoping Questionnaire
2. Pre-Engagement Meeting
3. Kick-Off Meeting

## Tài liệu quan trọng
- NDA: thỏa thuận bảo mật.
- Scoping Questionnaire: bảng câu hỏi xác định nhu cầu và phạm vi.
- Scoping Document: tài liệu tổng hợp scope.
- SoW / Contract: hợp đồng hoặc phạm vi công việc.
- RoE: Rules of Engagement, luật chơi của pentest.
- Contractors Agreement: cần khi có physical assessment.
- Report: báo cáo trong/sau quá trình pentest.

## Scoping Questionnaire cần hỏi gì?
- Loại assessment: internal/external pentest, web app, wireless, social engineering, red team...
- Số lượng live hosts, IP/CIDR, domain/subdomain.
- Số lượng web/mobile app.
- Có authenticated testing không? Có bao nhiêu role?
- Phishing target bao nhiêu người?
- Có physical sites không?
- Red Team objective là gì?
- Có cần Active Directory assessment không?
- Test từ anonymous user hay domain user?
- Có cần bypass NAC không?

## Black/Grey/White Box
- Black box: gần như không có thông tin.
- Grey box: có một phần thông tin như IP, URL, account.
- White box: có nhiều thông tin như source code, kiến trúc, tài khoản.

## Evasiveness
- Non-evasive: không cố né detection.
- Hybrid-evasive: bắt đầu nhẹ, sau đó tăng dần độ ồn.
- Fully evasive: cố né detection như attacker thật.

## Pre-Engagement Meeting
Dùng để thống nhất:
- Goals
- Scope
- Pentest type
- Methodology
- Time estimation
- Third parties
- Evasive testing
- Risks
- Restrictions
- Information handling
- Contact information
- Communication channels
- Reporting
- Payment terms

## RoE - Rules of Engagement
RoE là tài liệu quy định:
- Được test cái gì
- Không được test cái gì
- Test khi nào
- Liên hệ ai khi có sự cố
- Khi nào phải dừng test
- Evidence lưu như thế nào
- Report như thế nào
- Permission to test đã ký chưa

## Kick-Off Meeting
Cuộc họp chính thức trước khi pentest bắt đầu. Xác nhận lại scope, thời gian, người liên hệ, emergency contact, rủi ro, cách báo cáo và điều kiện dừng test.

## Physical Assessment
Nếu có kiểm thử vật lý, cần Contractors Agreement. Đây là giấy tờ chứng minh pentester được phép thực hiện khi bị bảo vệ/nhân viên/cảnh sát hỏi.

## Takeaway
Pre-Engagement không phải phần kỹ thuật exploit, nhưng là nền tảng pháp lý và vận hành của toàn bộ pentest. Pentester chuyên nghiệp phải làm rõ scope, permission, risk, contact và RoE trước khi chạm vào hệ thống.

---
# Checklist - Pre-Engagement

## Hiểu khái niệm
- [ ] Giải thích được Pre-Engagement là gì.
- [ ] Nhớ được 3 phần chính: Scoping Questionnaire, Pre-Engagement Meeting, Kick-Off Meeting.
- [ ] Hiểu vì sao cần NDA trước khi trao đổi thông tin nhạy cảm.
- [ ] Hiểu vì sao phải xác minh người có thẩm quyền ký pentest.

## Tài liệu
- [ ] Phân biệt được NDA, Scoping Questionnaire, Scoping Document, SoW, RoE.
- [ ] Biết RoE là tài liệu “luật chơi” của pentest.
- [ ] Biết Contractors Agreement dùng cho physical assessment.
- [ ] Biết report được tạo trong và sau pentest.

## Scope
- [ ] Xác định được in-scope assets: IP, CIDR, domain, URL, app, account.
- [ ] Xác định được out-of-scope assets.
- [ ] Biết hỏi số lượng host, IP range, domain, subdomain, app, role.
- [ ] Biết hỏi có third-party/cloud/ISP/hosting provider không.
- [ ] Biết cần written permission nếu test tài sản liên quan bên thứ ba.

## Loại pentest
- [ ] Phân biệt Vulnerability Assessment và Penetration Test.
- [ ] Phân biệt Internal và External test.
- [ ] Phân biệt Web App, Wireless, Social Engineering, Physical, Red Team.
- [ ] Phân biệt Black Box, Grey Box, White Box.
- [ ] Phân biệt Non-evasive, Hybrid-evasive, Fully evasive.

## Rủi ro và vận hành
- [ ] Biết pentest có thể tạo log/alert.
- [ ] Biết brute force có thể gây account lockout.
- [ ] Biết DoS thường không được làm nếu không có cho phép rõ.
- [ ] Biết khi phát hiện critical vuln có thể phải dừng test và báo khách hàng.
- [ ] Biết cần emergency contact.
- [ ] Biết cần kênh liên lạc rõ ràng.

## Câu hỏi tự kiểm tra
- [ ] Nếu khách hàng chỉ đưa domain, đây là black/grey/white box?
- [ ] Nếu tìm thấy unauthenticated RCE trong external pentest thì nên làm gì?
- [ ] Nếu hệ thống không nằm trong RoE nhưng có liên kết tới target thì có được test không?
- [ ] Nếu test physical và bị bảo vệ bắt lại thì cần tài liệu nào?
- [ ] Nếu target dùng cloud provider, cần thêm điều kiện gì?

---
1. What assets are in scope?
2. What assets are explicitly out of scope?
3. What is the testing window?
4. Are third-party providers involved?
5. Is authenticated testing required?
6. How many user roles will be provided?
7. Are brute force attacks allowed?
8. Is DoS testing allowed?
9. Should testing be evasive or non-evasive?
10. Who should be contacted in case of critical findings?