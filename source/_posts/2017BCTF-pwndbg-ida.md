---
title: 2017BCTF & pwndbg & ida
tags:
  - PWN
  - pwndbg
  - ida
categories: []
date: 2017-04-22 21:07:06
---

## 工具

pwndbg和ida可以很好的实现将idb文件导入到gdb调试界面中，使得gdb中的函数与变量以及结构体的名字可以和ida同步，并且在ida中可以实时看到调试的进度。

今天发现了这个功能还是很欣喜的，国内没人写，大概我是很早这样用的一批人吧。

pwndbg的README说的很详细了，不在赘述。

## babyuse

看了看别人的wp，总结了几种利用方法。
1. leak都用UAF
2. malloc一个大内存(>128)，会存在unsortbins中，可以leak出main_arena，计算出libc
3. 利用fastbin的链表结构，leak出heap_base
4. Nu1L用末尾添加\x00的方法，修改指针最后一位\x00使其指向vtable，leak出pie的地址
5. shell一般都用one_gadget，Nu1L的`_init_arry`没看懂。

为了方便调试，关闭ASLR，并且简单写了个gdb脚本。

```
# import glibc
# directory ~/desktop/glibc-2.24/malloc/

#######################################################
# context config
set ida-rpc-host 192.168.234.1

######################################################
# ignore SIGALRM
# handle SIGALRM ignore

#######################################################
define p32break
	break * 0x56555000 + $arg0
end

document p32break
	break when PIE is on
end

########################################################
define p32telescope
	telescope 0x56555000+$arg0 $arg1
end

document p32telescope
	telescope when PIE is on
end

#######################################################
source aa
define ssa
    session save aa
end
```

## 100levels