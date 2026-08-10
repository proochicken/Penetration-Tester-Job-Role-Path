# SMTP Enumeration

## SMTP là gì?

SMTP là giao thức dùng để gửi email trong mạng IP.

Các port phổ biến:

```bash
25/tcp   # SMTP mặc định
587/tcp  # Mail submission, thường dùng STARTTLS
465/tcp  # SMTPS / SMTP over SSL
```

SMTP thường đi kèm với:

```text
SMTP  -> gửi email
IMAP  -> đọc/lấy email
POP3  -> tải email về client
```

## Luồng gửi email

```text
MUA -> MSA -> MTA -> MDA -> Mailbox
```

|Thành phần|Ý nghĩa|
|---|---|
|MUA|Mail User Agent, ví dụ Outlook/Thunderbird|
|MSA|Mail Submission Agent, nhận mail từ user đã xác thực|
|MTA|Mail Transfer Agent, chuyển tiếp mail giữa server|
|MDA|Mail Delivery Agent, đưa mail vào mailbox|

## SMTP Commands quan trọng

| Command    | Ý nghĩa                                     |
| ---------- | ------------------------------------------- |
| HELO       | Bắt đầu SMTP session                        |
| EHLO       | Bắt đầu ESMTP session và liệt kê capability |
| MAIL FROM  | Khai báo sender                             |
| RCPT TO    | Khai báo recipient                          |
| DATA       | Bắt đầu gửi nội dung email                  |
| RSET       | Hủy transaction hiện tại nhưng giữ kết nối  |
| VRFY       | Kiểm tra user/mailbox có tồn tại không      |
| EXPN       | Kiểm tra/mở rộng alias hoặc mailing list    |
| NOOP       | Giữ kết nối, tránh timeout                  |
| QUIT       | Kết thúc session                            |
| AUTH PLAIN | Xác thực client                             |

## Tương tác SMTP bằng telnet

```bash
telnet <target-ip> 25
```

Bắt đầu session:

```text
HELO mail1.inlanefreight.htb
EHLO mail1
```

`EHLO` thường trả về capability của SMTP server, ví dụ:

```text
PIPELINING
SIZE
VRFY
ETRN
8BITMIME
SMTPUTF8
CHUNKING
```

## Enumerate user bằng VRFY

```text
VRFY root
VRFY testuser
VRFY admin
```

Lưu ý: không phải lúc nào `VRFY` cũng đáng tin. Một số server trả về code `252` cho cả user thật và user giả.

## Gửi email thủ công qua SMTP

```text
EHLO inlanefreight.htb
MAIL FROM:<sender@inlanefreight.htb>
RCPT TO:<recipient@inlanefreight.htb>
DATA
From: <sender@inlanefreight.htb>
To: <recipient@inlanefreight.htb>
Subject: Test

Message body here.
.
QUIT
```

Dấu `.` một mình trên một dòng dùng để kết thúc phần DATA.

## Open Relay

Open Relay là lỗi cấu hình khi SMTP server cho phép bất kỳ IP nào gửi/chuyển tiếp email qua nó.

Cấu hình nguy hiểm:

```text
mynetworks = 0.0.0.0/0
```

Rủi ro:

- Gửi spam
    
- Phishing
    
- Mail spoofing
    
- Domain/IP bị blacklist
    

## Nmap SMTP Enumeration

Scan SMTP:

```bash
sudo nmap -sC -sV -p25 <target-ip>
```

Kiểm tra open relay:

```bash
sudo nmap <target-ip> -p25 --script smtp-open-relay -v
```

## Tư duy khi làm lab SMTP

1. Xác định port SMTP đang mở.
    
2. Dùng Nmap lấy service/version/capability.
    
3. Dùng `telnet` hoặc `nc` tương tác thủ công.
    
4. Kiểm tra `EHLO` capability.
    
5. Thử `VRFY` để enumerate user.
    
6. Kiểm tra open relay bằng Nmap NSE.
    
7. Không tin tuyệt đối output tool, phải hiểu server trả code gì.

---
# SMTP Lab Checklist

## 1. Port scanning

- [ ] Scan các port SMTP phổ biến:

```bash
sudo nmap -sC -sV -p25,465,587 <target-ip>
```
 
- Nếu chỉ biết port 25:    

```
sudo nmap -sC -sV -p25 <target-ip>
```

-  Ghi lại service/version, ví dụ `Postfix smtpd`.
## 2. Kiểm tra SMTP capability

-  Kết nối bằng telnet:
    

```
telnet <target-ip> 25
```

-  Gửi `HELO`:
    
```
HELO test.local
```

-  Gửi `EHLO`:
    
```
EHLO test.local
```

-  Ghi lại capability:
    
```
VRFY
STARTTLS
AUTH
SIZE
PIPELINING
ETRN
```

## 3. Enumerate user bằng VRFY

-  Thử user phổ biến:
    
```
VRFY root
VRFY admin
VRFY test
VRFY user
```

-  Nếu có wordlist user:
    
```bash
for user in $(cat users.txt); do
    echo "VRFY $user" | nc -nv <target-ip> 25
done
```

- Ta có thể sử dụng tool `smtp-user-enum` để enumerate user với syntax:
```bash
smtp-user-enum -M VRFY -U /opt/useful/seclists/Usernames/Names/names.txt -t <TARGET-IP> -p 25 -w 30
```
-  Cẩn thận với response code `252`, vì server có thể trả kết quả không đáng tin.
    
## 4. Kiểm tra gửi mail thủ công
-  Tương tác bằng telnet:
    
```
telnet <target-ip> 25
```

-  Thử SMTP flow:
    
```
EHLO inlanefreight.htb
MAIL FROM:<test@inlanefreight.htb>
RCPT TO:<user@inlanefreight.htb>
DATA
Subject: Test

Hello
.
QUIT
```

## 5. Kiểm tra Open Relay

-  Chạy Nmap NSE:
    
```
sudo nmap <target-ip> -p25 --script smtp-open-relay -v
```

-  Nếu thấy:
    
```
Server is an open relay
```

thì đây là finding quan trọng.

## 6. Kiểm tra cấu hình nếu có quyền đọc file
-  Với Postfix:
    
```
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
```
-  Kiểm tra các dòng quan trọng:
    
```
myhostname
mydestination
mynetworks
smtpd_banner
smtpd_helo_restrictions
```

-  Nếu thấy:
    
```
mynetworks = 0.0.0.0/0
```

thì đây là cấu hình open relay nguy hiểm.
## 7. Ghi chú report
-  SMTP version.
-  Supported commands/capabilities.
-  VRFY có enumerate user được không.
-  Có open relay không.
-  Có STARTTLS/AUTH không.
-  Có thông tin banner lộ hostname không.

