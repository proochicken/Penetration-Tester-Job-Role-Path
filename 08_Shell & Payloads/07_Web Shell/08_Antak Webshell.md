# Antak Webshell

## Tổng quan

Antak là ASP.NET web shell thuộc bộ công cụ Nishang. Nó chạy trên Windows web server sử dụng IIS/ASP.NET và cung cấp giao diện giống PowerShell thông qua trình duyệt.

Luồng hoạt động:

```text
Upload Antak.aspx
    ↓
Truy cập file qua HTTP
    ↓
IIS chuyển ASPX cho ASP.NET runtime
    ↓
Antak thực thi PowerShell command
    ↓
Output được trả về trình duyệt
```

## ASPX

- `.aspx` là extension thường dùng trong ASP.NET Web Forms.
    
- Code ASPX được thực thi phía server.
    
- Nếu upload được ASPX vào thư mục có quyền thực thi, file có thể được dùng làm web shell.
    
- Web shell chạy với quyền của IIS Application Pool, không mặc định là Administrator.
    

## Vị trí Antak

```bash
ls /usr/share/nishang/Antak-WebShell
```

Thông thường gồm:

```text
antak.aspx
Readme.md
```

## Tạo bản sao để chỉnh sửa

```bash
cp /usr/share/nishang/Antak-WebShell/antak.aspx \
   /home/administrator/Upload.aspx
```

Không nên sửa trực tiếp payload gốc trong `/usr/share`.

## Chỉnh sửa Antak

- Sửa username và password trong file ASPX.
    
- Có thể bỏ banner, ASCII art và comment để giảm static signature.
    
- Việc sửa chuỗi không bảo đảm bypass AV/EDR.
    
- Credential Antak chỉ bảo vệ giao diện web shell, không phải Windows credential.
    

## Cấu hình hostname

Thêm vào `/etc/hosts`:

```text
<TARGET_IP> status.inlanefreight.local
```

Ví dụ:

```text
10.129.201.20 status.inlanefreight.local
```

Điều này giúp request sử dụng đúng virtual host.

## Truy cập shell

Sau khi upload vào thư mục `files`:

```text
http://status.inlanefreight.local/files/Upload.aspx
```

Đăng nhập bằng credential đã cấu hình trong Antak.

## Enumeration ban đầu

```powershell
whoami
hostname
Get-Location
Get-ChildItem
Get-ChildItem C:\Users
ipconfig
$env:COMPUTERNAME
```

## Antak thực thi command theo process mới

Mỗi command có thể chạy trong process riêng, vì vậy trạng thái như `cd` có thể không tồn tại ở request tiếp theo.

Thay vì:

```powershell
cd C:\Users
```

rồi chạy riêng:

```powershell
dir
```

nên dùng:

```powershell
Get-ChildItem C:\Users
```

hoặc:

```powershell
Set-Location C:\Users; Get-ChildItem
```

## Phân biệt các loại đường dẫn

URL:

```text
/files/Upload.aspx
```

Đường dẫn vật lý có thể là:

```text
C:\inetpub\wwwroot\status.inlanefreight.local\files\Upload.aspx
```

Current working directory được lấy bằng:

```powershell
Get-Location
```

Nếu HTB hỏi “directory you land in”, phải nộp current working directory, không phải đường dẫn của file web shell.

## Chức năng Antak

- Chạy PowerShell command.
    
- Upload/download file.
    
- Encode và execute command.
    
- Chạy script trong memory.
    
- Parse `web.config`.
    
- Có thể hỗ trợ truy vấn SQL.
    
- Có thể dùng để triển khai reverse shell/callback.
    

## Dấu vết phòng thủ

Antak có thể bị phát hiện qua:

- IIS access logs.
    
- ASPX file upload.
    
- Process tree `w3wp.exe → powershell.exe`.
    
- PowerShell logging.
    
- AMSI.
    
- EDR telemetry.
    
- Outbound callback.
    
- Command line bất thường.