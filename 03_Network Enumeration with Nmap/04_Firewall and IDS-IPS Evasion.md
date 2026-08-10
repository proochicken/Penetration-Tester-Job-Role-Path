# Firewall and IDS/IPS Evasion

## Mục tiêu

Section này nói về cách dùng Nmap để phân tích firewall rule và thử một số kỹ thuật scan né lọc trong môi trường được phép.

Ý quan trọng nhất:

> Khi Nmap báo `filtered`, đừng vội kết luận port đóng. Có thể packet đang bị firewall drop/reject hoặc IPS can thiệp.

---

## Firewall

Firewall là cơ chế lọc traffic dựa trên rule.

Firewall có thể:

- Cho packet đi qua
- Drop packet: im lặng bỏ packet
- Reject packet: chặn và trả phản hồi như TCP RST hoặc ICMP error

Các ICMP error có thể gặp:

- Net Unreachable
- Net Prohibited
- Host Unreachable
- Host Prohibited
- Port Unreachable
- Proto Unreachable

---

## IDS/IPS

IDS = Intrusion Detection System

- Giám sát traffic
- Phát hiện dấu hiệu tấn công
- Cảnh báo admin

IPS = Intrusion Prevention System

- Phát hiện dấu hiệu tấn công
- Tự động chặn hoặc can thiệp

IDS giống camera báo động.
IPS giống camera có thêm khả năng khóa cửa.

---

## SYN Scan

```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
```
- `-sS`: SYN scan
- Gửi TCP SYN
- Nếu nhận SYN-ACK -> port open
- Nếu nhận RST -> port closed
- Nếu không nhận phản hồi hoặc ICMP error -> filtered

Ví dụ:

```
21/tcp filtered ftp22/tcp open     ssh25/tcp filtered smtp
```

---

## ACK Scan

```
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

- `-sA`: ACK scan
- Dùng để kiểm tra firewall rule
- Không dùng để xác định open/closed theo cách thông thường

Kết quả cần nhớ:

```
RST trả về     -> unfilteredKhông phản hồi -> filtered
```

Ví dụ:

```
22/tcp unfiltered ssh
```

`unfiltered` nghĩa là packet đi qua firewall, không có nghĩa chắc chắn port open.

---

## Detect IDS/IPS

IDS/IPS khó phát hiện hơn firewall vì có thể hoạt động thụ động.

Cách suy luận:

- Scan từ một IP/VPS
- Nếu IP bị block sau khi scan mạnh, có thể có IPS hoặc admin đang phản ứng
- Tiếp tục test bằng IP khác nếu nằm trong scope cho phép
- Nếu bị block, cần scan chậm hơn, ít ồn hơn

---

## Decoy Scan

```
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

- `-D RND:5`: tạo 5 decoy IP ngẫu nhiên
- IP thật được trộn vào cùng các IP giả
- Mục đích: làm khó việc xác định source IP thật trong log

Lưu ý:

- Decoy IP nên là host còn sống
- Dùng bừa có thể gây kết quả không ổn định hoặc bị cơ chế bảo vệ phát hiện

---

## Scan bằng source IP khác

```
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
```

- `-S 10.129.2.200`: đặt source IP
- `-e tun0`: gửi packet qua interface `tun0`
- Dùng để kiểm tra firewall có rule theo subnet/source IP hay không

Nếu scan thường thấy:

```
445/tcp filtered
```

Nhưng scan với source IP khác thấy:

```
445/tcp open
```

Có thể firewall đang cho phép source IP/subnet cụ thể.

---

## DNS Proxying / Source Port 53

Scan bình thường:

```
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace
```

Kết quả:

```
50000/tcp filtered
```

Scan với source port 53:

