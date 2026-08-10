# Protected File Transfer

## Mục tiêu

Khi pentest, ta có thể truy cập dữ liệu nhạy cảm như:

- User list
- Credentials
- NTDS.dit
- Enumeration result
- Network infrastructure info
- Active Directory information

Vì vậy khi cần transfer file, nên:

- Dùng secure channel: SSH, SFTP, HTTPS
- Hoặc mã hóa file trước khi transfer

## Lưu ý nghề nghiệp

Không nên exfiltrate dữ liệu thật như:

- PII
- Financial data
- Trade secrets
- Credit card numbers
- Dữ liệu khách hàng nhạy cảm

Trừ khi khách hàng yêu cầu rõ trong scope.

Nếu test DLP hoặc egress filtering, hãy tạo dummy data mô phỏng dữ liệu cần bảo vệ thay vì lấy dữ liệu thật.

## Windows File Encryption

Có thể dùng PowerShell script `Invoke-AESEncryption.ps1`.

Import module:

```powershell
Import-Module .\Invoke-AESEncryption.ps1
```
Encrypt string:

```
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text"
```

Decrypt string:

```
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "<Base64CipherText>"
```

Encrypt file:

```
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```

Output:

```
scan-results.txt.aes
```

Decrypt file:

```
Invoke-AESEncryption -Mode Decrypt -Key "p4ssw0rd" -Path .\scan-results.txt.aes
```

## Linux File Encryption với OpenSSL

Encrypt file:

```
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

Decrypt file:

```
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```

## Giải thích nhanh

- `enc`: dùng OpenSSL encryption mode
- `-aes256`: dùng AES-256
- `-iter 100000`: dùng 100,000 vòng lặp để derive key
- `-pbkdf2`: dùng PBKDF2 để derive key từ password
- `-in`: file input
- `-out`: file output
- `-d`: decrypt mode

## Ghi nhớ

- File nhạy cảm nên được mã hóa trước khi transfer.
- Ưu tiên SSH/SFTP/HTTPS nếu có thể.
- Nếu phải dùng kênh không mã hóa, hãy encrypt file trước.
- Dùng password mạnh và unique cho từng khách hàng.
- Không dùng lại một password cho nhiều engagement.
- Không exfiltrate dữ liệu thật nếu không cần thiết hoặc không nằm trong scope.