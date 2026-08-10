# Writing and Importing Metasploit Modules

## Tổng quan

Có ba tình huống chính:

1. Cập nhật Metasploit để lấy module mới đã được merge.
    
2. Import một module Metasploit có sẵn từ ExploitDB.
    
3. Port hoặc tự viết exploit thành Metasploit module bằng Ruby.
    

> File `.rb` chưa chắc là module Metasploit. Module phải có đúng class, metadata, mixin và API của Framework.

## Tìm module bằng Searchsploit

```bash
searchsploit nagios3
```

Tìm theo title và loại bỏ kết quả Python:

```bash
searchsploit -t Nagios3 --exclude=".py"
```

Không nên mặc định mọi file `.rb` đều dùng được trong `msfconsole`.

## Thư mục module

Framework system-wide:

```text
/usr/share/metasploit-framework/modules/
```

Custom module của user:

```text
~/.msf4/modules/
```

Ví dụ cấu trúc exploit web Unix:

```text
~/.msf4/modules/exploits/unix/webapp/
```

## Quy tắc đặt tên

Sử dụng snake_case:

```text
nagios3_command_injection.rb
```

Tránh dấu gạch ngang, khoảng trắng và ký tự đặc biệt.

## Import module

```bash
mkdir -p ~/.msf4/modules/exploits/unix/webapp/

cp 9861.rb \
~/.msf4/modules/exploits/unix/webapp/nagios3_command_injection.rb
```

Trong `msfconsole`:

```text
reload_all
search nagios3
use exploit/unix/webapp/nagios3_command_injection
show options
```

Có thể tải thêm module path:

```text
loadpath /path/to/modules/
```

Hoặc khi khởi động:

```bash
msfconsole -m /path/to/modules/
```

## Port exploit sang Metasploit

Porting không phải chỉ đổi đuôi file. Cần chuyển:

```text
Command-line arguments → Metasploit datastore options
HTTP library riêng     → HttpClient mixin
puts                    → print_status/print_good/print_error
Payload tự xử lý        → Metasploit payload
Session tự xử lý        → handler của Framework
```

Nên tìm module có chức năng gần giống để dùng làm boilerplate.

## Cấu trúc module cơ bản

```ruby
class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::HttpClient

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'Module name',
      'Description'    => %q{
        Module description
      },
      'License'        => MSF_LICENSE,
      'Author'         => ['Author'],
      'References'     => [
        ['CVE', 'YYYY-NNNN']
      ],
      'Targets'        => [
        ['Automatic', {}]
      ],
      'DefaultTarget'  => 0
    ))

    register_options(
      [
        OptString.new(
          'TARGETURI',
          [true, 'Base application path', '/']
        )
      ]
    )
  end
end
```

## Các mixin quan trọng

```ruby
Msf::Exploit::Remote::HttpClient
```

Hỗ trợ giao tiếp HTTP.

```ruby
Msf::Exploit::PhpEXE
```

Hỗ trợ sinh PHP payload.

```ruby
Msf::Exploit::FileDropper
```

Theo dõi và dọn file được upload lên target.

```ruby
Msf::Auxiliary::Report
```

Ghi host, service, credential và kết quả vào MSF database.

Chỉ include những mixin module thực sự cần.

## Register options

String option:

```ruby
OptString.new(
  'BLUDITUSER',
  [true, 'Username for Bludit']
)
```

File path option:

```ruby
OptPath.new(
  'PASSWORDS',
  [
    true,
    'Password wordlist',
    File.join(
      Msf::Config.data_directory,
      'wordlists',
      'passwords.txt'
    )
  ]
)
```

Truy cập option trong code:

```ruby
datastore['BLUDITUSER']
datastore['PASSWORDS']
```

## Bludit brute-force bypass

Bludit tin tưởng header:

```text
X-Forwarded-For
Client-IP
```

Attacker thay đổi `X-Forwarded-For` sau mỗi lần thử password. Ứng dụng tưởng request đến từ IP mới nên giới hạn brute-force theo IP bị bypass.

Quy trình:

```text
GET /admin/login
    ↓
Lấy tokenCSRF
    ↓
Đọc một password từ wordlist
    ↓
Thay đổi X-Forwarded-For
    ↓
POST username + password + tokenCSRF
    ↓
Kiểm tra redirect tới /admin/dashboard
```

