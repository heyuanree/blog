---
title: C++ STL Map
tags:
  - STL
categories: []
date: 2017-02-19 11:36:24
---

## map

`map`是STL的一个关联容器，提供`key`到`value`映射，类似`python`里的字典的功能。`map`内自建一棵红黑树，具有对数据自动排序的功能，所以`map`内部的所有数据都是有序的。

`#include<map>`
`map<key, value> key_to_value`

## 成员函数

|函数|用法|
|:--:|:--:|
|begin()|返回指向`map`头部的迭代器|
|clear()|删除所有元素|
|count(elem)|返回指定出现的次数|
|empty()|判断是否空|
|end()|指向`map`末尾的迭代器|
|equal_range()|返回特殊条目的迭代器对|
|erase()|删除一个元素|
|find()|查找一个元素|
|get_allocator()|返回`map`的配置器|
|insert()|插入元素|
|key_comp()|返回比较元素`key`的函数|
|lower_bound()|返回键值>=给定元素的第一个位置|
|max_size()|返回可以容纳的最大元素数|
|rebign()|返回一个可以指向`map`尾部的逆向迭代器|
|rend()|返回一个指向`map`头部的逆向迭代器|
|size()|返回`map`中元素的数目|
|swap()|交换两个`map`|
|upper_bound()|返回键值>给定元素的第一个位置|
|value_comp()|返回比较`value`的函数|