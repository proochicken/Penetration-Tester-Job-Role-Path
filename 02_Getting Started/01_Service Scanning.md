
# Service Scanning
## Ý chính
Service scanning là bước đầu khi kiểm tra một target. Mục tiêu là xác định:
- Target đang chạy OS gì.
- Port nào đang mở.
- Service nào đang chạy trên các port đó.
- Phiên bản service là gì.
- Có misconfiguration hoặc thông tin nhạy cảm nào bị lộ không.
## Nmap cơ bản

```bash
nmap <target>
```
- Mặc định scan 1000 TCP port phổ biến nhất.
- Output gồm PORT, STATE, SERVICE.
- SERVICE chỉ là phỏng đoán theo port, chưa chắc đúng service thật.

### Nmap scan full port 

```
sudo nmap -p- --open -T4 -n -Pn --stats-every <IP>
```

## Nmap nâng cao


```
nmap -sV -sC -p- <target>
```

- `-sV`: dò version service.
- `-sC`: chạy default NSE scripts.
- `-p-`: scan toàn bộ 65535 TCP port.

## Banner Grabbing

```
nc -nv <target> <port>
nmap -sV --script=banner -p<port> <target>
```

Dùng để lấy banner, giúp nhận diện service/version.

## FTP Enumeration

```
nmap -sC -sV -p21 <target>
ftp -p <target>
```

Nếu thấy `Anonymous FTP login allowed`, thử đăng nhập:

```
username: anonymous
password: anonymous hoặc để trống
```

Các lệnh FTP hay dùng:

```
ls
cd <folder>
get <file>
exit
```

## SMB Enumeration

```
nmap --script smb-os-discovery.nse -p445 <target>
nmap -A -p445 <target>
smbclient -N -L \\\\<target>
smbclient -U <user> \\\\<target>\\<share>
```

Mục tiêu:

- Tìm share.
- Kiểm tra guest/null session.
- Dùng credential tìm được để truy cập share.
- Tải file nhạy cảm nếu có.

## SNMP Enumeration

```
snmpwalk -v 2c -c public <target> 1.3.6.1.2.1.1.5.0
onesixtyone -c dict.txt <target>
```

SNMP có thể lộ hostname, process, route, software version, credential trong process args.

## Điều cần nhớ

- Luôn scan kỹ port và version.
- Đừng chỉ nhìn port rồi kết luận service.
- FTP anonymous, SMB share, SNMP public/private là các điểm cần kiểm tra sớm.
- Credential tìm được ở một service có thể dùng lại ở service khác.

---

---

# Checklist làm lab

## A. Chuẩn bị

```bash
export IP=10.129.42.253
```
## B. Scan port cơ bản

```
nmap $IP
```

Ghi lại:
- Port mở.
- Service được đoán.
- Có port quen thuộc nào không: 21, 22, 80, 139, 445, 161.

## C. Scan full port + version + script

```
nmap -sV -sC -p- $IP -oN nmap_full.txt
```

Ghi lại:
- Service version.
- OS hint.
- Script output.
- Banner.
- Web title/header.
- Anonymous login nếu có.
- Share SMB nếu có.

## D. Kiểm tra banner từng service quan trọng

```
nc -nv $IP 21
nc -nv $IP 22
nc -nv $IP 80
```

Hoặc:

```
nmap -sV --script=banner -p21,22,80 $IP
```

## E. FTP checklist

```
nmap -sC -sV -p21 $IPftp -p $IP
```

Thử:

```
anonymous
```

Sau khi vào FTP:

```
lscd publsget <file>exit
```

Cần chú ý:

- Có anonymous login không?
- Có file `.txt`, `.conf`, `.bak`, `.zip`, `.kdbx`, `.ssh`, `id_rsa` không?
- Có credential không?
- Có quyền upload không?

## F. SMB checklist

Liệt kê share:

```
smbclient -N -L \\\\$IP
```

Thử truy cập share:

```
smbclient \\\\$IP\\users
```

Nếu có credential:

```
smbclient -U bob \\\\$IP\\users
```

Trong SMB shell:

```
ls
cd <folder>
get <file>
```

Cần chú ý:

- Share nào không mặc định?
- Guest có đọc được không?
- Credential lấy từ FTP có dùng được cho SMB không?
- Có file password/config/backup không?

## G. SNMP checklist

Scan UDP port 161 trước nếu cần:

```
sudo nmap -sU -p161 $IP
```

Thử community string phổ biến:

```
snmpwalk -v 2c -c public $IP
snmpwalk -v 2c -c private $IP
```

Lấy hostname:

```
snmpwalk -v 2c -c public $IP 1.3.6.1.2.1.1.5.0
```

Brute force community string:

```
onesixtyone -c dict.txt $IP
```

Cần chú ý:

- Hostname.
- Running processes.
- Installed software.
- Network interfaces.
- Route table.
- Credential lộ trong command line.

## H. Sau khi có credential

Thử reuse credential trên các service khác:

```
ssh user@$IPsmbclient -U user \\\\$IP\\share
ftp $IP
```

---
# Mindset 
- Đừng scan xong rồi bỏ qua output. Mỗi service hãy tự hỏi:
```
1. Service này là gì?
2. Version là bao nhiêu?
3. Có anonymous/default access không?
4. Có file/config/credential bị lộ không?
5. Credential tìm được có reuse được ở service khác không?
6. Có CVE/misconfiguration phổ biến nào liên quan không?
7. Có cần enum sâu hơn bằng tool chuyên dụng không?
```
