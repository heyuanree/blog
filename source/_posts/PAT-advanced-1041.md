---
title: PAT advanced 1041
tags:
  - PAT
categories: []
date: 2017-02-18 09:57:21
---

## Description

> 1041.Be Unique

> Being unique is so important to people on Mars that even their lottery is designed in a unique way. The rule of winning is simple: one bets on a number chosen from [1, 104]. The first one who bets on a unique number wins. For example, if there are 7 people betting on 5 31 5 88 67 88 17, then the second one who bets on 31 wins.

> Input Specification:

> Each input file contains one test case. Each case contains a line which begins with a positive integer N (<=105) and then followed by N bets. The numbers are separated by a space.

> Output Specification:

> For each test case, print the winning number in a line. If there is no winner, print "None" instead.

> Sample Input 1:
7 5 31 5 88 67 88 17
Sample Output 1:
31

> Sample Input 2:
5 888 666 666 888 888
Sample Output 2:
None

不会啥好算法，空间换时间。。用一个数组记录每个数字出现的次数。

## Code

```
#include <stack>
#include<cstdio>  
#include<string>  
#include<cstring>  
#include<vector>  
#include<iostream>  
#include<queue>  
#include<algorithm>  
using namespace std;
typedef long long LL;

const int maxN = 1e5 + 10;
const int maxM = 1e4 + 10;

int n, flag, k, ans = 0;
int m[maxN];
int c[maxM];

int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		cin >> m[i];
		c[m[i]]++;
	}
	for (int i = 0; i < n; i++)
		if (c[m[i]] == 1)
		{
			cout << m[i];
			return 0;
		}
	cout << "None";
	return 0;
}
```