---
title: LIEF部分中文文档
tags:
  - PWN
  - python
categories: []
date: 2017-12-13 23:07:20
---

# LIEF

## 分析和操作格式

这部分教程的目的是概述LIEF的API用来分析处理文件的格式
// 由学弟翻译整理
<!-- more -->

### ELF

- 我们从ELF格式开始。要从一个文件创建一个ELF.Binary,我们只需朝lief.parse()或lief.ELF.parse()函数传入他的路径。

>注意：如果使用的是python的API，那么lief.parse()和lief.ELF.parse()具有相同的行为。但是在C++中，LIEF::Parser::parse()将返回指向LIEF::Binary对象的指针，而LIEF::ELF::Parser::parse()将返回LIEF::ELF::**

```python
import lief
Binary = lief.parse("/bin/ls")
```

 一旦ELF文件被解析，我们可以访问它的Header

`header = binary.header`

 我们还可以更改它的入口点和目标架构(ARCH)

```python
header.entrypoint = 0x123
header.machine_type = lief.ELF.ARCH.AARCH64
```

并重建这个文件
`binary.write("ls.modified")`

- 我们也可以遍历这个二进制文件的段部分

```python
for section in sections:
    print section.name
    print section.size
    print len(section.content)
```

也可以修改它的`.text`部分

```python
text = binary.get_section(".text")
text.content = bytes([0x33] * text.size)
```

## 玩转ELF符号

在本教程中，我们将会介绍如何修改二进制及库中的动态符号。当二进制文件将要链接到库的时候，所需要用到的库储存在动态表的DT_NEEDED条目中，所需要的功能在表中注册并具有以下属性：

- `value` 设置为 `0`
- `种类` 设置为 `FUNC`类似的，当一个库导出函数时，它在动态表中有一个DT_SONAME条目，导出的函数在动态符号表中注册，并具有如下属性：
- `value` 设置为库中函数地址
- `type` 设置为 `FUNC`而导入导出函数由LIEF来抽象，因此你可以使用`exported_functions`和`imported_functions`来遍历这些元素

```python
import lief
binary  = lief.parse("/usr/bin/ls")
library = lief.parse("/usr/lib/libc.so.6")
print(binary.imported_functions)
print(library.exported_functions)
```

在分析二进制文件时，导入的函数名称对逆向工程非常有用。 一个解决方案是静态链接二进制文件和库。 另一个解决方案是通过交换这些符号来打击逆转者的思维。比如以下代码：

```C
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

double hashme(double input) {
  return pow(input, 4) + log(input + 3);
}

int main(int argc, char** argv) {
  if (argc != 2) {
    printf("Usage: %s N\n", argv[0]);
    return EXIT_FAILURE;
  }
  double N = (double)atoi(argv[1]);
  double hash = hashme(N);
  printf("%f\n", hash);
  return EXIT_SUCCESS;
}
```

这本书基本上是让这个程序接受一个整数作为参数，并对这个值进行一些计算。

```bash
$ hasme 123
228886645.836282
```

![](hashme.png)

该pow和log功能都位于libm.so.6库中。使用LIEF的一个有趣的技巧是将此函数名称与其他函数名称交换。在本教程中，我们将把他们交换为cos和sin。首先，我们必须加载库和二进制文件：

```python
＃/usr/bin/env python3
import lief
hasme = lief.parse("hasme")
libm = lief.parse("/usr/lib/libm.so.6")
```

然后当更改二进制中的两个导入函数的名称时：

```python
hashme_pow_sym = next(filter(lambda e : e.name == "pow", my_binary.imported_symbols))
hashme_log_sym = next(filter(lambda e : e.name == "log", my_binary.imported_symbols))
hashme_pow_sym.name = "cos"
hashme_log_sym.name = "sin"
```

最后我们在库中用`log`交换`sin`，用`pow`交换`cos`，然后重构两个对象