```
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

Kết quả:

```
50000/tcp open
```

Ý nghĩa:

- Firewall có thể đang tin tưởng traffic có source port 53
- Đây là dấu hiệu rule firewall cấu hình yếu

---

## Kiểm tra bằng ncat

```
ncat -nv --source-port 53 10.129.2.28 50000
```

Nếu output:

```
Ncat: Connected to 10.129.2.28:50000.220 ProFTPd
```

Thì có thể kết nối thật tới service.  
Banner `220 ProFTPd` cho thấy service thực tế là ProFTPd.

---

## Tư duy cần nhớ

- `filtered` không đồng nghĩa với `closed`
- ACK scan giúp kiểm tra firewall rule
- SYN scan giúp kiểm tra trạng thái port
- Decoy scan dùng để làm nhiễu source IP trong log
- `--source-port 53` có thể bypass firewall cấu hình yếu
- Luôn xác minh service thật bằng banner grabbing hoặc `-sV`


---
# Firewall and IDS/IPS Evasion Checklist

## 1. Chuẩn bị

- [ ] Xác định target IP.
- [ ] Xác định scope được phép scan.
- [ ] Tạo thư mục lưu kết quả.
- [ ] Lưu output bằng `-oA` nếu cần report.
- [ ] Không scan quá aggressive nếu lab có IPS.

---

## 2. Scan SYN cơ bản

- [ ] Chạy SYN scan trên port cần kiểm tra:

```bash
sudo nmap <target> -p <ports> -sS -Pn -n --disable-arp-ping --packet-trace
```
-  Ghi lại port nào `open`.
-  Ghi lại port nào `filtered`.
-  Xem packet `RCVD` để hiểu target phản hồi gì.

---

## 3. Kiểm tra firewall bằng ACK scan
-  Chạy ACK scan:

```
sudo nmap <target> -p <ports> -sA -Pn -n --disable-arp-ping --packet-trace
```

-  Nếu thấy `unfiltered`, hiểu là packet đi qua firewall.
    
-  Nếu thấy `filtered`, hiểu là packet bị lọc hoặc không có phản hồi.
    
-  Không kết luận `unfiltered = open`.
    

---

## 4. So sánh SYN scan và ACK scan

-  Nếu SYN báo `filtered` nhưng ACK báo `unfiltered`, có thể firewall đang lọc SYN nhưng cho ACK đi qua.
    
-  Nếu cả SYN và ACK đều `filtered`, rule có thể chặn chặt hơn.
    
-  Nếu SYN báo `open`, xác minh tiếp bằng `-sV` hoặc banner grabbing.
    

---

## 5. Kiểm tra IDS/IPS
-  Scan chậm, tránh spam request.
-  Quan sát xem IP có bị block sau khi scan không.
-  Nếu bị block, ghi chú lại hành vi.
-  Giảm tốc độ scan hoặc đổi kỹ thuật scan nếu lab cho phép
-  Không dùng kỹ thuật này ngoài scope được phép.
    
---

## 6. Thử decoy scan
-  Chạy decoy scan:

```
sudo nmap <target> -p <port> -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```
-  Kiểm tra packet trace.
-  Ghi nhận port có phản hồi không.
-  Nhớ rằng decoy không đảm bảo ẩn danh tuyệt đối.
---
## 7. Thử source IP khác nếu phù hợp lab
-  Chạy scan thường:
```
sudo nmap <target> -n -Pn -p <port> -O
```

-  Chạy scan với source IP khác:
```
sudo nmap <target> -n -Pn -p <port> -O -S <source-ip> -e <interface>
```
-  So sánh kết quả.
-  Nếu port từ `filtered` thành `open`, có thể firewall rule phụ thuộc source IP/subnet.
---
## 8. Thử source port 53
-  Scan bình thường:
```
sudo nmap <target> -p <port> -sS -Pn -n --disable-arp-ping --packet-trace
```

-  Scan với source port 53:
```
sudo nmap <target> -p <port> -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

-  Nếu kết quả thay đổi từ `filtered` sang `open`, ghi chú khả năng firewall trust DNS source port.
---
## 9. Xác minh service bằng ncat
-  Kết nối thử bằng source port tương ứng:
```
ncat -nv --source-port 53 <target> <port>
```
-  Đọc banner trả về.
-  Ghi lại service thật nếu banner khác với Nmap guess.
    
---
## 10. Kết luận sau lab
-  Firewall drop hay reject packet?
-  ACK scan cho thấy port nào `unfiltered`?
-  Source port 53 có giúp bypass không?
-  Có dấu hiệu IDS/IPS block không?
-  Service thật là gì sau khi banner grabbing