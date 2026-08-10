# Post-Engagement

## Ý chính
Post-Engagement là giai đoạn sau khi pentest kỹ thuật đã hoàn thành. Mục tiêu không còn là khai thác thêm, mà là kết thúc engagement một cách chuyên nghiệp: cleanup, thu thập bằng chứng, viết report, họp review, hỗ trợ remediation, retest, xử lý dữ liệu và đóng dự án.

## Cleanup
Sau khi test xong, pentester cần dọn dẹp các artifact đã tạo ra:
- Tool/script/file đã upload
- Account đã tạo
- Cấu hình đã thay đổi
- Payload hoặc file tạm

Nếu không thể cleanup, cần báo khách hàng và ghi rõ trong appendices. Ngay cả khi cleanup thành công, vẫn nên document để khách hàng biết các alert đó là hoạt động pentest hợp lệ.

## Documentation & Reporting
Trước khi kết thúc test, cần đảm bảo đã thu thập đủ bằng chứng:
- Command output
- Screenshot
- Affected hosts
- Scan/log output
- Proof-of-concept
- Thông tin môi trường liên quan

Không nên giữ PII hoặc dữ liệu nhạy cảm không cần thiết.

Report nên có:
- Attack chain nếu có compromise sâu
- Executive summary cho non-technical audience
- Finding chi tiết: risk rating, impact, remediation, references
- Step to reproduce
- Khuyến nghị ngắn hạn/trung hạn/dài hạn
- Appendices: scope, OSINT, ports/services, compromised hosts/accounts, files transferred, account/system modifications, AD analysis, scan data

## Report Review Meeting
Sau khi gửi draft report, hai bên họp để review:
- Đi qua finding quan trọng
- Giải thích rủi ro
- Trả lời câu hỏi
- Làm rõ hoặc sửa nội dung nếu cần

Không cần đọc report word-by-word.

## Deliverable Acceptance
Quy trình thường là:
1. Gửi report bản DRAFT
2. Khách hàng review/comment
3. Pentester chỉnh sửa nếu cần
4. Gửi bản FINAL

Một số audit firm không chấp nhận report còn trạng thái DRAFT.

## Post-Remediation Testing
Sau khi khách hàng vá lỗi, pentester test lại để xác nhận:
- Finding đã được remediate chưa
- Payload cũ còn hoạt động không
- Scan còn phát hiện lỗi không
- Cần evidence chứng minh lỗi đã hết hoặc vẫn còn

Status thường dùng:
- Remediated
- Not Remediated
- Partially Remediated
- Risk Accepted

## Role of Pentester in Remediation
Pentester nên giữ vai trò độc lập:
- Không trực tiếp sửa code
- Không trực tiếp patch hệ thống
- Không trực tiếp đổi cấu hình AD
- Không tự remediate finding của mình

Pentester có thể tư vấn ở mức tổng quát, giải thích finding và đưa best practice. Điều này giúp tránh conflict of interest.

## Data Retention
Sau pentest có nhiều dữ liệu nhạy cảm như scan result, log, credential, screenshot. Cần:
- Tuân theo SOW/RoE/chính sách công ty
- Lưu trữ an toàn, encrypted at rest
- Chỉ giữ trong thời gian cần thiết
- Wipe dữ liệu khỏi máy tester sau assessment
- Tạo VM mới nếu cần retest

## Close Out
Dự án được đóng khi:
- Final report đã gửi
- Khách hàng đã được hỗ trợ câu hỏi/remediation
- Retest đã xong nếu có
- Dữ liệu/artifact đã được xử lý đúng
- Invoice/payment hoàn tất
- Có thể gửi survey để cải thiện quy trình

## Bài học quan trọng
Pentester chuyên nghiệp không chỉ giỏi khai thác. Khách hàng thường nhớ cách pentester giao tiếp, hỗ trợ và làm việc có trách nhiệm hơn là exploit chain phức tạp.