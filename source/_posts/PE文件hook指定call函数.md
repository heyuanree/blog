---
title: PE文件hook指定call函数
tags:
  - Win
  - RE
categories: []
date: 2018-01-27 21:26:16
---

## 简介

其实就是相当于CTF中的patch，在Kali中类似于Backdoor-factory，我们patch一个指定`call`，然后在最后加上jmp回来的代码即可。

## 用到的python库

**lief**
这个用了很多了，但是有个bug，可以说很大影响了在win10上的使用体验[Cannot add section and rebuild PE on Windows 10](https://github.com/lief-project/LIEF/issues/109)
这个库主要用来处理PE文件格式，以及patch函数。
**capstone**
反汇编引擎，看了说明，使用基本也很简单。看雪上有前辈说capstone对一些不常见的指令没有暴露接口，我还没有遇到过，也许以后需要魔改了。

## 一些小知识点

+ [`jmp`的各种跳转与机器码](http://blog.csdn.net/wzsy/article/details/17163589)
+ [python`struct`的一些处理](https://www.cnblogs.com/kuzhon/articles/5627977.html)

## 实现逻辑

+ 解析PE文件

+ 搜索符合条件的`call`指令

```
def record_call_address(binary):
    callAddress = []
    sectionTextRawData = ""
    sectionText = binary.get_section('.text')
    for i in sectionText.content:
        sectionTextRawData += struct.pack('B', i)
    md = Cs(CS_ARCH_X86, CS_MODE_32)
    for i in md.disasm(sectionTextRawData, sectionText.virtual_address):
        if i.mnemonic == 'call':
            callAddress.append(i.address)
    return callAddress
```

首先得到`.text`段的`content`，这里要注意这个格式是`list`，里面的成员是`int`类型的，要把他们16进制保存。

+ 初始化shellcode

这里pwntools上是用的binutils实现的，为了全平台的兼容型，以后可能会在windows上使用MinGW，在linux上用GCC编译。

+ 初始化shellcode_section

```
def init_shellcode_section(binary, shellcode, virtualAddress, invokeAddress):
    section_shellcode = lief.PE.Section('stext')
    section_shellcode.content = list(ord(i) for i in shellcode+'\xe9'+'\xde\xad\xbe\xef')
    section_shellcode.virtual_address = virtualAddress
    section_shellcode.characteristics = lief.PE.SECTION_CHARACTERISTICS.CNT_CODE | \
                                        lief.PE.SECTION_CHARACTERISTICS.MEM_EXECUTE| \
                                        lief.PE.SECTION_CHARACTERISTICS.MEM_READ
    return section_shellcode
```

这里的跳转我还没想好，因为设计到shellcode的执行流程，我在想也许可以把这里的`jmp`放回shellcode的编译中处理

+ patch目标`call`函数

```
def patch_pe_call_address(binary, callAddress, shellcodeVirtualAddress):
    patchValue = []
    patchValue.append(ord("\xe9")) # jmp
    address = struct.pack("L", shellcodeVirtualAddress - callAddress + 5)
    address = [ord(i) for i in address]
    patchValue.extend(address) # here should be asm code
    print((hex(callAddress), patchValue))
    binary.patch_address(callAddress, patchValue)
```

+ 初始化builder

可能会有一些关于`ASLR`和`NX`等PE头的config

+ 保存修改后的PE文件
