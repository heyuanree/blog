---
title: PAT advanced 1001
tags: [PAT]
categories: []
date: 2017-02-02 00:02:49
---

## Description

> 1001.A+B Format(20)
时间限制
400 ms
内存限制
65536 kB
代码长度限制
16000 B
判题程序
Standard
作者
CHEN, Yue
Calculate a + b and output the sum in standard format -- that is, the digits must be separated into groups of three by commas (unless there are less than four digits).

> Input

> Each input file contains one test case. Each case contains a pair of integers a and b where -1000000 <= a, b <= 1000000. The numbers are separated by a space.

> Output

> or each test case, you should output the sum of a and b in one line. The sum must be written in the standard format.

> Sample Input
-1000000 9
Sample Output
-999,991

暴力可以解决PAT上不少问题= =

## Code 

```
#include <cstdio>
#include <queue>
#include <vector>
#include <cstring>
#include <string>
#include <iostream>
#include <algorithm>

typedef long long LL;
using namespace std;

int main()
{
	int flag = 0;
	int j = 0;
	LL a, b, c;
	int num[20];
	cin >> a >> b;
	c = a + b;
	if (c < 0){ flag = 1; c = -c; }
	if (flag)cout << '-';
	if (c == 0)
	{
		num[0] = 0;
		j = 1;
	}
	while (c){ num[j] = c % 10; c = c / 10; j++; }
	for (int i = j - 1; i > -1; i--)
	{
		cout << num[i];
		if (i % 3 == 0 && i)
			cout << ',';
	}
		cout << endl;
	return 0;
}
```
