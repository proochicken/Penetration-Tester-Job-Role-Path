# Enumeration

## Ý chính

Enumeration là giai đoạn quan trọng nhất trong pentest. Mục tiêu không phải là chiếm quyền ngay, mà là tìm ra tất cả các hướng có thể tấn công target.

Tool chỉ hỗ trợ thu thập dữ liệu. Giá trị thật sự nằm ở việc hiểu dữ liệu đó, biết service hoạt động như thế nào và biết tương tác tiếp ra sao.

## Tư duy quan trọng

Không nên nghĩ: "Mình chưa chạy đủ tool."

Nên nghĩ:

- Service này dùng để làm gì?
- Nó đang expose thông tin gì?
- Có chức năng nào cho phép mình tương tác không?
- Có tài nguyên nào bị lộ không?
- Thông tin này có dẫn tới thông tin quan trọng hơn không?
- Có misconfiguration nào không?

## 2 loại thông tin cần tìm

1. Chức năng/tài nguyên cho phép tương tác với target hoặc lấy thêm thông tin.
2. Thông tin dẫn tới thông tin quan trọng hơn để truy cập target.

## Manual Enumeration

Manual enumeration là bắt buộc. Tool scan có thể bỏ sót do timeout, firewall, filter hoặc service phản hồi bất thường.

Kết quả từ tool không phải lúc nào cũng đúng tuyệt đối. Cần kiểm chứng thủ công.

## Ghi nhớ

Enumeration is the key.

Nhưng "key" không nằm ở việc chạy nhiều tool, mà nằm ở việc hiểu service và biết hỏi tiếp câu hỏi đúng.

---

# Enumeration Checklist

## 1. Xác định target

- [ ] Xác định IP/domain trong scope.
- [ ] Kiểm tra host có alive không.
- [ ] Nếu ping không trả lời, thử scan với `-Pn`.

## 2. Scan port

- [ ] Scan top ports.
- [ ] Scan full TCP ports.
- [ ] Scan service version.
- [ ] Scan default scripts nếu phù hợp.
- [ ] Nếu kết quả ít bất thường, thử chỉnh timeout/retry/rate.

## 3. Phân tích từng service

- [ ] Port này chạy service gì?
- [ ] Version là gì?
- [ ] Service này thường có chức năng gì?
- [ ] Có anonymous/default access không?
- [ ] Có banner/version leak không?
- [ ] Có file/config/user/path nào bị lộ không?

## 4. Enumeration thủ công

- [ ] Dùng `curl`, browser, Burp để kiểm tra web.
- [ ] Dùng `nc`/`telnet` để đọc banner nếu phù hợp.
- [ ] Dùng client native của service: `ftp`, `smbclient`, `mysql`, `ssh`, v.v.
- [ ] Kiểm tra robots.txt, source HTML, headers, cookies.
- [ ] Kiểm tra directory/file hidden nếu là web.

## 5. Tìm attack vector

- [ ] Có login form không?
- [ ] Có upload không?
- [ ] Có parameter nghi vấn không?
- [ ] Có user/credential leak không?
- [ ] Có service cũ/lỗi thời không?
- [ ] Có misconfiguration không?
- [ ] Có thông tin nào dẫn sang service khác không?

## 6. Không tin tool tuyệt đối

- [ ] Nếu port bị `filtered`, thử scan lại chậm hơn.
- [ ] Nếu kết quả quá ít, thử full port scan.
- [ ] Nếu service phản hồi lạ, tương tác thủ công.
- [ ] Nếu stuck, học thêm service đó thay vì chỉ đổi tool.