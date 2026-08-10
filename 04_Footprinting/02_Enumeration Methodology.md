# Enumeration Methodology

## Ý chính

Enumeration là quá trình thu thập thông tin có hệ thống về target. Trong pentest, nếu không có methodology rõ ràng thì rất dễ bỏ sót thông tin hoặc tốn thời gian vào hướng không đem lại kết quả.

HTB mô tả enumeration như việc đi qua nhiều lớp/bức tường. Mục tiêu không phải là phá bừa, mà là tìm khe hở hợp lý để tiến sâu hơn vào hệ thống.

## 3 mức enumeration

| Level                            | Ý nghĩa                                                                    |
| -------------------------------- | -------------------------------------------------------------------------- |
| Infrastructure-based enumeration | Thu thập thông tin về hạ tầng: domain, subdomain, ASN, IP, cloud, firewall |
| Host-based enumeration           | Thu thập thông tin trên host: port, service, version, process              |
| OS-based enumeration             | Thu thập thông tin sâu bên trong OS: user, privilege, config, patch level  |
|                                  |                                                                            |
![02_Enumeration Methodology-20260528013007947.png](02_Enumeration%20Methodology/02_Enumeration%20Methodology-20260528013007947.png)
## 6 Layer Enumeration

| Layer | Mục tiêu | Thông tin cần tìm |
|---|---|---|
| 1. Internet Presence | Xác định các target có thể kiểm thử | Domain, subdomain, vHost, ASN, netblock, IP, cloud instance |
| 2. Gateway | Hiểu target được bảo vệ như thế nào | Firewall, DMZ, IDS/IPS, EDR, proxy, NAC, VPN, Cloudflare |
| 3. Accessible Services | Xác định service có thể truy cập | Service type, port, version, interface, functionality |
| 4. Processes | Hiểu process xử lý phía sau service | PID, task, source, destination, processed data |
| 5. Privileges | Xác định quyền hạn của user/service | User, group, permission, restriction, environment |
| 6. OS Setup | Hiểu cấu hình OS bên trong | OS type, patch level, network config, config file, sensitive file |

## Tư duy quan trọng

- Không phải lỗ hổng nào cũng dẫn vào sâu hơn.
- Pentest luôn bị giới hạn thời gian, nên phải biết ưu tiên.
- Methodology không phải là danh sách command cố định.
- Tool và command chỉ là cheat sheet hỗ trợ.
- Điều quan trọng là biết mình đang ở layer nào, cần tìm thông tin gì, và thông tin đó giúp đi tiếp như thế nào.

## Câu hỏi tự hỏi khi enumeration

- Mình đang kiểm tra layer nào?
- Mình đã biết gì về target?
- Còn thiếu thông tin gì?
- Thông tin này có giúp mở rộng attack surface không?
- Có lớp bảo vệ nào như firewall, WAF, Cloudflare, IDS/IPS không?
- Service này dùng để làm gì?
- Service chạy với quyền nào?
- Có file cấu hình, credential, hoặc dữ liệu nhạy cảm nào không?

```
Layer 1: Target này có những domain/subdomain nào?
Layer 2: Có WAF/CDN/firewall không?
Layer 3: Web service chạy công nghệ gì, version gì, endpoint nào?
Layer 4: Request được xử lý bởi app/process nào?
Layer 5: App chạy với quyền gì, có user/session/role nào?
Layer 6: Nếu có foothold, OS là gì, config và file nhạy cảm ở đâu?
```