## Ghi nhớ

- Đặt module đúng category và đúng directory.
    
- Dùng tên file snake_case.
    
- Chạy `reload_all` sau khi thêm hoặc sửa module.
    
- Đọc lỗi Ruby nếu module không load.
    
- `.rb` không đồng nghĩa với module MSF.
    
- Không giữ mixin hoặc code boilerplate không cần thiết.
    
- Dùng module tương tự làm mẫu, nhưng phải hiểu logic trước khi sửa.
    
- Chỉ thử nghiệm trên hệ thống được phép.

---
# Metasploit Custom Module Lab Checklist

## A. Tìm exploit

-  Xác định chính xác sản phẩm và phiên bản.
    
-  Tìm trong `msfconsole` trước:
    

```text
search <product>
```

-  Tìm trong ExploitDB:
    

```bash
searchsploit <product>
```

-  Kiểm tra exploit có ghi `(Metasploit)` hay không.
    
-  Kiểm tra file `.rb` có class `MetasploitModule` hay chỉ là Ruby script độc lập.
    
-  Đọc source trước khi chạy.
    

## B. Chuẩn bị thư mục

-  Xác định category:
    

```text
exploit/linux/http
exploit/unix/webapp
auxiliary/scanner/http
```

-  Tạo custom module directory:
    

```bash
mkdir -p ~/.msf4/modules/exploits/unix/webapp/
```

-  Đặt tên file theo snake_case.
    
-  Copy module vào đúng đường dẫn.
    

## C. Load module

-  Khởi động `msfconsole`.
    
-  Chạy:
    

```text
reload_all
```

-  Kiểm tra lỗi syntax hoặc dependency.
    
-  Tìm module:
    

```text
search <module-name>
```

-  Load module:
    

```text
use exploit/unix/webapp/<module-name>
```

-  Xem thông tin:
    

```text
info
```

-  Xem options:
    

```text
show options
```

## D. Cấu hình module

-  Đặt target:
    

```text
set RHOSTS <target-ip>
```

-  Đặt port:
    

```text
set RPORT <port>
```

-  Đặt SSL nếu dùng HTTPS:
    

```text
set SSL true
```

-  Đặt virtual host nếu cần:
    

```text
set VHOST <hostname>
```

-  Đặt URI đúng:
    

```text
set TARGETURI /
```

-  Đặt username/password/wordlist theo module.
    
-  Dùng `show missing` để kiểm tra option còn thiếu.
    
-  Dùng `check` nếu module hỗ trợ.
    

## E. Khi port exploit

-  Tìm một module tương tự làm boilerplate.
    
-  Xác định exploit thuộc loại HTTP, TCP, file format hay local.
    
-  Chọn đúng class cha.
    
-  Chọn đúng mixin.
    
-  Viết metadata chính xác.
    
-  Chuyển arguments thành `register_options`.
    
-  Chuyển input sang `datastore`.
    
-  Chuyển HTTP logic sang API Metasploit.
    
-  Xử lý redirect, cookie và CSRF token.
    
-  Sử dụng `print_status`, `print_good`, `print_error`.
    
-  Thêm kiểm tra response và trường hợp lỗi.
    
-  Không để credential hoặc target bị hard-code.
    
-  Chạy Ruby syntax check:
    

```bash
ruby -c custom_module.rb
```

-  Reload module và sửa toàn bộ lỗi.
    
-  Test với target lab đúng phiên bản.
    

## F. Debug

-  Module có nằm đúng thư mục không?
    
-  Tên file có dùng snake_case không?
    
-  Class có tên `MetasploitModule` không?
    
-  Có thiếu `end` không?
    
-  Mixin có tồn tại không?
    
-  Dependency Ruby có được cài không?
    
-  Method đang dùng có thuộc API Metasploit không?
    
-  Target URI, VHOST và SSL có đúng không?
    
-  CSRF token hoặc cookie có thay đổi theo request không?
    
-  Response thành công được nhận diện chính xác chưa?
    

## G. Sau khi test

-  Lưu source module.
    
-  Lưu output và bằng chứng.
    
-  Ghi lại phiên bản target.
    
-  Ghi lại option đã sử dụng.
    
-  Dọn file hoặc thay đổi do exploit tạo ra.
    
-  Không chạy ngoài scope được cấp phép.