```python
#!/usr/bin/env python3
import lief


hasme = lief.parse("hasme")
libm  = lief.parse("/usr/lib/libm.so.6")


def swap(obj, a, b):
    symbol_a = next(filter(lambda e : e.name == a, obj.dynamic_symbols))
    symbol_b = next(filter(lambda e : e.name == b, obj.dynamic_symbols))
    b_name = symbol_b.name
    symbol_b.name = symbol_a.name
    symbol_a.name = b_name

hashme_pow_sym = next(filter(lambda e : e.name == "pow", my_binary.imported_symbols))
hashme_log_sym = next(filter(lambda e : e.name == "log", my_binary.imported_symbols))

hashme_pow_sym.name = "cos"
hashme_log_sym.name = "sin"


swap(libm, "log", "sin")
swap(libm, "pow", "cos")

hashme.write("hashme.obf")
libm.write("libm.so.6")
```

![](2.png)
有了这个脚本，我们`libm`在当前目录下建立了一个修改，我们必须强制Linux加载器在执行时使用这个`binary.obf`。为此，我们export环境变量LD_LIBRARY_PATH到当前目录：

```bash
$ LD_LIBRARY_PATH=. hashme.obf 123
228886645.836282
```

如果我们忽略它，它会使用默认`libm`和哈希计算完成`sin`和`cos`：

```bash
$ hashme.obf 123
-0.557978
```

一个真正的用例可能是在像OpenSSL这样的密码库中交换符号。例如`EVP_DecryptInit`，`EVP_EncryptInit`有相同的原型，所以我们可以交换它们。

## ELF挂钩

本教程的目标是钩住一个库函数
在前面的教程中，我们看到了如何从共享库中交换符号名称，现在我们将看到在共享库中挂钩函数的机制。

目标库是标准的数学库(`libm.so`)，我们将在exp函数中插入一个钩子，使得\(exp(x)= x + 1\)。下面的清单给出了使用这个函数的样例的源代码：

```C
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char **argv) {
  if (argc != 2) {
    printf("Usage: %s <a> \n", argv[0]);
    exit(-1);
  }

  int a = atoi(argv[1]);
  printf("exp(%d) = %f\n", a, exp(a));
  return 0;
}
```

挂钩功能如下:

```C
double hook(double x) {
  return x + 1;
}
```

编译`gcc -Os -nostdlib -nodefaultlibs -fPIC -Wl,-shared hook.c -o hook`

为了将这个钩子注入到库中，我们使用add()(段)方法

`Binary.add(*args, **kwargs)`

重载函数

1.add(self: _pylief.ELF.Binary, arg0: LIEF::ELF::DynamicEntry) -> LIEF::ELF::DynamicEntry

dynamic_entry

2.add(self: _pylief.ELF.Binary, section: LIEF::ELF::Section, loaded: bool=True) -> LIEF::ELF::Section

将给的`Section`添加到二进制文件中。
如果该部分不应加载到内存中，loaded参数必须设置为False（默认值：True）

3.add（self:_pylief.ELF.Binary, segment:LIEF::ELF::Segment, base:int = 0）-> LIEF::ELF::Segment

在二进制文件中添加一个段

4.add(self: _pylief.ELF.Binary, note: LIEF::ELF::Note) -> LIEF::ELF::Note

在二进制文件中添加一个行的`Note`一旦stub被注入，我们只需要改变exp符号的地址：

```python
exp_symbol  = libm.get_symbol("exp")
hook_symbol = hook.get_symbol("hook")

exp_symbol.value = segment_added.virtual_address + hook_symbol.value
```

测试修补过的库：

```bash
./do_math.bin 1
exp(1) = 2.718282
LD_LIBRARY_PATH=. ./do_math.bin 1
exp(1) = 2.000000
```

## 感染plt / got

本教程的目标是在ELF二进制文件中挂接导入的函数。
通过感染`.got`部分挂钩导入的函数是一个众所周知的技术，本教程将重点介绍使用LIEF的实现。

这些数字说明了这个plt/got机制：
![](3.png)
使用延迟绑定，第一次调用该函数时，该`got`条目将重定向到plt指令。
![](4.png)
第二次，`got`条目在共享库中保存地址
基本上感染分两步完成：

- 首先，我们注入我们的钩子
- 其次，我们通过打补丁将目标函数重定向到我们的钩子`got`

