# SMB/Samba Enumeration

## Mục tiêu

SMB là giao thức dùng để chia sẻ file, thư mục, máy in và tài nguyên mạng. Trong pentest, nếu thấy port `139/tcp` hoặc `445/tcp` mở thì cần thực hiện SMB enumeration.

Samba là triển khai SMB trên Linux/Unix, giúp Linux giao tiếp với Windows qua SMB/CIFS.

## Port quan trọng

```bash
139/tcp  # SMB qua NetBIOS
445/tcp  # SMB trực tiếp qua TCP
```

## Kiểm tra SMB bằng Nmap

```bash
sudo nmap -sV -sC -p139,445 <target>
```

- `-sV`: phát hiện version service
    
- `-sC`: chạy default NSE scripts
    
- `-p139,445`: chỉ scan port SMB
## Liệt kê share bằng smbclient

```bash
smbclient -N -L //<target>
```

- `-N`: null session, không dùng password
- `-L`: list share
    
Ví dụ output có thể thấy:

```text
print$
home
dev
notes
IPC$
```

## Kết nối vào share

```bash
smbclient //<target>/<share>
```

Ví dụ:

```bash
smbclient //10.129.14.128/notes
```

Một số lệnh trong smbclient:

```bash
help                 # xem lệnh hỗ trợ
ls                   # liệt kê file trong share
cd <dir>             # chuyển thư mục
get <file>           # tải file về local
mget *               # tải nhiều file
put <file>           # upload file nếu có quyền ghi
pwd                  # xem thư mục hiện tại trên SMB
!ls                  # chạy ls trên máy local
!cat <file>          # đọc file local
exit                 # thoát
```

## Cấu hình Samba nguy hiểm

```ini
browseable = yes
read only = no
writable = yes
guest ok = yes
create mask = 0777
directory mask = 0777
```

Ý nghĩa:
- `browseable = yes`: share có thể bị liệt kê
- `guest ok = yes`: cho phép anonymous/guest access
- `writable = yes`: cho phép ghi file
- `read only = no`: không giới hạn chỉ đọc
- `0777`: quyền quá rộng
    
Nếu share vừa cho guest access vừa writable, attacker có thể upload hoặc sửa file.
## Enumeration bằng rpcclient

Kết nối anonymous:

```bash
rpcclient -U "" <target>
```

Một số lệnh quan trọng:

```bash
srvinfo                 # thông tin server
enumdomains             # liệt kê domain/workgroup
querydominfo            # thông tin domain
netshareenumall         # liệt kê share
netsharegetinfo <share> # thông tin chi tiết share
enumdomusers            # liệt kê user
queryuser <RID>         # xem thông tin user theo RID
querygroup <RID>        # xem thông tin group
```

## Brute-force RID bằng rpcclient

```bash
for i in $(seq 500 1100);do rpcclient -N -U "" <target> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```
Mục tiêu: thử các RID để tìm user tồn tại.
## Công cụ bổ sung
```bash
samrdump.py <target>
smbmap -H <target>
crackmapexec smb <target> --shares -u '' -p ''
./enum4linux-ng.py <target> -A
```

## Tư duy chính
Không chỉ chạy tool tự động. Với SMB cần kiểm tra thủ công:
1. Port 139/445 có mở không?
2. Có null session không?
3. Liệt kê được share không?
4. Share nào đọc được?
5. Share nào ghi được?
6. Có file nhạy cảm không?
7. Có enum được user/group không?
8. Có thể dùng username thu được cho bước tiếp theo không?
    
SMB enumeration tốt thường mở ra nhiều hướng: credential discovery, user enumeration, password spraying, lateral movement hoặc privilege escalation.

---
# SMB/Samba Lab Checklist

## 1. Xác định SMB service

- [ ] Scan port SMB:

```bash
sudo nmap -sV -sC -p139,445 <target>
```
-  Ghi lại version SMB/Samba.
    
-  Kiểm tra SMB signing.
    
-  Kiểm tra SMBv1 có bật không.
    
-  Ghi chú hostname, NetBIOS name, domain/workgroup nếu có.
    
## 2. Liệt kê share anonymous
-  Thử null session:
    
```
smbclient -N -L //<target>
```
-  Nếu thành công, ghi lại danh sách share.
-  Nếu thất bại, thử username/password nếu lab cung cấp.
-  Chú ý các share lạ như `dev`, `backup`, `notes`, `users`, `home`, `public`, `shared`.
    
## 3. Kiểm tra quyền từng share
-  Kết nối vào từng share:
    
```
smbclient //<target>/<share>
```
- Hoặc cũng có thể sử dụng `smbmap` để kiểm tra quyền của tất cả các share
-  Thử anonymous trước.
-  Chạy:
    
```
ls
pwd
```
-  Nếu có file, tải về:
    
```
get <filename>
```
-  Nếu có nhiều file:
    
```
recurse ON
prompt OFF
mget *
```

-  Nếu nghi có quyền ghi, test upload file vô hại:
    
```
put test.txt
```
## 4. Tìm thông tin nhạy cảm trong file tải về
-  Tìm username.
-  Tìm password.
-  Tìm config file.
-  Tìm backup file.
-  Tìm SSH key.
-  Tìm database credential.
-  Tìm internal hostname/domain.
-  Tìm note/todo/dev hint.

Lệnh local hữu ích:
```
grep -Ri "password\|passwd\|user\|login\|key\|token\|secret" .
```
## 5. Enumeration bằng rpcclient
-  Kết nối RPC anonymous:
    
```
rpcclient -U "" -N <target>
```
-  Chạy:
    
```
srvinfo
enumdomains
querydominfo
netshareenumall
enumdomusers
```
-  Nếu có RID, query user:
    
```
queryuser <RID>
```
-  Query group nếu có:
    
```
querygroup <RID>
```
## 6. Brute-force RID nếu cần
-  Dùng vòng lặp RID:
    
```
for i in $(seq 500 1100);do rpcclient -N -U "" <target> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

-  Lưu username tìm được vào file:
    
```
users.txt
```
## 7. Dùng tool tự động để đối chiếu
-  Chạy `smbmap`:
    
```
smbmap -H <target>
```
-  Chạy `CrackMapExec/NetExec` nếu có:
    
```
crackmapexec smb <target> --shares -u '' -p ''
```

-  Chạy `enum4linux-ng`:
    
```
./enum4linux-ng.py <target> -A
```

-  Chạy `samrdump`:
    
```
samrdump.py <target>
```

## 8. Tổng hợp kết quả
-  Target có SMB không?
-  Version là gì?
-  Null session có thành công không?
-  Có bao nhiêu share?
-  Share nào READ được?
-  Share nào WRITE được?
-  Có file nhạy cảm không?
-  Có user/group nào bị leak không?
-  Có thể dùng thông tin này để brute-force/login service khác không?