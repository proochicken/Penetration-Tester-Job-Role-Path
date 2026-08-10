# NFS Enumeration

## Ports
```
111/tcp -> rpcbind
2049/tcp -> nfs
```
## Discover
```
nmap -sV -sC -p111,2049 <IP>

nmap --script nfs* -p111,2049 <IP>

showmount -e <IP>
```
## Mount
```
mkdir target-NFS

sudo mount -t nfs <IP>:/ target-NFS -o nolock
```
## Enumerate
```
tree target-NFS

ls -la

ls -n
```

## Dangerous Options
```
rw
insecure
nohide
no_root_squash
```
## Important
```
root_squash:
root client -> anonymous user

no_root_squash:
root client = root server
```
=> Possible PrivEsc via SUID binary
## Interesting Files
```
id_rsa
authorized_keys
backup.sh
config files
credentials
```
---
# Checklist làm lab

## Bước 1: Scan port 
```bash
sudo nmap -sV -sC -p111,2049 <IP>
```
- Port quan trọng:
```
111 -> rpcbind
2049 -> nfs
```
- Nếu thấy:
```bash
111/tcp open rpcbind
2049/tcp open nfs
```
=> Có khả năng NFS đang chạy
## Bước 2: Liệt kê NFS bằng NSE
```bash
sudo nmap --script nfs* -p111,2049 <IP>
```
- Script sẽ hiển thị:
	- Share name
	- File name
	- Quyền truy cập
	- Dung lượng
- Ví dụ:
```
id_rsa
id_rsa.pub
```
- Nếu gặp private key như trên thì ngon luôn
## Bước 3: Xem các share được export
```bash
showmount -e <IP>
```
- Ví dụ:
```
/mnt/nfs 10.129.14.0/24
```
## Bước 4: Mount share
- Tạo thư mục:
```bash
mkdir target-NFS
```
- Mount:
```bash
sudo mount -t nfs 10.129.14.128:/ ./target-NFS -o nolock
```
## Bước 5: Duyệt dữ liệu
```bash
tree .
ls -la
```
## Bước 6: Kiểm tra UID/GUID
```bash
ls -n
```
