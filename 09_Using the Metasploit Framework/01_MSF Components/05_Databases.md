# Metasploit Database

## Mục tiêu

Metasploit Database dùng PostgreSQL để lưu và quản lý:

- Hosts
    
- Services
    
- Vulnerabilities
    
- Credentials
    
- Loot
    
- Notes
    
- Scan results
    

Metasploit Database là database cục bộ của pentester, không phải database trên target.

## Khởi động PostgreSQL

Kiểm tra trạng thái:

```bash
sudo service postgresql status
```

Khởi động:

```bash
sudo systemctl start postgresql
```

Khởi tạo Metasploit database:

```bash
sudo msfdb init
```

Chạy Metasploit cùng database:

```bash
sudo msfdb run
```

Kiểm tra kết nối trong `msfconsole`:

```text
db_status
```

Kết quả mong muốn:

```text
Connected to msf. Connection type: postgresql.
```

## Database Commands

```text
help database
```

Các lệnh chính:

```text
db_status       Kiểm tra kết nối
db_import       Import kết quả scan
db_export       Export database
db_nmap         Chạy Nmap và tự lưu kết quả
hosts           Xem host
services        Xem service
vulns           Xem vulnerability
creds           Xem credential
loot            Xem dữ liệu thu thập
notes           Xem ghi chú
workspace       Quản lý workspace
```

## Workspace

Workspace dùng để tách dữ liệu giữa các assessment.

Xem workspace:

```text
workspace
```

Tạo:

```text
workspace -a Target_1
```

Chuyển:

```text
workspace Target_1
```

Xóa:

```text
workspace -d Target_1
```

Dấu `*` chỉ workspace đang active.

## Import Nmap

Scan và tạo XML:

```bash
nmap -sC -sV -oA Target 10.10.10.40
```

Import:

```text
db_import Target.xml
```

Kiểm tra:

```text
hosts
services
```

File XML được ưu tiên vì chứa dữ liệu có cấu trúc để Metasploit parse.

## Chạy Nmap trong Metasploit

```text
db_nmap -sV -sS 10.10.10.8
```

`db_nmap` chạy Nmap và tự động lưu kết quả vào database.

## Backup

Export workspace:

```text
db_export -f xml backup.xml
```

Import lại:

```text
db_import backup.xml
```

## Hosts

Xem host:

```text
hosts
```

Chỉ host đang up:

```text
hosts -u
```

Chọn cột:

```text
hosts -c address,name,os_name
```

Export CSV:

```text
hosts -o hosts.csv
```

Đặt kết quả thành RHOSTS:

```text
hosts -R
```

## Services

Xem service:

```text
services
```

Lọc theo port:

```text
services -p 445
```

Lọc theo service:

```text
services -s smb
```

Chỉ service đang mở:

```text
services -u
```

Đặt host phù hợp thành RHOSTS:

```text
services -p 445 -R
```

## Credentials

Xem credential:

```text
creds
```

Thêm username/password:

```text
creds add user:admin password:notpassword realm:workgroup
```

Thêm SSH key:

```text
creds add user:sshadmin ssh-key:/path/to/id_rsa
```

Lọc SMB credential:

```text
creds -s smb
```

Export:

```text
creds -o credentials.csv
creds -o hashes.jtr
creds -o hashes.hcat
```

## Loot

Xem loot:

```text
loot
```

Loot có thể gồm:

- Hash dump
    
- passwd/shadow
    
- Configuration file
    
- Screenshot
    
- Database dump
    
- File thu thập từ target
    

`creds` lưu thông tin xác thực đã chuẩn hóa, còn `loot` lưu file hoặc dữ liệu thu thập.

## Workflow

```text
1. Start PostgreSQL
2. Check db_status
3. Create/select workspace
4. Run db_nmap hoặc db_import XML
5. Review hosts
6. Review services
7. Filter targets
8. Set RHOSTS từ database
9. Run scanner/exploit
10. Review creds và loot
11. Export backup
```