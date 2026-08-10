# Metasploit Targets

## Khái niệm

Trong Metasploit:

```text
RHOSTS = IP máy đích
TARGET = profile kỹ thuật của exploit
```

`TARGET` giúp exploit thích nghi với một phiên bản OS/application cụ thể.

Target có thể khác nhau theo:

- OS và service pack.
    
- Phiên bản application.
    
- Architecture.
    
- Language pack.
    
- DLL/version.
    
- Return address.
    
- ROP chain.
    
- Exploit delivery method.
    

## Xem target

Phải chọn exploit trước:

```text
use <exploit_module>
show targets
```

Nếu chạy từ root menu:

```text
msf6 > show targets
[-] No exploit module selected.
```

## Chọn target

```text
show targets
set target <ID>
```

Ví dụ:

```text
set target 6
```

Có thể tương ứng với:

```text
IE 9 on Windows 7
```

## Automatic Target

```text
set target 0
```

`Automatic` cho phép module tự chọn strategy dựa trên thông tin phát hiện được.

Automatic không luôn chính xác. Nếu biết chắc phiên bản target, có thể chọn profile thủ công.

## Lệnh quan trọng

```text
info
show targets
show options
set target <ID>
```

`info` nên được chạy trước khi dùng module mới để xem:

- Vulnerability.
    
- Supported targets.
    
- Dependencies.
    
- Rank.
    
- Check support.
    
- References.
    
- Side effects.
    

## Browser Exploit Example

Module:

```text
exploit/windows/browser/ie_execcommand_uaf
```

Targets:

- IE 7 / Windows XP SP3.
    
- IE 8 / Windows XP SP3.
    
- IE 7 / Windows Vista.
    
- IE 8 / Windows Vista.
    
- IE 8 / Windows 7.
    
- IE 9 / Windows 7.
    

Module mở web server và yêu cầu victim truy cập exploit page bằng browser bị ảnh hưởng.

Các option:

```text
SRVHOST
SRVPORT
URIPATH
SSL
OBFUSCATE
```

## Use-After-Free

Object bị giải phóng nhưng chương trình vẫn tiếp tục sử dụng.

```text
Allocate → Free → Reuse dangling pointer
```

Attacker cố chiếm vùng memory đã được giải phóng để điều khiển dữ liệu mà chương trình sử dụng.

## Return Address và ROP

Target profile có thể chứa:

- Return address.
    
- `jmp esp`.
    
- `pop/pop/ret`.
    
- ROP gadgets.
    
- ROP chain.
    
- Memory offsets.
    

Các giá trị này có thể thay đổi giữa OS, service pack, language pack và software version.

## Target Identification

Để tạo target mới hoặc xác minh target thủ công:

1. Lấy đúng binary/DLL từ môi trường target.
    
2. Kiểm tra version và architecture.
    
3. Phân tích security protections.
    
4. Tìm instruction hoặc gadget phù hợp.
    
5. Kiểm tra bad characters và địa chỉ ổn định.
    
6. Test trong lab tương ứng.
    

## Ghi nhớ

Chọn sai target có thể gây:

- Exploit thất bại.
    
- Browser/service crash.
    
- Return address không hợp lệ.
    
- ROP chain không hoạt động.
    
- Payload không được thực thi.