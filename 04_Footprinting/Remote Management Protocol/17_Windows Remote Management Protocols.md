# Windows Remote Management Protocols

## Tổng quan

Windows có nhiều cơ chế quản trị từ xa. Trong pentest, các service này rất quan trọng vì nếu có credential hợp lệ hoặc cấu hình yếu, ta có thể đăng nhập GUI, mở PowerShell shell hoặc execute command từ xa.

Các protocol chính:

```text
RDP   -> Remote Desktop GUI
WinRM -> Remote command / PowerShell Remoting
WMI   -> Remote management / command execution
```

---

## RDP

RDP là giao thức điều khiển Windows desktop từ xa.

Port mặc định:

```text
3389/tcp
3389/udp
```

Dùng để:

- Đăng nhập GUI vào máy Windows
    
- Điều khiển desktop từ xa
    
- Quản trị server bằng giao diện đồ họa
    

Scan RDP:

```bash
nmap -sV -sC <target-ip> -p3389 --script rdp*
```

Thông tin cần chú ý:

```text
CredSSP (NLA): SUCCESS
Target_Name
NetBIOS_Domain_Name
NetBIOS_Computer_Name
DNS_Domain_Name
DNS_Computer_Name
Product_Version
System_Time
```

NLA bật nghĩa là server yêu cầu xác thực trước khi tạo RDP session đầy đủ.

Kết nối RDP từ Linux:

```bash
xfreerdp /u:<username> /p:"<password>" /v:<target-ip>
```

Nếu thấy cảnh báo certificate self-signed hoặc name mismatch, cần hiểu đây là cảnh báo xác thực server, không nhất thiết là lỗi đăng nhập.

---

## RDP Security Check

Cài dependency Perl:

```bash
sudo cpan
install Encoding::BER
```

Clone tool:

```bash
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git
cd rdp-sec-check
```

Chạy kiểm tra RDP security:

```bash
./rdp-sec-check.pl <target-ip>
```

Tool này kiểm tra server hỗ trợ protocol/security layer nào:

```text
PROTOCOL_RDP
PROTOCOL_SSL
PROTOCOL_HYBRID / CredSSP / NLA
ENCRYPTION_METHOD_NONE
ENCRYPTION_METHOD_40BIT
ENCRYPTION_METHOD_56BIT
ENCRYPTION_METHOD_128BIT
ENCRYPTION_METHOD_FIPS
```

---

## WinRM

WinRM là Windows Remote Management, dùng để quản trị Windows từ xa qua command line/PowerShell.

Port mặc định:

```text
5985/tcp -> HTTP
5986/tcp -> HTTPS
```

Scan WinRM:

```bash
nmap -sV -sC <target-ip> -p5985,5986 --disable-arp-ping -n
```

Nếu có credential hợp lệ, dùng evil-winrm:

```bash
evil-winrm -i <target-ip> -u <username> -p '<password>'
```

Sau khi vào được, ta có PowerShell shell:

```powershell
*Evil-WinRM* PS C:\Users\<user>\Documents>
```

---

## WMI

WMI là Windows Management Instrumentation, interface quản trị rất mạnh của Windows.

Port khởi tạo:

```text
135/tcp
```

Sau khi kết nối thành công, WMI có thể chuyển sang random high port.

Dùng Impacket wmiexec để execute command từ xa:

```bash
wmiexec.py <username>:'<password>'@<target-ip> "hostname"
```

Ví dụ:

```bash
/usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
```

Nếu thành công, output trả về hostname của target.

---

## Pentest mindset

Khi gặp Windows remote management service:

1. Scan port remote management.
    
2. Fingerprint service/version.
    
3. Với RDP, kiểm tra NLA, hostname, domain, product version.
    
4. Với WinRM, kiểm tra 5985/5986.
    
5. Với WMI, nhớ rằng port ban đầu là 135 nhưng traffic có thể chuyển sang random port.
    
6. Nếu có credential hợp lệ, thử login bằng RDP/WinRM/WMI.
    
7. Ghi lại hostname, domain, OS version, system time để phục vụ enum/pivot tiếp.
    
8. Không brute-force bừa bãi nếu không được phép vì dễ lock account hoặc bị EDR phát hiện.