---
title: nasm使用小结
tags:
  - NASM
categories: []
date: 2018-02-08 10:49:03
---

## nasm

nams是一个跨平台的编译器，在win与linux上有很好的适配性，这也是我放弃了ketstone——keystone实在限制太多，抽象过高，一些底层工作没法开展，但是在便捷性上依然是不二选择——选择它的原因之一。

## 指令与功能

+ `nasm -f`表示编译出来的文件格式，`nasm -hf`可以查看支持的输出文件格式。

```
valid output formats for -f are (`*' denotes default):
  * bin       flat-form binary files (e.g. DOS .COM, .SYS)
    ith       Intel hex
    srec      Motorola S-records
    aout      Linux a.out object files
    aoutb     NetBSD/FreeBSD a.out object files
    coff      COFF (i386) object files (e.g. DJGPP for DOS)
    elf32     ELF32 (i386) object files (e.g. Linux)
    elf64     ELF64 (x86_64) object files (e.g. Linux)
    elfx32    ELFX32 (x86_64) object files (e.g. Linux)
    as86      Linux as86 (bin86 version 0.3) object files
    obj       MS-DOS 16-bit/32-bit OMF object files
    win32     Microsoft Win32 (i386) object files
    win64     Microsoft Win64 (x86-64) object files
    rdf       Relocatable Dynamic Object File Format v2.0
    ieee      IEEE-695 (LADsoft variant) object file format
    macho32   NeXTstep/OpenStep/Rhapsody/Darwin/MacOS X (i386) object files
    macho64   NeXTstep/OpenStep/Rhapsody/Darwin/MacOS X (x86_64) object files
    dbg       Trace of all info passed to output stage
    elf       ELF (short name for ELF32)
    macho     MACHO (short name for MACHO32)
    win       WIN (short name for WIN32)
```

其中支持将shellcode独立生成为exe文件。

+ `-Ox`表示优化级别，其中`x`为`1~n`优化级别递增

## 关于section和segment

新声明一个section的时候`.text``.data``.bss`这三个section会默认偏移是`0`，可以通过指定`[section .text vstart=0x1234]`的形式来指定偏移。

同时还有其他的属性可以指定，如`align`，`RWX`等，详情参见[nasm官方文档](http://www.nasm.us/xdoc/2.13.03/html/nasmdoc0.html)

## nasm的特性

`$`表示当前指令的地址，`$$`表示当前段的地址，`$-$$`表示距离段的偏移

在我的一段代码中遇到了很麻烦的重定位问题...最后还是通过这个不那么优雅的方法解决的

```asm
shellcode:
	db 0x12, 0x34, 0x12, 0x34,

start:
	mov eax, [shellcode]
```
这里的`mov eax, [shellcode]`会被nasm处理成`mov eax, ds:[xx]`这种类似的形式，但是我是把`shellcode`加在`.text`节（代码段）中的，所以应该是用`cs:[xx]`来寻址，于是有了两个选择：

1. 保存`ds`寄存器的值，再把`cs`赋值给`ds`
2. 通过`call $+5; pop ebx`来重定位`shellcode`的地址

最后我选择了第二种，汇编代码变成了如下形式

```asm
shellcode:
	call $+5
    pop ebx
    ret
    db 0x12, 0x34, 0x12, 0x34

start:
	call shellcode
    mov eax, [ebx+2]
```

说时候也只能说比第一种好了，但是对杀软，这个动作可以说相当奇怪了

## reference

[[Intel汇编-NASM]基本语法](http://blog.csdn.net/Lirx_Tech/article/details/42340619)
[学习 nasm 语言](http://www.mouseos.com/assembly/nasm03.html#023)