# DNS Enumeration

## Mục tiêu

DNS dùng để phân giải tên miền thành IP, nhưng trong pentest DNS còn là nguồn thông tin rất quan trọng để thu thập:

- Nameserver
    
- Mail server
    
- Subdomain
    
- Hostname nội bộ
    
- IP nội bộ
    
- TXT record
    
- Zone information
    
- Lỗi cấu hình zone transfer
    

## Các loại DNS server

| Loại                         | Ý nghĩa                                                   |
| ---------------------------- | --------------------------------------------------------- |
| DNS Root Server              | Server gốc quản lý TLD như `.com`, `.org`, `.net`.        |
| Authoritative Nameserver     | Server có quyền trả lời chính thức cho một zone/domain.   |
| Non-authoritative Nameserver | Server không sở hữu zone, chỉ query/cache từ server khác. |
| Caching DNS Server           | Cache kết quả DNS trong một thời gian.                    |
| Forwarding Server            | Chuyển tiếp DNS query đến DNS server khác.                |
| Resolver                     | Thành phần phân giải tên miền trên máy/router.            |

## DNS Records quan trọng

| Record | Ý nghĩa                                               |
| ------ | ----------------------------------------------------- |
| A      | Domain trỏ tới IPv4.                                  |
| AAAA   | Domain trỏ tới IPv6.                                  |
| MX     | Mail server của domain.                               |
| NS     | Nameserver của domain.                                |
| TXT    | Text record, thường chứa SPF, DMARC, xác minh domain. |
| CNAME  | Alias trỏ sang domain khác.                           |
| PTR    | Reverse lookup: IP sang domain.                       |
| SOA    | Thông tin quản trị zone.                              |

## BIND9 Configuration

Các file cấu hình thường gặp:

```bash
/etc/bind/named.conf.local
/etc/bind/named.conf.options
/etc/bind/named.conf.log
```

Ví dụ khai báo zone:

```bash
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

## Zone File

Zone file chứa toàn bộ record của một DNS zone.

Ví dụ:

```bash
/etc/bind/db.domain.com
```

Zone file thường có:

- 1 record SOA
    
- Ít nhất 1 record NS
    
- Các record A, MX, CNAME, TXT, PTR...
    

Nếu zone file bị lộ, attacker có thể biết nhiều hostname/subdomain quan trọng.

## Reverse Lookup

Reverse lookup dùng PTR record để đổi IP thành domain name.

Ví dụ:

```text
10.129.14.5 -> server1.domain.com
```

## Dangerous Settings

| Option          | Ý nghĩa                        |
| --------------- | ------------------------------ |
| allow-query     | Ai được phép query DNS server. |
| allow-recursion | Ai được phép recursive query.  |
| allow-transfer  | Ai được phép zone transfer.    |
| zone-statistics | Thu thập thống kê zone.        |

Cấu hình nguy hiểm nhất cần kiểm tra trong lab:

```text
allow-transfer
```

Nếu cấu hình sai, attacker có thể dùng AXFR để tải toàn bộ zone file.

## DNS Footprinting Commands

Query NS record:

```bash
dig ns inlanefreight.htb @10.129.14.128
```

Query version BIND:

```bash
dig CH TXT version.bind @10.129.120.85
```

Query tất cả record server chịu trả về:

```bash
dig any inlanefreight.htb @10.129.14.128
```

Thử zone transfer:

```bash
dig axfr inlanefreight.htb @10.129.14.128
```

Thử zone transfer zone nội bộ:

```bash
dig axfr internal.inlanefreight.htb @10.129.14.128
```

Brute-force subdomain bằng wordlist:

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do
    dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt
done
```

Dùng dnsenum:

```bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

## Tư duy cần nhớ

DNS enumeration không chỉ là tìm IP của domain. Trong pentest, DNS giúp map hạ tầng:

- Domain nào tồn tại?
    
- Nameserver nào chịu trách nhiệm?
    
- Có mail server nào?
    
- Có subdomain nội bộ bị lộ không?
    
- Có bản ghi TXT tiết lộ dịch vụ bên thứ ba không?
    
- Có cấu hình sai zone transfer không?
    

Lỗi nghiêm trọng nhất trong section này là AXFR mở sai quyền, vì nó có thể làm lộ toàn bộ zone file.

---

# DNS Enumeration Checklist

## 1. Xác định target

- [ ] Có IP DNS server chưa?
- [ ] Có domain/zone chưa? Ví dụ: `inlanefreight.htb`
- [ ] Kiểm tra port DNS:

```bash
sudo nmap -sU -sT -p53 <target-ip>
```
## 2. Query record cơ bản
- [ ]  Query A record:

```
dig A <domain> @<dns-server-ip>
```

- [ ]  Query AAAA record:

```
dig AAAA <domain> @<dns-server-ip>
```

- [ ]  Query NS record: (Lấy FQDN - tên miền đầy đủ của host trong DNS)

```
dig NS <domain> @<dns-server-ip>
```

- [ ]  Query MX record:

```
dig MX <domain> @<dns-server-ip>
```

- [ ]  Query TXT record:

```
dig TXT <domain> @<dns-server-ip>
```

- [ ]  Query SOA record:

```
dig SOA <domain> @<dns-server-ip>
```

## 3. Kiểm tra thông tin server

- [ ]  Query version BIND nếu có:

```
dig CH TXT version.bind @<dns-server-ip>
```

- [ ]  Chú ý version cũ có thể liên quan tới CVE.

## 4. Query ANY

- [ ]  Thử lấy các record server chịu trả về:

```
dig ANY <domain> @<dns-server-ip>
```

- [ ]  Ghi lại TXT, NS, MX, SOA, A record nếu có.

## 5. Kiểm tra Zone Transfer

- [ ]  Thử AXFR với domain chính:

```
dig AXFR <domain> @<dns-server-ip>
```

- [ ]  Nếu có subdomain/zone nội bộ, thử tiếp:

```
dig AXFR internal.<domain> @<dns-server-ip>
```

- [ ]  Nếu AXFR thành công, lưu toàn bộ output:

```
dig AXFR <domain> @<dns-server-ip> | tee axfr.txt
```

## 6. Phân tích kết quả AXFR

- [ ]  Tìm hostname quan trọng:

```
dc
vpn
mail
dev
staging
test
admin
intranet
wsus
backup
internal
```

- [ ]  Tách danh sách domain/subdomain.
- [ ]  Tách danh sách IP.
- [ ]  Đưa IP sang bước port scanning.

## 7. Brute-force subdomain

- [ ]  Dùng SecLists:

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do
    dig $sub.<domain> @<dns-server-ip> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt
done
```

- [ ]  Hoặc dùng `dnsenum`:

```bash
dnsenum --dnsserver <dns-server-ip> --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt <domain>
```

## 8. Chuẩn bị bước tiếp theo

- [ ]  Scan port các IP tìm được.
- [ ]  Thêm domain/subdomain vào `/etc/hosts` nếu cần.
- [ ]  Kiểm tra web service trên các subdomain.
- [ ]  Ghi chú record nào tìm được từ DNS.