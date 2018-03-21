---
title: PAT advanced 1010
tags:
  - PAT
categories: []
date: 2017-02-13 12:32:46
---

## Description

> 1010.Radix (25)

> Given a pair of positive integers, for example, 6 and 110, can this equation 6 = 110 be true? The answer is "yes", if 6 is a decimal number and 110 is a binary number.

> Now for any pair of positive integers N1 and N2, your task is to find the radix of one number while that of the other is given.

> Input Specification:

> Each input file contains one test case. Each case occupies a line which contains 4 positive integers:
N1 N2 tag radix
Here N1 and N2 each has no more than 10 digits. A digit is less than its radix and is chosen from the set {0-9, a-z} where 0-9 represent the decimal numbers 0-9, and a-z represent the decimal numbers 10-35. The last number "radix" is the radix of N1 if "tag" is 1, or of N2 if "tag" is 2.

> Output Specification:

> For each test case, print in one line the radix of the other number so that the equation N1 = N2 is true. If the equation is impossible, print "Impossible". If the solution is not unique, output the smallest possible radix.

> Sample Input 1:
6 110 1 10
Sample Output 1:
2

>Sample Input 2:
1 ab 1 2
Sample Output 2:
Impossible

自己写的超时了，后来改了下又没全过，找不到问题郁闷极了。。借鉴了别人的

## Code

```
#include <cstdio>
#include <cmath>
#include <queue>
#include <vector>
#include <cstring>
#include <string>
#include <iostream>
#include <algorithm>
#include <climits>

using namespace std;
typedef unsigned long long uLL;
const int maxn = 1e5 + 10;
string a, b;
int tag, radix, res;
uLL ans = 0, l, r;

uLL get(char ch)
{
	if ('0' <= ch&&ch <= '9') return ch - '0';
	return ch - 'a' + 10;
}

int main()
{
	cin >> a >> b >> tag >> radix;
	if (tag == 2) swap(a, b);
	for (int i = 0; a[i]; i++) ans = ans*radix + get(a[i]);
	for (int i = 0; b[i]; i++) l = max(l, get(b[i]));
	for (l++, r = ans + 1; l <= r;)
	{
		uLL mid = l + r >> 1;
		uLL check = 0;
		for (int i = 0; b[i]; i++) check = check*mid + get(b[i]);
		if (check == ans) res = mid;
		if (check >= ans) r = mid - 1; else l = mid + 1;
	}
	res ? printf("%d\n", res) : printf("Impossible\n");
	return 0;
}
```