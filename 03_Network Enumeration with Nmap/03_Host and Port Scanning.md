# Host and Port Scanning Checklist
## 1. Sau khi host alive
- [ ] Xác nhận IP target nằm trong scope.
- [ ] Tạo thư mục lưu kết quả scan.
- [ ] Đặt tên output rõ ràng theo target. 
## 2. Scan TCP cơ bản
- [ ] Scan top 1000 TCP ports:
```bash
sudo nmap <IP> -oA scans/basic
```

- [ ]  Scan top ports nếu muốn nhanh:

```bash
sudo nmap <IP> --top-ports=10 -oA scans/top10
```

- [ ]  Scan all TCP ports:

```bash
sudo nmap -p- <IP> -oA scans/all-tcp
```

## 3. Scan kỹ các port mở

- [ ]  Lấy danh sách port open.
- [ ]  Scan version/service:

```bash
sudo nmap -sV -sC -p <ports> <IP> -oA scans/service
```

- [ ]  Ghi lại service name, version, banner, hostname nếu có.

## 4. Debug trạng thái port

- [ ]  Nếu thấy `closed`, kiểm tra có RST không:

```bash
sudo nmap <IP> -p <port> --packet-trace -Pn -n --disable-arp-ping
```

- [ ]  Nếu thấy `filtered`, kiểm tra packet bị drop hay reject.
- [ ]  Dùng `--reason` để biết lý do Nmap kết luận:

```bash
sudo nmap <IP> -p <port> --reason
```

## 5. Kiểm tra firewall behavior

- [ ]  Nếu scan lâu và không có response, nghi firewall drop.
- [ ]  Nếu có ICMP error, nghi firewall reject.
- [ ]  Ghi chú port nào bị filtered để quay lại sau.

## 6. UDP Scan

- [ ]  Không bỏ qua UDP nếu lab/internal pentest yêu cầu.
- [ ]  Scan nhanh top UDP ports:

```bash
sudo nmap -sU -F <IP> -oA scans/udp-fast
```

- [ ]  Scan UDP port cụ thể nếu nghi ngờ:

```bash
sudo nmap -sU -p <port> <IP> --reason -oA scans/udp-port
```

## 7. Phân tích kết quả

- [ ]  Port open này tương ứng service gì?
- [ ]  Service version có CVE không?
- [ ]  Service có anonymous/default access không?
- [ ]  Có cần enumeration sâu hơn không?
- [ ]  Có thông tin OS/hostname/domain/workgroup không?

## 8. Không tin output một cách máy móc

- [ ]  `filtered` không có nghĩa là bỏ qua.
- [ ]  `open|filtered` với UDP cần kiểm tra thêm.
- [ ]  Service name của Nmap có thể đoán sai nếu không dùng `-sV`.
- [ ]  Version detection có thể không chính xác tuyệt đối.