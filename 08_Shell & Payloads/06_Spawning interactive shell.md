# Spawning an Interactive Shell

## Mục tiêu

Sau khi có reverse shell hoặc system shell, session có thể là non-TTY và bị hạn chế:

- Không có prompt rõ ràng
    
- Không có job control
    
- Ctrl+C dễ làm chết shell
    
- `su`, `sudo`, `vim` hoạt động không ổn định
    
- Không có arrow key hoặc tab completion
    

Mục tiêu là dùng binary/ngôn ngữ có sẵn để gọi `/bin/sh` hoặc `/bin/bash`.

## Kiểm tra shell có sẵn

```bash
cat /etc/shells
ls -l /bin/sh /bin/bash
echo $SHELL
```

`$SHELL` chỉ cho biết login shell được cấu hình, không chắc là shell hiện tại.

Kiểm tra shell/process hiện tại:

```bash
ps -p $$ -o pid,ppid,tty,cmd
tty
```

## Các cách spawn shell

### `/bin/sh`

```bash
/bin/sh -i
```

- `-i`: interactive mode
    
- Có thể vẫn báo `no job control`
    

### Perl

```bash
perl -e 'exec "/bin/sh";'
```

### Ruby

```bash
ruby -e 'exec "/bin/sh"'
```

### Lua

```bash
lua -e "os.execute('/bin/sh')"
```

### AWK

```bash
awk 'BEGIN {system("/bin/sh")}'
```

### Find

```bash
find . -exec /bin/sh \; -quit
```

### Vim từ command line

```bash
vim -c ':!/bin/sh'
```

### Vim từ bên trong editor

```vim
:set shell=/bin/sh
:shell
```

Có thể thay `/bin/sh` bằng:

```text
/bin/bash
/bin/zsh
/bin/ash
```

## Kiểm tra binary/ngôn ngữ

```bash
command -v sh
command -v bash
command -v python3
command -v perl
command -v ruby
command -v lua
command -v awk
command -v find
command -v vim
```

## Kiểm tra quyền file

```bash
ls -la /path/to/file
```

Quyền ví dụ:

```text
-rwxr-xr--
```

- Owner: read, write, execute
    
- Group: read, execute
    
- Others: read
    

## Kiểm tra sudo

```bash
sudo -l
```

Output cần chú ý:

```text
(root) NOPASSWD: /path/to/binary
```

Nếu user được sudo một binary có khả năng gọi shell, đó có thể là đường privilege escalation.

## Ghi nhớ

- Non-TTY shell không nhất thiết là jail shell.
    
- Spawn shell không đồng nghĩa privilege escalation.
    
- `/bin/sh -i` tạo interactive shell nhưng có thể chưa có full TTY.
    
- Nhiều binary có chức năng command execution: AWK, Find, Vim, Perl, Ruby, Lua.
    
- Kiểm tra `sudo -l` sau khi có stable interactive shell.
    
- Đọc quyền và context trước khi chạy binary.