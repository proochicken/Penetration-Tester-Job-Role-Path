# DNS Zone Transfers

## Ý chính

DNS Zone Transfer là cơ chế sao chép toàn bộ DNS records trong một zone từ DNS server chính sang DNS server phụ.

Mục đích hợp pháp:

```text
Primary DNS Server -> Secondary DNS Server
```
để đảm bảo DNS có tính nhất quán và dự phòng.
## Vì sao nguy hiểm?

Zone transfer bản thân không phải lỗ hổng. Lỗ hổng xảy ra khi DNS server cấu hình sai và cho phép người không được phép thực hiện AXFR.

Nếu attacker/pentester lấy được zone file, họ có thể thấy:

- Danh sách subdomain
- IP address tương ứng
- Mail server
- Name server
- TXT/SRV/PTR records
- Dev/staging/admin/backup systems

Điều này làm lộ bản đồ hạ tầng DNS của target.

## Quy trình Zone Transfer
1. Secondary DNS gửi AXFR request đến Primary DNS.
2. Primary gửi SOA record.
3. Primary truyền toàn bộ DNS records.
4. Primary báo hoàn tất transfer.
5. Secondary gửi ACK xác nhận.

## Thuật ngữ quan trọng

- `Zone`: vùng DNS của một domain.
- `Zone file`: file chứa DNS records.
- `Primary DNS`: server DNS chính.
- `Secondary DNS`: server DNS phụ.
- `AXFR`: Full Zone Transfer.
- `SOA`: Start of Authority, record quản trị zone.
- `Serial number`: số phiên bản zone.
- `ACK`: xác nhận đã nhận dữ liệu.

## Command kiểm tra Zone Transfer

```
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

Ý nghĩa:

- `dig`: tool query DNS
- `axfr`: yêu cầu full zone transfer
	- `@nsztm1.digi.ninja`: DNS server cần hỏi
- `zonetransfer.me`: domain/zone cần transfer

## Dấu hiệu thành công

Nếu thành công, output sẽ trả về nhiều DNS records như:

- `SOA`
- `NS`
- `A`
- `MX`
- `TXT`
- `CNAME`
- `SRV`
- `PTR`

Ví dụ cuối output có thể thấy:

```
XFR size: 50 records
```

## Remediation

Chỉ cho phép zone transfer tới trusted secondary DNS servers.

Không cấu hình:

```
allow-transfer { any; };
```

Nên giới hạn theo IP cụ thể của secondary DNS.

## Ghi nhớ

Zone transfer là một bước recon quan trọng. Trong thực tế thường thất bại vì DNS server đã được cấu hình đúng, nhưng nếu thành công thì có thể lộ gần như toàn bộ attack surface DNS của target.

---
# DNS Zone Transfer Checklist

## 1. Xác định domain trong scope

- [ ] Xác định domain cần kiểm tra.
- [ ] Đảm bảo được phép kiểm thử.
- [ ] Không thử AXFR ngoài scope.

Ví dụ:

```text
inlanefreight.com
```
## 2. Tìm name servers

- [ ]  Query NS record.

```
dig NS example.com
```

Hoặc:

```
host -t NS example.com
```

- [ ]  Ghi lại tất cả name server.

Ví dụ:

```
ns1.example.com
ns2.example.com
```

## 3. Thử zone transfer trên từng name server

- [ ]  Thử AXFR với NS thứ nhất.

```
dig axfr @ns1.example.com example.com
```

- [ ]  Thử AXFR với NS thứ hai.

```
dig axfr @ns2.example.com example.com
```

- [ ]  Lặp lại với tất cả NS tìm được.

## 4. Xác định kết quả

### Nếu thất bại

Có thể thấy:

```
Transfer failed.
connection timed out
REFUSED
```

Ghi chú:

- [ ]  Server không cho AXFR.
- [ ]  Có thể firewall chặn.
- [ ]  Đây thường là cấu hình đúng.

### Nếu thành công

Có thể thấy nhiều record:

```
example.com.      IN SOA ...
example.com.      IN NS ...
www.example.com.  IN A ...
mail.example.com. IN MX ...
dev.example.com.  IN A ...
```

Ghi lại:

- [ ]  Toàn bộ output.
- [ ]  Danh sách subdomain.
- [ ]  IP tương ứng.
- [ ]  Mail server.
- [ ]  Name server.
- [ ]  TXT/SRV/PTR record đáng chú ý.

## 5. Phân tích dữ liệu thu được

- [ ]  Tìm subdomain nhạy cảm:

```
admin
dev
staging
test
vpn
portal
backup
internal
jira
git
jenkins
grafana
kibana
```

- [ ]  Map subdomain -> IP.
- [ ]  Kiểm tra HTTP/HTTPS service.
- [ ]  Kiểm tra công nghệ web.
- [ ]  Kiểm tra service không phải web nếu có.
- [ ]  Ưu tiên host có dấu hiệu dev/staging/legacy.

## 6. Validate

- [ ]  Resolve lại từng subdomain.

```
dig A dev.example.com
```

- [ ]  Kiểm tra web sống không.

```
curl -I http://dev.example.comcurl -I https://dev.example.com
```

- [ ]  Nếu nhiều subdomain trỏ cùng IP, kiểm tra virtual host.
- [ ]  Thêm vào `/etc/hosts` nếu lab yêu cầu hostname.

## 7. Ghi chú report

- [ ]  Ghi rõ DNS server nào cho phép AXFR.
- [ ]  Ghi rõ số lượng record bị lộ.
- [ ]  Ghi ví dụ subdomain/IP nhạy cảm.
- [ ]  Mô tả impact: lộ attack surface, hỗ trợ attacker mapping hạ tầng.
- [ ]  Đề xuất remediation: giới hạn AXFR cho secondary DNS tin cậy.