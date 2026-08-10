# DNS

## DNS là gì?

DNS - Domain Name System là hệ thống phân giải tên miền sang IP.

Ví dụ:

```text
www.example.com -> 192.0.2.1
```
DNS giúp con người dùng tên miền dễ nhớ thay vì phải nhớ địa chỉ IP.

## DNS hoạt động như thế nào?

Khi truy cập `www.example.com`:

1. Máy kiểm tra DNS cache local.
2. Nếu không có, máy hỏi DNS resolver.
3. Resolver hỏi Root Name Server.
4. Root chỉ đến TLD Name Server của `.com`.
5. TLD chỉ đến Authoritative Name Server của `example.com`.
6. Authoritative Name Server trả về IP của `www.example.com`.
7. Resolver trả IP cho máy client.
8. Client dùng IP để kết nối web server.

Luồng:

```
Client -> Resolver -> Root -> TLD -> Authoritative -> Resolver -> Client
```

## Hosts File

`hosts` file dùng để map hostname sang IP thủ công, bypass DNS lookup thông thường.

Vị trí:

```
Windows: C:\Windows\System32\drivers\etc\hosts
Linux/macOS: /etc/hosts
```

Format:

```
<IP Address>    <Hostname> [Alias]
```

Ví dụ:

```
127.0.0.1       localhost10.10.10.10     target.htb
```

Trong HTB, thường thêm hostname vào `/etc/hosts` để truy cập web đúng virtual host.

## DNS Zone

DNS zone là vùng quản lý DNS của một domain.

Ví dụ zone `example.com` có thể chứa:

```
www.example.com
mail.example.com
ftp.example.com
```
## Zone File

Zone file là file cấu hình chứa DNS records.

Ví dụ:

```
$TTL 3600
@       IN SOA   ns1.example.com. admin.example.com. (
                2024060401
                3600
                900
                604800
                86400 )

@       IN NS    ns1.example.com.
@       IN NS    ns2.example.com.
@       IN MX 10 mail.example.com.
www     IN A     192.0.2.1
mail    IN A     198.51.100.1
ftp     IN CNAME www.example.com.
```
## Các DNS Record quan trọng

|Record|Ý nghĩa|
|---|---|
|`A`|Hostname -> IPv4|
|`AAAA`|Hostname -> IPv6|
|`CNAME`|Alias hostname -> hostname khác|
|`MX`|Mail server của domain|
|`NS`|Name server của domain|
|`TXT`|Text record, thường dùng cho SPF/DKIM/DMARC/verification|
|`SOA`|Thông tin quản trị zone|
|`SRV`|Service + port + hostname|
|`PTR`|Reverse DNS: IP -> hostname|

## Vì sao DNS quan trọng trong Web Recon?

DNS giúp pentester:

- Tìm subdomain
- Tìm mail server
- Tìm name server
- Tìm IP thật của host
- Phát hiện hạ tầng cloud/third-party
- Tìm dev/staging/old server
- Phát hiện subdomain takeover qua CNAME
- Mapping network infrastructure
- Monitor asset mới như `vpn.example.com`
## Ghi nhớ
DNS không chỉ là phân giải domain. Trong pentest, DNS là nguồn OSINT/recon quan trọng để hiểu hạ tầng mục tiêu và tìm entry point.

- Vài tool được dùng trong phần này đó là:
	- `dig`
	- `nslookup`
	- `dnsenum`, `dnsrecon`
