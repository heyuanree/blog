---
title: C++ STL poriority_queue
tags:
  - STL
categories: []
date: 2017-03-17 12:36:37
---

## poriority_queue

排在队首的是优先级最高的节点，越大的`int`优先级越高。对于自定义的数据类型，不需要准确的定义优先级的大小，只要能相互比较大小即可。

模板定义如下：

```
template <class T, class Container = vector<T>, class Compare = less<typename Container::value_type> > class priority_queue;
```

## 成员函数

|函数|用法|
|:--:|:--:|
|empty()|判断是否非空|
|size()|返回有限队列大小|
|top()|返回优先级最大的node的值|
|push()|插入一个元素|
|emplace()|建立并插入一个元素|
|pop()|弹出优先队列的优先级最大的node|
|swap()|交换两个优先队列|