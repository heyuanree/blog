---
title: Lurk Server
tags:
  - python
  - web
  - Trojan
  - flask
categories: []
date: 2018-03-13 00:55:52
---

## Bootstrap

### `modal`框无法弹出，页面变灰

因为嵌套样式冲突，我再`navbar`中写的，将其移出到`<header>`中即可。

## flask

### 400错误

CSRF开启的话需要注意，自定义头会造成400响应错误

## Trojan

### `gethostbyname()` 返回`NULL`，`socket()`失败

windows下使用套接字库都要先初始化`WSAStartup()`

### `malloc(0)`导致`free()`时候内存破坏

原因未知，改位`calloc(4, 1)`。猜测位内存为初始化，部分str系列函数判断错误