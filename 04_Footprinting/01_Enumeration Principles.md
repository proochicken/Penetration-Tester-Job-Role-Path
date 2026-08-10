# Enumeration Principles

## Ý chính

Enumeration là quá trình thu thập thông tin về target bằng phương pháp chủ động và thụ động. Đây không phải là bước làm một lần rồi xong, mà là một vòng lặp: có thông tin → phân tích → tìm thêm thông tin → tiếp tục đào sâu.

OSINT nên được tách riêng khỏi enumeration vì OSINT chỉ dựa trên thu thập thông tin thụ động, không tương tác trực tiếp với target.

## Tư duy quan trọng

Mục tiêu của pentester không phải là cố khai thác hệ thống ngay lập tức, mà là tìm tất cả các con đường có thể dẫn tới khai thác.

Không nên thấy SSH/RDP/WinRM là brute-force ngay. Brute-force là phương pháp noisy, dễ bị log, dễ bị blacklist và có thể làm hỏng quá trình pentest.

## Câu hỏi cần tự hỏi khi enumeration

- Ta nhìn thấy gì?
- Vì sao ta nhìn thấy nó?
- Những gì ta thấy tạo ra bức tranh gì về target?
- Ta thu được gì từ thông tin đó?
- Ta có thể dùng nó như thế nào?
- Ta không nhìn thấy gì?
- Vì sao ta không nhìn thấy nó?
- Việc không nhìn thấy đó nói lên điều gì?

## Ba nguyên tắc chính

1. Có nhiều thứ hơn những gì ta nhìn thấy ban đầu. Hãy nhìn từ nhiều góc độ.
2. Phân biệt rõ cái ta thấy và cái ta chưa thấy.
3. Luôn có cách để thu thêm thông tin. Hãy hiểu target trước khi khai thác.

## Ghi nhớ

Enumeration tốt không chỉ phụ thuộc vào tool, mà phụ thuộc vào khả năng hiểu target, hiểu service, hiểu protocol và biết đặt câu hỏi đúng.

---
# Enumeration Principles Checklist

## 1. Xác định phạm vi

- [ ] Target thuộc scope nào?
- [ ] Có domain/IP/range nào được phép kiểm thử?
- [ ] Có giới hạn nào về tốc độ scan hoặc brute-force không?
- [ ] Có service nào không được phép tác động không?

## 2. Thu thập thông tin ban đầu

- [ ] Ghi lại domain/IP/service đã biết.
- [ ] Phân loại thông tin: domain, IP, port, service, user, technology, third-party.
- [ ] Tách rõ passive information và active information.
- [ ] Không vội khai thác khi chưa hiểu target.

## 3. Hiểu target

- [ ] Công ty/ứng dụng này phục vụ mục đích gì?
- [ ] Người dùng tương tác với hệ thống qua đâu?
- [ ] Admin quản trị qua đâu?
- [ ] Có third-party provider nào liên quan không?
- [ ] Có cơ chế bảo vệ nào có thể tồn tại không? Ví dụ: WAF, IDS, IPS, firewall, rate limit.

## 4. Phân tích cái nhìn thấy

- [ ] Ta thấy service nào?
- [ ] Vì sao service đó cần tồn tại?
- [ ] Service đó liên kết với thành phần nào khác?
- [ ] Có version/banner/config nào đáng chú ý không?
- [ ] Có endpoint, route, hostname, certificate, metadata nào lộ ra không?

## 5. Phân tích cái chưa thấy

- [ ] Có service nào đáng lẽ nên có nhưng lại không thấy không?
- [ ] Có khả năng firewall đang lọc port không?
- [ ] Có khả năng service chỉ mở nội bộ không?
- [ ] Có hostname/subdomain chưa resolve được không?
- [ ] Có thông tin nào bị che bởi CDN/WAF không?

## 6. Tránh hành vi noisy quá sớm

- [ ] Không brute-force ngay khi thấy SSH/RDP/WinRM.
- [ ] Không fuzz quá mạnh khi chưa hiểu rate limit.
- [ ] Không scan toàn bộ bằng tốc độ cao khi chưa cần thiết.
- [ ] Ưu tiên thu thập và phân tích trước khi tấn công.

## 7. Xây dựng attack path

- [ ] Từ mỗi thông tin tìm được, hỏi: “Thông tin này dùng để làm gì?”
- [ ] Tìm mối liên hệ giữa các service.
- [ ] Tìm đường từ external vào internal nếu có.
- [ ] Tìm credential leak, misconfiguration, exposed admin panel, default config.
- [ ] Ghi lại từng giả thuyết và bằng chứng.

## 8. Sau mỗi vòng enumeration

- [ ] Cập nhật sơ đồ target.
- [ ] Ghi lại phát hiện mới.
- [ ] Loại bỏ giả thuyết sai.
- [ ] Tạo giả thuyết mới.
- [ ] Tiếp tục đào sâu theo hướng có cơ sở.