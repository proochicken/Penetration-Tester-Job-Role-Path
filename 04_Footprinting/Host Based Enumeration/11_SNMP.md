# SNMP Enumeration

## Mục tiêu

SNMP là giao thức dùng để giám sát và quản lý thiết bị mạng như router, switch, server, IoT device.

Trong pentest, nếu thấy `UDP/161` mở thì cần kiểm tra SNMP vì dịch vụ này có thể leak nhiều thông tin nội bộ.

## Port quan trọng

```text
UDP/161 -> SNMP query
UDP/162 -> SNMP trap
```
## Thành phần chính

```text
SNMP Agent   -> service chạy trên target, ví dụ snmpd
SNMP Client  -> máy query SNMP, ví dụ Kali
MIB          -> bản đồ mô tả object SNMP
OID          -> địa chỉ cụ thể của object trong cây SNMP
Community    -> chuỗi giống password dùng trong SNMPv1/v2c
```

## SNMP Versions

```
SNMPv1  -> cũ, không mã hóa, không authentication mạnh
SNMPv2c -> phổ biến, dùng community string, vẫn plaintext
SNMPv3  -> an toàn hơn, có authentication và encryption
```
## Community String

Community string giống như password đơn giản.

Ví dụ phổ biến:

```
public
private
manager
admin
```

Nếu đoán được community string, có thể dùng `snmpwalk` để dump thông tin hệ thống.

## Dangerous Settings

```
rwuser noauth
rwcommunity <community string> <IPv4>
rwcommunity6 <community string> <IPv6>
```

Các cấu hình này nguy hiểm vì có thể cấp quyền ghi hoặc truy cập rộng vào OID tree.

## Tools

```
snmpwalk    -> query OID và dump thông tin SNMP
onesixtyone -> brute-force community string
braa        -> brute-force/enumerate OID nhanh
```

## Commands chính

Kiểm tra SNMP:

```
sudo nmap -sU -p161,162 --open <IP>
```

Query bằng community string `public`:

```
snmpwalk -v2c -c public <IP>
```

Brute-force community string:

```
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <IP>
```

Enumerate OID bằng braa:

```
braa public@<IP>:.1.3.6.*
```

## Tư duy pentest

SNMP không phải RCE trực tiếp, nhưng có thể leak thông tin cực kỳ giá trị:

- OS/kernel version
- hostname
- contact email
- installed packages
- running services
- process list
- network interfaces
- usernames
- internal naming convention

Thông tin này dùng để chọn exploit, tìm credential, hiểu môi trường nội bộ và mở rộng attack path.

---
# SNMP Lab Checklist  
  
## 1. Kiểm tra port SNMP  
  
- [ ] Scan UDP port 161/162:  
  
```bash  
sudo nmap -sU -p161,162 --open <IP>
```
-  Nếu muốn lấy version/script cơ bản:
    
```
sudo nmap -sU -sV -p161,162 <IP>
```

-  Có thể thử NSE script:
    
```
sudo nmap -sU -p161 --script=snmp-info <IP>
```

## 2. Thử community string phổ biến
-  Thử `public`:
    
```
snmpwalk -v2c -c public <IP>
```
-  Thử query system info:
    
```
snmpwalk -v2c -c public <IP> 1.3.6.1.2.1.1
```
-  Nếu có output, lưu lại kết quả:
    
```
snmpwalk -v2c -c public <IP> | tee snmpwalk-public.txt
```
## 3. Brute-force community string nếu chưa biết
-  Dùng onesixtyone:
    
```
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <IP>
```

-  Nếu tìm được community string, ghi lại:
    
```
<IP> [community] system description
```
## 4. Enumerate sâu bằng community string tìm được

-  Dump toàn bộ SNMP tree:
    
```bash
snmpwalk -v2c -c <community> <IP>
```

-  Enumerate system info:
    
```
snmpwalk -v2c -c <community> <IP> 1.3.6.1.2.1.1
```

-  Enumerate host resources:
    
```
snmpwalk -v2c -c <community> <IP> 1.3.6.1.2.1.25
```
## 5. Tìm thông tin quan trọng trong output
-  Hostname
    
-  OS/kernel version
    
-  Uptime
    
-  Contact email
    
-  Location
    
-  Installed packages
    
-  Running processes
    
-  Network interfaces
    
-  Usernames hoặc service names
    
-  Internal domain/hostname pattern
    
## 6. Dùng braa để enumerate OID nhanh

```
braa <community>@<IP>:.1.3.6.*
```
Ví dụ:
```
braa public@10.129.14.128:.1.3.6.*
```
## 7. Đánh giá rủi ro
-  Có dùng SNMPv1/v2c không?
    
-  Community string có phải mặc định không?
    
-  Có query được nhiều thông tin nhạy cảm không?
    
-  Có read-write community không?
    
-  SNMP có bị expose quá rộng không?
    
-  Có cần khuyến nghị chuyển sang SNMPv3 không?