---
title: 2017HITCON整理
tags:
  - PWN
  - writeup
categories: []
date: 2017-11-15 18:48:35
---

## Start

ruby的pwntools写一下，其中有一些小坑。

## 完美无瑕

用[seccomp-tools](https://github.com/david942j/seccomp-tools)可以发现其中存在seccomp规则。详细的说名在这篇文章中可以看到[Eigenstate : Seccomp Sandboxing](https://eigenstate.org/notes/seccomp)

```
root@kali ~/c/2/p/artifact# seccomp-tools dump ./artifact 
 line  CODE  JT   JF      K
=================================
 0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x00 0x10 0xc000003e  if (A != ARCH_X86_64) goto 0018
 0002: 0x20 0x00 0x00 0x00000020  A = args[2]
 0003: 0x07 0x00 0x00 0x00000000  X = A
 0004: 0x20 0x00 0x00 0x00000000  A = sys_number
 0005: 0x15 0x0d 0x00 0x00000000  if (A == read) goto 0019
 0006: 0x15 0x0c 0x00 0x00000001  if (A == write) goto 0019
 0007: 0x15 0x0b 0x00 0x00000005  if (A == fstat) goto 0019
 0008: 0x15 0x0a 0x00 0x00000008  if (A == lseek) goto 0019
 0009: 0x15 0x01 0x00 0x00000009  if (A == mmap) goto 0011
 0010: 0x15 0x00 0x03 0x0000000a  if (A != mprotect) goto 0014
 0011: 0x87 0x00 0x00 0x00000000  A = X
 0012: 0x54 0x00 0x00 0x00000001  A &= 0x1
 0013: 0x15 0x04 0x05 0x00000001  if (A == 1) goto 0018 else goto 0019
 0014: 0x1d 0x04 0x00 0x0000000b  if (A == X) goto 0019
 0015: 0x15 0x03 0x00 0x0000000c  if (A == brk) goto 0019
 0016: 0x15 0x02 0x00 0x0000003c  if (A == exit) goto 0019
 0017: 0x15 0x01 0x00 0x000000e7  if (A == exit_group) goto 0019
 0018: 0x06 0x00 0x00 0x00000000  return KILL
 0019: 0x06 0x00 0x00 0x7fff0000  return ALLOW
```

可以看出允许了`read`，`write`，`fstat`这几个函数功能，如果第三个参数是奇数则`kill`，所以`mmap`和`mprotect`不能申请执行段。同时如果`arg[2]`和`syscallnum`相同则可以执行，所以`open`是可以的。最后用`ORW`得到flag即可。