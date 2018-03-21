---
title: 一款rootkit分析
tags:
  - rookit
  - Linux内核
categories: []
date: 2017-07-31 16:33:18
---

## Makefile

## LKM学习

## adore-ng分析

## adore-ng从2.6内核升级为3.0内核

在内核升级的过程中遇到了一些问题，导致我耽误了很久在这个事情上。

### 内核的版本问题

在`#include <linux/version.h>`中存在关于内核的版本的定义
```
#define LINUX_VERSION_CODE 199182
#define KERNEL_VERSION(a,b,c) (((a)<< 16) + ((b)<< 8) + (c))
```

在使用时只要在预编译处定义即可
```
#if (LINUX_VERSION_CODE >= KERNEL_VERSION(3, 11, 0))
#else
#endif
```

### C中的内联函数

耐着性子看完了。。
[汇编语言---GCC内联汇编](http://www.cnblogs.com/taek/archive/2012/02/05/2338838.html)

并且在64位的机器上写汇编指令一定要精确操作符的位

[C uses assemble: operand type mismatch for push](https://stackoverflow.com/questions/21245245/c-uses-assemble-operand-type-mismatch-for-push)

| 后缀 | cpu位数 | 寄存器 |
| :-------------: | :-----------------------: |
| q | 64 | rax |
| l | 32 | eax |
| w | 16 | ax |

关于`static inline`和`extend inline`的区别什么的。。。

### 一些宏定义

由于CVE-2009-2009的漏洞（具体没看，大概是寄存器高地址的UAF造成的提权）的原因，3.6版本内核及以下的`sock_map_fd`等函数到3.7以后都用`SYSCALL_DEFINE3`的宏来定义，接口全部变为`sys_socket()`了。