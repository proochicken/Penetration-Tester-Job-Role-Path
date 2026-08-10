# HTB Academy - Lateral Movement

## Ý chính
Lateral Movement là giai đoạn di chuyển ngang trong mạng nội bộ sau khi đã có initial access hoặc post-exploitation trên một host. Mục tiêu là kiểm tra attacker có thể đi sâu tới đâu, truy cập được hệ thống nào, tận dụng credential/hash ra sao, và hệ thống phòng thủ nội bộ có phát hiện/ngăn chặn được không.

> Mục tiêu không chỉ là chiếm một máy public-facing, mà là đánh giá impact thật sự nếu attacker vào được mạng nội bộ.

## Vì sao quan trọng?
Nếu một host trong mạng bị chiếm, attacker/ransomware có thể:
- Di chuyển sang máy khác
- Truy cập file server/database
- Lấy credential
- Leo quyền trong domain
- Làm gián đoạn toàn bộ hệ thống
- Đánh cắp hoặc mã hóa dữ liệu nhạy cảm

## Các phase trong Lateral Movement
1. Pivoting
2. Evasive Testing
3. Information Gathering
4. Vulnerability Assessment
5. Privilege Exploitation
6. Post-Exploitation

Lateral Movement là vòng lặp:
pivot → enum → phân tích → khai thác → post-exploitation → tiếp tục pivot.

## Pivoting
Pivoting là dùng host đã chiếm làm proxy/cầu nối để truy cập các mạng nội bộ không thể truy cập trực tiếp từ Internet.

Mô hình:
Attacker → Compromised Host → Internal Network

Mục tiêu:
- Truy cập subnet private
- Scan internal host
- Tương tác service nội bộ
- Đi sâu hơn vào mạng

## Evasive Testing
Trong lateral movement, nhiều hành động dễ bị phát hiện:
- Internal scan
- SMB/RDP/WinRM access
- Credential spraying
- Pass-the-hash
- Remote command execution
- Tool upload

Cần cân nhắc EDR, IDS/IPS, SIEM, segmentation và RoE trước khi test.

## Information Gathering nội bộ
Từ host đã chiếm, cần enum:
- IP/subnet
- Route
- DNS
- ARP
- Domain info
- Reachable hosts
- SMB/RDP/WinRM/SSH
- File shares
- Database servers
- Domain controllers
- Credential có thể reuse

## Vulnerability Assessment nội bộ
Bên trong mạng thường có nhiều lỗi:
- Share mở rộng
- Service nội bộ chưa vá
- Password reuse
- Credential trong script/config
- Group/permission sai
- Legacy protocol
- Internal app yếu
- Dev/test server chứa secret

Group và quyền truy cập rất quan trọng. Nếu chiếm được user thuộc nhóm developer/admin/helpdesk, có thể mở ra nhiều tài nguyên nội bộ.

## Privilege Exploitation
Các hướng lateral movement thường gặp:
- Crack hash/password
- Credential reuse
- Pass-the-hash
- Truy cập SMB share
- RDP/WinRM/SSH sang host khác
- Abuse service account
- Abuse group permission

Responder có thể được dùng trong lab/pentest để thu NTLMv2 hash. Nếu thu được hash của account quyền cao, có thể dùng pass-the-hash trong một số điều kiện.

## Post-Exploitation trên host mới
Khi truy cập được host mới, lặp lại:
- Xác định user/privilege
- Enum local system
- Pillaging
- Tìm credential
- Đánh giá privilege escalation
- Ghi evidence
- Xem có thể pivot tiếp không

## Takeaway
Lateral Movement là giai đoạn chứng minh mức độ lan truyền của attacker trong mạng nội bộ. Pentester giỏi không chỉ lấy shell, mà phải chỉ ra attacker có thể đi tới đâu, bằng đường nào, và tổ chức cần chặn ở đâu.

---
# Checklist - Lateral Movement

## 1. Điều kiện trước khi lateral movement
- [ ] Đã có initial access hoặc credential hợp lệ.
- [ ] Xác nhận lateral movement được RoE/scope cho phép.
- [ ] Ghi lại host ban đầu, user, privilege, thời gian truy cập.
- [ ] Xác định có cần evasive testing không.
- [ ] Không scan/tấn công ngoài scope.

## 2. Local network awareness
- [ ] Xem IP/interface của host hiện tại.
- [ ] Xem routing table.
- [ ] Xem DNS server.
- [ ] Xem ARP cache.
- [ ] Xác định subnet nội bộ.
- [ ] Xác định domain/workgroup.
- [ ] Xác định host đã từng giao tiếp với máy hiện tại.

## 3. Pivoting
- [ ] Xác định subnet không truy cập trực tiếp được từ attacker machine.
- [ ] Chọn kỹ thuật pivot phù hợp: SOCKS proxy, SSH tunnel, chisel, ligolo, sshuttle...
- [ ] Kiểm tra tunnel hoạt động.
- [ ] Route traffic qua pivot đúng cách.
- [ ] Scan nhẹ để xác nhận host reachable.
- [ ] Tránh scan ồn nếu engagement yêu cầu stealth.

## 4. Internal Information Gathering
- [ ] Discover host nội bộ.
- [ ] Xác định port/service quan trọng: SMB, WinRM, RDP, SSH, LDAP, MSSQL, HTTP.
- [ ] Xác định domain controller nếu có.
- [ ] Xác định file server/database/dev server.
- [ ] Tìm SMB shares.
- [ ] Tìm internal web apps.
- [ ] Tìm thông tin AD nếu là Windows domain.

## 5. Credential/Hash usage
- [ ] Kiểm tra credential thu được có thể dùng ở đâu.
- [ ] Kiểm tra password reuse nếu được phép.
- [ ] Kiểm tra hash có thể crack không.
- [ ] Kiểm tra pass-the-hash nếu lab/scope cho phép.
- [ ] Không credential spraying bừa nếu chưa được phép.
- [ ] Ghi rõ tài khoản nào truy cập được host nào.

## 6. Vulnerability Assessment nội bộ
- [ ] Kiểm tra service version nội bộ.
- [ ] Kiểm tra misconfiguration.
- [ ] Kiểm tra share permission.
- [ ] Kiểm tra group/role/permission.
- [ ] Kiểm tra credential trong file/script/config.
- [ ] Ưu tiên vector ít rủi ro trước.
- [ ] Tránh exploit có khả năng gây crash nếu chưa được phép.

## 7. Truy cập host mới
- [ ] Xác nhận host mới nằm trong scope.
- [ ] Truy cập bằng phương thức được phép.
- [ ] Ghi lại cách truy cập.
- [ ] Xác định user/privilege trên host mới.
- [ ] Lặp lại post-exploitation checklist.
- [ ] Tìm khả năng pivot tiếp.

## 8. Evidence và report
- [ ] Ghi sơ đồ đường đi attack path.
- [ ] Ghi credential/hash nào dùng cho bước nào.
- [ ] Ghi host nguồn → host đích.
- [ ] Ghi bằng chứng truy cập.
- [ ] Ghi impact.
- [ ] Ghi điểm kiểm soát bị thiếu: segmentation, least privilege, monitoring, patching...
- [ ] Ghi khuyến nghị remediation.