可以用下图来总结：
![](5.png)
作为例子，我们将使用一个基本的crackme在`memcmp`上的Flag和用户的输入。

```python
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Damn_YoU_Got_The_Flag
char password[] = "\x18\x3d\x31\x32\x03\x05\x33\x09\x03\x1b\x33\x28\x03\x08\x34\x39\x03\x1a\x30\x3d\x3b";

inline int check(char* input);

int check(char* input) {
  for (int i = 0; i < sizeof(password) - 1; ++i) {
    password[i] ^= 0x5c;
  }
  return memcmp(password, input, sizeof(password) - 1);
}

int main(int argc, char **argv) {
  if (argc != 2) {
    printf("Usage: %s <password>\n", argv[0]);
    return EXIT_FAILURE;
  }

  if (strlen(argv[1]) == (sizeof(password) - 1) && check(argv[1]) == 0) {
    puts("You got it !!");
    return EXIT_SUCCESS;
  }

  puts("Wrong");
  return EXIT_FAILURE;

}
```

这个Flag的值和`0x5c`进行了`xor`操作，为了验证crackme，用户必须输入`Damn_YoU_Got_The_Flag`:

```bash
$ crackme.bin foo
Wrong
$ crackme.bin Damn_YoU_Got_The_Flag
You got it !!
```

挂钩将包含打印参数`memcmp`并返回0：

```C
#include "arch/x86_64/syscall.c"
#define stdout 1

int my_memcmp(const void* lhs, const void* rhs, int n) {
  const char msg[] = "Hook memcmp\n";
  _write(stdout, msg, sizeof(msg));
  _write(stdout, (const char*)lhs, n);
  _write(stdout, "\n", 2);
  _write(stdout, (const char*)rhs, n);
  _write(stdout, "\n", 2);
  return 0;
}
```

由于钩子将被注入Creakme，因此它必须具备以下要求：

- 汇编代码必须是位置独立的(使用`-fPIC`或`-pie/-fPIE`标记编译)
- 不要使用外部库比如`libc.so`(标志)`-nostdlib -nodefaultlibs`(标志)

基于要求，这个钩子的编译为：`gcc -nostdlib -nodefaultlibs -fPIC -Wl,-shared hook.c -o hook`

### 注入钩子

第一步是将钩子注入bin。为此我们将添加一个`Segment`:

```python
import lief

crackme = lief.parse("crackme.bin")
hook    = lief.parse("hook")

segment_added  = crackme.add(hook.segments[0])
```

钩子的所有汇编代码都存在`hook`的第一段的`LOAD`中。
一旦钩子被添加，钩子的虚拟地址是`segment_added`的虚拟地址`virtual_address`,然后我们进行`got`patch。

### Patching `got`

LIEF提供了一个功能，可以轻松修补`got`与`Symbol`相关的条目：

`Binary.patch_pltgot（* args，** kwargs ）`

重载函数
1.patch_pltgot(self: _pylief.ELF.Binary, symbol_name: str, address: int) -> None

用导入的符号名称Patch `address`

2.patch_pltgot(self: _pylief.ELF.Binary, symbol: LIEF::ELF::Symbol, address: int) -> None

Patch导入的`Symbol`和`address`

`memcmp`函数的偏移量存储在value关联的动态符号的属性中。因此，它的虚拟地址将是：

- `my_memcpy`= `value` + `segment_added.virtual_address`

```python
my_memcmp      = hook.get_symbol("my_memcmp")
my_memcmp_addr = segment_added.virtual_address + my_memcmp.value
```

最后我们可以用`memcmp`的值来patch这个crackme.

`crackme.patch_pltgot('memcmp', my_memcmp_addr)`

最后rebuild

`crackme.write("crackme.hooked")`

### 运行
由于在检查标志值之前检查输入大小，我们必须提供正确长度的输入（不管其内容）：

```bash
$ crackme.hooked XXXXXXXXXXXXXXXXXXXXX
Hook add
Damn_YoU_Got_The_Flag
XXXXXXXXXXXXXXXXXXXXX
You got it !!
```
