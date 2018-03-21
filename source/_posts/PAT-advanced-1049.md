---
title: PAT advanced 1049
tags:
  - PAT
categories: []
date: 2017-02-21 20:18:39
---

## Description

> 1049.Counting Ones

> The task is simple: given any positive integer N, you are supposed to count the total number of 1's in the decimal form of the integers from 1 to N. For example, given N being 12, there are five 1's in 1, 10, 11, and 12.

> Input Specification:

> Each input file contains one test case which gives the positive N (<=230).

> Output Specification:

> For each test case, print the number of 1's in one line.

> Sample Input:
12

> Sample Output:
5

发现了别人的更好的解法(想法一样，实现比我好)，贴上来

> 如果这位大于1，那么1的总数等于（左边的值+1）\*（右边的位数）
如果这位等于1，那么1的总数等于（左边的值）\*（右边的位数）+右边的值
如果这位等于0，那么1的总数等于（左边的值）\*（右边的位数）

## Code

```
#include<cstdio>
#include<string>
#include<cmath>
#include<iostream>
#include<algorithm>
#include<limits>
#include<queue>

using namespace std;
typedef long long LL;

int n;
int r, l, it;
int ans = 0;

int main()
{
	cin >> n;
	int rr = 0;;
	for (int i = 0, l = n / 10, r = 1; n; n /= 10, l /= 10, r *= 10)
	{
		int x = n % 10;
		if (x > 1)
			ans += (l + 1) * r;
		if (x == 1)
			ans += l * r + rr;
		if (x == 0)
			ans += l * r;
		rr += n % 10 * r;
	}
	printf("%d\n", ans);
	return 0;
}
```