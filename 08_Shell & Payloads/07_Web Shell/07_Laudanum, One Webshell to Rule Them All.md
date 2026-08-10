# Laudanum – Web Shell

## Laudanum là gì?

Laudanum là repository chứa web shell/payload cho nhiều công nghệ:

```text
ASP
ASPX
JSP
PHP
```

Trên Kali/Parrot thường nằm tại:

```text
/usr/share/laudanum
```

Laudanum không tự tìm hoặc khai thác lỗ hổng. Cần có một đường đưa file lên target và khiến web server thực thi file đó.

## Workflow

```text
Xác định web technology
→ chọn payload đúng extension
→ copy payload để chỉnh sửa
→ cấu hình allowed IP/callback
→ upload qua chức năng vulnerable
→ xác định URL file
→ trigger payload
→ chạy command
→ kiểm tra quyền hiện tại
```

## Chọn payload đúng web stack

|Stack|Extension|
|---|---|
|IIS + ASP.NET|`.aspx`|
|IIS legacy ASP|`.asp`|
|Java/Tomcat|`.jsp`|
|Apache/Nginx + PHP|`.php`|

## Copy ASPX shell

```bash
cp /usr/share/laudanum/aspx/shell.aspx ./demo.aspx
```

Không nên sửa trực tiếp file gốc trong `/usr/share/laudanum`.

## Cấu hình `allowedIps`

Sửa danh sách IP được phép truy cập web shell và thêm IP VPN của attacker.

Kiểm tra IP:

```bash
ip -br addr show tun0
```

`allowedIps` là access control của chính payload, không phải địa chỉ target.

## Virtual host

Thêm target vào `/etc/hosts`:

```text
<TARGET_IP> status.inlanefreight.local
```

Ví dụ:

```bash
sudo sh -c 'echo "<TARGET_IP> status.inlanefreight.local" >> /etc/hosts'
```

Kiểm tra:

```bash
getent hosts status.inlanefreight.local
```

## Upload

Upload `demo.aspx` bằng chức năng upload của ứng dụng.

Sau upload phải xác định:

- File được lưu ở đâu?
    
- Tên có bị đổi không?
    
- Thư mục có public không?
    
- Extension có được giữ không?
    
- Web server có thực thi ASPX không?
    

Ví dụ đường dẫn:

```text
\files\demo.aspx
```

URL thường dùng:

```text
http://status.inlanefreight.local/files/demo.aspx
```

## Command execution

Web shell gửi command qua HTTP và trả output qua response.

Các command kiểm tra ban đầu:

```cmd
whoami
hostname
systeminfo
ipconfig /all
```

Payload thường chạy dưới IIS application pool/service account, không mặc định là Administrator hoặc SYSTEM.

## Web shell và reverse shell

### Web shell

```text
Browser → HTTP command → Web shell → HTTP output
```

### Reverse shell

```text
Target → callback tới attacker listener
```

Web shell hữu ích khi outbound connection bị chặn, nhưng thường kém tương tác hơn reverse shell.

## Root cause của lỗ hổng

```text
Cho upload file thực thi
+ lưu trong web-accessible directory
+ giữ extension
+ cho phép server-side execution
= RCE
```

## Biện pháp phòng thủ

- Allowlist extension
    
- Kiểm tra nội dung và MIME
    
- Đổi tên file
    
- Lưu ngoài web root
    
- Tắt script execution trong upload directory
    
- Dùng account quyền thấp
    
- Theo dõi file mới và process tree IIS
    
- Chặn `w3wp.exe` sinh `cmd.exe`/`powershell.exe`
    

## Ghi nhớ

- Upload thành công chưa đồng nghĩa RCE.
    
- File phải nằm ở vị trí truy cập được và được runtime thực thi.
    
- Payload extension phải phù hợp web stack.
    
- `allowedIps` sai có thể khiến shell từ chối bạn.
    
- Web shell thường kế thừa quyền của web server.
    
- Xóa comment không làm payload vô hình.