# Linux Remote Management Protocols

## Ý chính

Các dịch vụ quản trị từ xa trên Linux như SSH, Rsync và R-Services thường xuất hiện trên server thật. Trong pentest, chúng là mục tiêu quan trọng vì có thể lộ version, cấu hình sai, credential reuse, file nhạy cảm hoặc quyền truy cập trực tiếp.

---

## SSH

SSH dùng để quản trị server từ xa qua kết nối mã hóa.

Port mặc định:

```text
22/tcp
```
SSH hỗ trợ:

- Remote shell
- Chạy command từ xa
- Copy file
- Port forwarding
- Tunneling

Có 2 phiên bản:

```
SSH-1: cũ, yếu, dễ bị MITM
SSH-2: mới hơn, an toàn hơn
```

Các kiểu auth phổ biến:

```
Password authentication
Public-key authentication
```

Public-key authentication dùng cặp key:

```
Private key: giữ bí mật trên client
Public key: đặt trên server
```

Private key nên được bảo vệ bằng passphrase.

---

## SSH Dangerous Settings

Các setting SSH nguy hiểm:

```
PasswordAuthentication yes  -> có thể brute-force password
PermitEmptyPasswords yes    -> cho phép password rỗng
PermitRootLogin yes         -> cho phép login trực tiếp bằng root
Protocol 1                  -> dùng SSH-1 lỗi thời
X11Forwarding yes           -> tăng attack surface
AllowTcpForwarding yes      -> có thể bị lạm dụng để pivot
PermitTunnel                -> cho phép tunneling
DebianBanner yes            -> lộ thông tin hệ thống
```

---
	
## SSH Enumeration

Kiểm tra cấu hình SSH server local:

```
cat /etc/ssh/sshd_config | grep -v "#" | sed -r '/^\s*$/d'
```

Fingerprint SSH bằng ==`ssh-audit`==:

```
git clone https://github.com/jtesta/ssh-audit.git
cd ssh-audit
./ssh-audit.py <target-ip>
```

Xem debug SSH connection:

```
ssh -v user@<target-ip>
```

Ép thử password authentication:

```
ssh -v user@<target-ip> -o PreferredAuthentications=password
```

---

## Rsync

Rsync dùng để sync/copy file local hoặc remote.

Port mặc định:

```
873/tcp
```

Rsync thường dùng cho:

- Backup
- Mirror folder
- Đồng bộ file giữa server

Rủi ro:

- Share không cần auth
- Lộ source code
- Lộ secrets/config
- Lộ thư mục `.ssh`
- Có thể download file nhạy cảm

Scan Rsync:

```
sudo nmap -sV -p 873 <target-ip>
```

Probe Rsync:

```
nc -nv <target-ip> 873
```

List share/module:

```
#list
```

List file trong module:

```
rsync -av --list-only rsync://<target-ip>/<module>
```

Download toàn bộ module:

```
rsync -av rsync://<target-ip>/<module> ./loot/
```

---

## R-Services

R-Services là nhóm dịch vụ Unix cũ để remote login/command/copy.

Port thường gặp:

```
512/tcp - rexec
513/tcp - rlogin
514/tcp - rsh/rcp
```

Các command:

```
rcp    -> remote copy
rexec  -> remote execution
rlogin -> remote login
rsh    -> remote shell
rwho   -> list user session
rusers -> list user session chi tiết hơn
```

Rủi ro chính:

- Không mã hóa
- Có thể bị sniff credential
- Trust sai qua `/etc/hosts.equiv` hoặc `.rhosts`
- Có thể login không cần password

File trust:

```
/etc/hosts.equiv  -> global trust
~/.rhosts         -> per-user trust
```

Wildcard nguy hiểm:

```
+ +
```

Có thể hiểu là tin tưởng mọi user từ mọi host.

Scan R-Services:

```
sudo nmap -sV -p 512,513,514 <target-ip>
```

Login bằng rlogin:

```
rlogin <target-ip> -l <username>
```

List user bằng rwho:

```
rwho
```

List user chi tiết bằng rusers:

```
rusers -al <target-ip>
```

---

## Pentest Mindset

Khi gặp remote management service:

1. Xác định service/version.
2. Kiểm tra banner/config/auth method.
3. Tìm misconfiguration.
4. Thử credential reuse nếu đã có credential hợp lệ trong scope.
5. Với Rsync, luôn kiểm tra share/module hở.
6. Với R-Services, kiểm tra trust relationship sai.
7. Nếu lấy được file như `.ssh`, config, secrets, backup thì phân tích để pivot tiếp.