---
title: Bin选手的武器库
tags:
  - weapon
categories: []
date: 2017-06-08 11:54:42
---

Bin工具整理，给出描述与使用经验。
<!--more-->

## ida Pro

反汇编神器。同时还有一份github上的插件[整理](https://github.com/onethawt/idaplugins-list)
一下列出使用过推荐的一些插件

| Name | Description |
|:----:|:-----:|
| Hexray | ida反编译神器，又称f5 |
| snowman decompiler | F3反编译工具(感觉不是很好用) |
| HexRayCodeXploer | 自动类型重建及对象浏览 |
| VMAttack | 主要用来对抗虚拟机与混淆，可提供自动与半自动的分析 |
| Hexlight | 按B可从一个花括号跳到另一个 |

## Ollydbg

win平台下使用最广泛ring3动态调试器。

## gdb

linux平台下调试器，同时有多个增强简脚本供选择。

| Name | Description |
|:-----------:|:-----------------:|
| [pwngdb](https://github.com/scwuaptx/Pwngdb) | 专门用来pwn的gdb增强脚本 |
| [pwndbg](https://github.com/pwndbg/pwndbg)| 个人体验最好的gdb增强脚本 |