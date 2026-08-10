# Host Discovery
## Mục tiêu
Host Discovery là bước xác định host nào đang online trong mạng trước khi scan port/service sâu hơn.Trong internal pentest, bước đầu tiên là có cái nhìn tổng quan về các hệ thống đang hoạt động trong network.
## Lưu ý quan trọng
Luôn lưu lại kết quả scan để phục vụ:
- So sánh kết quả
- Viết report
- Làm tài liệu
- Đối chiếu giữa nhiều tool
## Scan cả network range
```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet
```

- `10.129.2.0/24`: dải mạng cần scan
- `-sn`: không scan port, chỉ host discovery
- `-oA tnet`: lưu output ra nhiều định dạng với prefix `tnet`

## Scan từ danh sách IP

```
sudo nmap -sn -oA tnet -iL hosts.lst
```

- `-iL hosts.lst`: đọc target từ file danh sách IP

## Scan nhiều IP cụ thể

```
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20
```

Hoặc scan range ngắn:

```
sudo nmap -sn -oA tnet 10.129.2.18-20
```

## Scan một IP

```
sudo nmap 10.129.2.18 -sn -oA host
```

## Xem packet Nmap gửi/nhận

```
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

## Xem lý do Nmap kết luận host alive

```
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

## Ép Nmap dùng ICMP thay vì ARP

```
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

## Ghi nhớ

Không nhận được ping reply không có nghĩa là host đã chết. Firewall có thể chặn ICMP.

Trong cùng mạng LAN, Nmap thường dùng ARP request/reply để xác định host alive vì ARP đáng tin hơn ICMP trong local network.