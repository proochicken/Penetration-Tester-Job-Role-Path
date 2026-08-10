# HTB Academy - 08 Proof of Concept

## Ý chính
Proof of Concept (PoC) là bằng chứng chứng minh một lỗ hổng hoặc ý tưởng khai thác là khả thi. Trong pentest, PoC giúp dev/admin xác nhận lỗ hổng tồn tại, tái hiện được lỗi, hiểu impact và kiểm tra remediation.

> PoC không chỉ để chứng minh “tôi exploit được”, mà để giúp khách hàng hiểu và sửa đúng vấn đề.

## PoC dùng để làm gì?
- Chứng minh lỗ hổng thật sự tồn tại.
- Cho thấy lỗ hổng có thể khai thác được.
- Giúp dev/admin tái hiện lỗi.
- Chứng minh impact.
- Làm cơ sở để khách hàng quyết định hành động tiếp theo.
- Giúp kiểm tra lại sau khi vá.

## Các dạng PoC
PoC có thể là:
- Tài liệu mô tả từng bước tái hiện lỗi.
- Screenshot.
- Request/response.
- Video/screen recording.
- Script/code khai thác.
- Attack chain walkthrough.

## PoC script không phải toàn bộ vấn đề
Một PoC script chỉ là một cách khai thác lỗ hổng. Nếu dev/admin chỉ sửa để script không chạy nữa, root cause có thể vẫn còn.

Cần nhấn mạnh:
- Không nên chỉ “fight against the script”.
- Phải sửa nguyên nhân gốc.
- Phải hiểu vấn đề bảo mật rộng hơn.
- Phải kiểm tra xem còn cách khai thác khác không.

## Attack Chain
Attack chain là chuỗi nhiều lỗi/hành động kết hợp lại:
Ví dụ:
1. Weak password
2. Truy cập SMB share
3. Lấy credential
4. Login host khác
5. Leo quyền
6. Domain compromise

Fix một mắt xích có thể chặn chain hiện tại, nhưng các lỗi còn lại vẫn cần sửa.

## Ví dụ Password123
Nếu user dùng `Password123`, vấn đề không chỉ là password của user đó. Root cause có thể là password policy yếu.

Nếu chỉ đổi password của một Domain Admin, tổ chức vẫn có thể còn nhiều user khác dùng password yếu. Cần sửa policy để ngăn password yếu tồn tại ngay từ đầu.

## Takeaway
PoC tốt phải chứng minh impact, giúp tái hiện lỗi, và hướng khách hàng tới remediation đúng root cause. Pentester không nên chỉ đưa exploit script, mà phải giải thích bức tranh tổng thể.