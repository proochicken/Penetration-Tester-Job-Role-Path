# Metasploit Plugins

## Khái niệm

Plugin là thành phần mở rộng của Metasploit, thường được viết bằng Ruby (`.rb`).

Plugin có thể:

- Thêm command mới vào `msfconsole`.
    
- Tích hợp công cụ bên thứ ba.
    
- Tự động hóa tác vụ lặp lại.
    
- Truy cập Metasploit API và database.
    
- Quản lý host, service, vulnerability và session.
    
- Hỗ trợ discovery, pivoting và post-exploitation.
    

Plugin khác module:

```text
Module = thực hiện một chức năng cụ thể
Plugin = mở rộng workflow hoặc giao diện của Metasploit
```

## Thư mục plugin

```bash
ls /usr/share/metasploit-framework/plugins
```

Đường dẫn mặc định:

```text
/usr/share/metasploit-framework/plugins
```

Plugin thường là file Ruby:

```text
plugin_name.rb
```

## Load plugin

```text
load nessus
```

Nếu thành công:

```text
Successfully loaded Plugin: Nessus
```

Xem help của plugin:

```text
nessus_help
```

Hoặc kiểm tra menu chung:

```text
help
```

Nếu plugin không tồn tại hoặc không load được:

```text
Failed to load plugin
cannot load such file
```

Nguyên nhân thường gặp:

- Sai tên.
    
- Sai thư mục.
    
- Sai quyền.
    
- Thiếu dependency.
    
- Plugin lỗi hoặc không tương thích.
    

## Cài plugin bên thứ ba

```bash
git clone https://github.com/darkoperator/Metasploit-Plugins
```

Copy plugin:

```bash
sudo cp ./Metasploit-Plugins/pentest.rb \
  /usr/share/metasploit-framework/plugins/pentest.rb
```

Mở Metasploit:

```bash
msfconsole -q
```

Load:

```text
load pentest
```

Kiểm tra command mới:

```text
help
```

## Một số chức năng của Pentest Plugin

Discovery:

```text
discover_db
network_discover
pivot_network_discover
show_session_networks
```

Multi-session:

```text
multi_cmd
multi_meter_cmd
multi_post
```

Credential collection:

```text
app_creds
sys_creds
```

Auto exploit:

```text
show_client_side
vuln_exploit
```

Scanner finding phải được xác minh trước khi exploit vì có thể có false positive.

## Lưu ý bảo mật

Load plugin đồng nghĩa với thực thi code Ruby trên máy pentest.

Trước khi dùng plugin bên thứ ba:

- Kiểm tra nguồn.
    
- Kiểm tra tác giả.
    
- Đọc code.
    
- Kiểm tra dependency.
    
- Kiểm tra compatibility.
    
- Test trong lab.
    
- Không dùng repository không đáng tin cậy.
    
- Không chạy automation hàng loạt khi chưa xác nhận scope.
    

## Mixins

Metasploit được viết bằng Ruby.

Mixin cho phép một class dùng chức năng từ module thông qua:

```ruby
include ModuleName
```

Mixin được dùng khi:

- Một chức năng cần dùng lại trong nhiều class.
    
- Muốn thêm tính năng tùy chọn cho class.
    
- Muốn tránh cây inheritance phức tạp.
    

Người mới sử dụng Metasploit chưa cần viết Mixin, nhưng nên biết đây là cơ chế giúp framework tái sử dụng và mở rộng code.