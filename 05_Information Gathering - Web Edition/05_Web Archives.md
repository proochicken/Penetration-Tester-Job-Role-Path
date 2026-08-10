## Web Archives - Wayback Machine

**Wayback Machine** là công cụ của Internet Archive cho phép xem lại các phiên bản cũ của website theo từng mốc thời gian. Nó lưu các bản chụp gọi là **captures/snapshots/archives**.

### Cách hoạt động

1. **Crawling**  
    Bot tự động duyệt web, đi theo các liên kết và tải bản sao của trang web.
    
2. **Archiving**  
    Nội dung được lưu lại, bao gồm HTML, CSS, JavaScript, hình ảnh và các tài nguyên liên quan. Mỗi bản lưu gắn với ngày giờ cụ thể.
    
3. **Accessing**  
    Người dùng nhập URL vào Wayback Machine và chọn thời điểm muốn xem lại phiên bản cũ của website.
    
![05_Web Archives-20260701084503822.png](05_Web%20Archives/05_Web%20Archives-20260701084503822.png)
### Vì sao hữu ích trong Web Recon?

Wayback Machine giúp pentester/security researcher:

- Tìm các endpoint, thư mục, file hoặc subdomain cũ.
    
- Phát hiện nội dung từng tồn tại nhưng hiện tại đã bị xóa.
    
- So sánh sự thay đổi của website theo thời gian.
    
- Thu thập OSINT như công nghệ cũ, nhân sự, email, chiến dịch marketing, API cũ.
    
- Recon thụ động vì không gửi request trực tiếp tới hạ tầng hiện tại của target.
    

### Ví dụ thông tin có thể tìm thấy

- `/backup.zip`
    
- `/old-admin/`
    
- `/dev/`
    
- `/api/v1/`
    
- File JS cũ chứa endpoint nội bộ
    
- Email nhân viên
    
- Subdomain hoặc tên hệ thống cũ
    
- Công nghệ/framework từng sử dụng
    

### Lưu ý

Wayback Machine không lưu mọi trang web. Một số trang có thể không có snapshot, bị chặn crawl, hoặc đã bị chủ website yêu cầu loại bỏ khỏi archive.