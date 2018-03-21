---
title: pwn的一些小心得
tags:
  - pwndbg
  - libheap
categories: []
date: 2017-04-26 20:19:33
---

## libheaps与pwndbg兼容

其实本来也不存在conflict，就是几个命令用了同样的名字，所以我安装libheap的时候就改了几个命令的名字。

注册command的文件在这个目录下：
`/root/libheap/libheap/frontend/commands/gdb`

改几个注册名就好，下面以我把`heap`改为`libheap`为例，当然可能改成`heap_lib`这种更合适。。。
<!--more-->
```
class heap(gdb.Command):
    """libheap command help listing"""

# 主要就是__init__这里存在注册功能
    def __init__(self, debugger=None, version=None):
        super(heap, self).__init__("libheap", gdb.COMMAND_OBSCURE,
                                   gdb.COMPLETE_NONE)
```

当然，下面的功能名最好也改改。。。
```
    def invoke(self, arg, from_tty):
        # XXX: self.dbg.string_to_argv
        if arg.find("-h") != -1:
            # print_header("heap ", end="")
            # print("Options:", end="\n\n")
            # print_header("{:<15}".format("-a 0x1234"))
            # print("Specify an arena address")
            print_header("{:<15}".format("libheapls"))
            print("Print a flat listing of all chunks in an arena")
            print_header("{:<15}".format("libfastbins [#]"))
            print("Print all fast bins, or only a single fast bin")
            print_header("{:<15}".format("libsmallbins [#]"))
            print("Print all small bins, or only a single small bin")
            print_header("{:<15}".format("libfreebins"))
            print("Print compact bin listing (only free chunks)")
            print_header("{:<15}".format("libheaplsc"))
            print("Print compact arena listing (all chunks)")
            print_header("{:<15}".format("libmstats"), end="")
            print("Print memory alloc statistics similar to malloc_stats(3)")
            # print_header("{:<22}".format("print_bin_layout [#]"), end="")
            # print("Print the layout of a particular free bin")
            return
```

学习了一下gdb的python脚本写法，过两天改改pwndbg，添加点别的功能。。

## 新工具

[pwngdb一个专门用来打ctfpwn的gdb增强脚本](https://github.com/scwuaptx/Pwngdb)

用来功能的确好用。

## i386的libc-dbg安装

在使用heap命令时发现需要装32位的libc-dbg，于是上网查了查方法
```
dpkg --print-architecture
apt-get update
apt-get install libc6-dbg:i386
```

[Unable to locate package libc6-dbg:i386 in docker](https://askubuntu.com/questions/551840/unable-to-locate-package-libc6-dbgi386-in-docker/552273)

## gdb一些命令

`watch expr` 设置写watchpoint，当应用程序写expr, 修改其值时，程序停止运行
`rwatch expr`设置读watchpoint，当应用程序读表达式expr时，程序停止运行
`awatch expr`设置读写watchpoint, 当应用程序读或者写表达式expr时，程序都会停止运行

## 关于Kali

[Kali的拓容操作](http://blog.csdn.net/mazhuang2007/article/details/68925815)