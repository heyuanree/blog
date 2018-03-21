---
title: win内核驱动编程
tags:
  - win驱动
categories: []
date: 2017-07-20 10:07:36
---

## 准备知识与环境
### 理论

Intel处理器分为四个权限ring0，ring1，ring2，ring3，由高到低。其中win使用了ring0与ring3两个权限。内核及内核拓展模块（驱动程序）运行在ring0级别上。

### 双机调试环境

工具：
+ windbg
+ WDK，这个需要在微软的官网上下载，然还能顺便把windbg给装了
+ virtualKD，这个软件用来调试内核和驱动，通过vmware的通信进程来给系统中断

过程：
+ 安装win7_x64的虚拟机，然后把virtualKD的target文件夹放入虚拟机中，安装文件。
+ 重新启动虚拟机，这时有一个virtualKD的启动选择项。
+ 开启virtualKD能够看到我们开启的虚拟机的进程。
+ 第一次用时需要配置windbg的路径，路径配置完成自动/手动打开windbg
+ 我们需要载入符号表，符号表在`File->Symbols Path`快捷键`^+s`配置，路径为`srv*c:\symbols *https://msdl.microsoft.com/download/symbols`，第一个`*`后为本地的下载目录，勾选上`Reload`。
+ 至此调试环境配置完成，我们可以查看微软的一些符号表了。

### 内核/驱动开发环境

+ VS2013
+ WDK 8.1