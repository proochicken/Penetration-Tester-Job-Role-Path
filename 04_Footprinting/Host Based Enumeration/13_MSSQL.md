# MSSQL Enumeration - HTB Academy

## Tổng quan

MSSQL là Microsoft SQL Server, hệ quản trị cơ sở dữ liệu quan hệ của Microsoft. Nó thường xuất hiện trong môi trường Windows Server và các ứng dụng dùng .NET.

Port mặc định:

```text
1433/tcp
```

MSSQL thường gặp trên Windows target, đặc biệt trong môi trường domain/Active Directory.

## MSSQL Clients

Một số client dùng để kết nối MSSQL:

```text
SSMS
mssql-cli
SQL Server PowerShell
HeidiSQL
SQLPro
Impacket mssqlclient.py
```

Với pentester, `mssqlclient.py` của Impacket là công cụ rất quan trọng.

Tìm vị trí tool:

```bash
locate mssqlclient
```

Ví dụ output:

```text
/usr/bin/impacket-mssqlclient
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

## Default MSSQL Databases

|Database|Ý nghĩa|
|---|---|
|`master`|Chứa thông tin hệ thống của SQL Server instance|
|`model`|Template cho database mới|
|`msdb`|Dùng bởi SQL Server Agent để schedule jobs/alerts|
|`tempdb`|Lưu object tạm thời|
|`resource`|Read-only database chứa system objects|

## Authentication

MSSQL có thể dùng:

```text
Windows Authentication
SQL Authentication
```

Windows Authentication dùng tài khoản Windows/local/domain để xác thực.

SQL Authentication dùng tài khoản SQL riêng, ví dụ:

```text
sa
dbadmin
report_user
```

Tài khoản `sa` rất nhạy cảm. Nếu bật và dùng password yếu/default thì có thể bị khai thác.

## Dangerous Settings

Các điểm nguy hiểm cần chú ý:

- Client không dùng encryption khi kết nối MSSQL.
    
- Dùng self-signed certificate khi bật encryption.
    
- Named pipes được bật.
    
- Tài khoản `sa` dùng weak/default password.
    
- `xp_cmdshell` được bật.
    
- Account SQL/Windows có quyền quá cao.
    

## Nmap MSSQL Enumeration

```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <IP>
```

Thông tin cần note:

```text
hostname
instance name
MSSQL version
TCP port
named pipe
domain/computer name
xp_cmdshell status
empty password/default credential
database access
```

## Metasploit MSSQL Ping

```text
use auxiliary/scanner/mssql/mssql_ping
set rhosts <IP>
run
```

Module này giúp lấy thông tin MSSQL như:

```text
ServerName
InstanceName
Version
TCP port
Named pipe
Cluster status
```

## Connect bằng Impacket

Windows Authentication:

```bash
python3 mssqlclient.py Administrator@<IP> -windows-auth
```

Hoặc:

```bash
impacket-mssqlclient DOMAIN/user:password@<IP> -windows-auth
```

SQL Authentication:

```bash
impacket-mssqlclient user:password@<IP>
```

## Query cơ bản sau khi login

Liệt kê database:

```sql
select name from sys.databases;
```

Xem version:

```sql
select @@version;
```

Xem user hiện tại:

```sql
select system_user;
select user_name();
```

Kiểm tra quyền sysadmin:

```sql
select is_srvrolemember('sysadmin');
```

Chuyển database:

```sql
use <database>;
```

Liệt kê bảng:

```sql
select name from sys.tables;
```

Đọc dữ liệu:

```sql
select * from <table>;
```

## Quy trình tư duy

```text
1433 open
-> detect version
-> enumerate instance/domain/hostname
-> check named pipe
-> check weak/default sa
-> check Windows Authentication nếu có domain credential
-> login bằng mssqlclient.py
-> list databases
-> tìm database custom
-> enumerate tables/columns
-> kiểm tra quyền
-> tìm dữ liệu nhạy cảm
-> kiểm tra xp_cmdshell nếu có quyền
```

---
# MSSQL Lab Checklist

## 1. Xác định MSSQL service

- [ ] Scan port 1433:

```bash
sudo nmap -sV -p1433 <IP>
```
-  Nếu thấy `ms-sql-s`, ghi lại:
    -  IP
    -  Port
    -  Version
    -  Hostname nếu có
        
## 2. Chạy Nmap MSSQL scripts

```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <IP>
```

-  Ghi lại `Windows server name`.
    
-  Ghi lại `Instance name`.
    
-  Ghi lại MSSQL version.
    
-  Ghi lại `Named pipe`.
    
-  Kiểm tra `ms-sql-empty-password`.
    
-  Kiểm tra `ms-sql-xp-cmdshell`.
    
-  Kiểm tra `ms-sql-ntlm-info`.
    

## 3. Dùng Metasploit mssql_ping nếu cần

```
msfconsole
use auxiliary/scanner/mssql/mssql_ping
set rhosts <IP>
run
```

-  Ghi lại ServerName.
    
-  Ghi lại InstanceName.
    
-  Ghi lại Version.
    
-  Ghi lại TCP port.
    
-  Ghi lại Named Pipe.
    

## 4. Tìm client mssqlclient.py

```
locate mssqlclient
```

Nếu có `/usr/bin/impacket-mssqlclient`, dùng trực tiếp:

```
impacket-mssqlclient
```

Nếu chỉ có file Python example:

```
python3 /usr/share/doc/python3-impacket/examples/mssqlclient.py
```

## 5. Thử login nếu có credential

Windows Authentication:

```
impacket-mssqlclient DOMAIN/user:password@<IP> -windows-auth
```

Hoặc nhập password thủ công:

```
python3 mssqlclient.py Administrator@<IP> -windows-auth
```

SQL Authentication:

```
impacket-mssqlclient user:password@<IP>
```

## 6. Sau khi login

-  Xem database:
    

```
select name from sys.databases;
```

-  Xem version:
    

```
select @@version;
```

-  Xem user hiện tại:
    

```
select system_user;select user_name();
```

-  Kiểm tra quyền sysadmin:
    

```
select is_srvrolemember('sysadmin');
```

-  Chuyển sang database custom:
    

```
use <database>;
```

-  Liệt kê bảng:
    

```
select name from sys.tables;
```

-  Đọc dữ liệu bảng:
    

```
select * from <table>;
```

## 7. Kiểm tra hướng leo quyền nếu có quyền phù hợp

-  Kiểm tra `xp_cmdshell` có bật không.
    
-  Kiểm tra user có phải sysadmin không.
    
-  Kiểm tra database chứa credential/token/API key không.
    
-  Kiểm tra SQL Agent jobs nếu có quyền.
    
-  Ghi chú mọi misconfiguration để report.