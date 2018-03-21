---
title: PAT advanced 1007
tags:
  - PAT
categories: []
date: 2017-02-09 19:01:40
---

## Description

> 1007.Maximum Subsequence Sum (25)

> Given a sequence of K integers { N1, N2, ..., NK }. A continuous subsequence is defined to be { Ni, Ni+1, ..., Nj } where 1 <= i <= j <= K. The Maximum Subsequence is the continuous subsequence which has the largest sum of its elements. For example, given sequence { -2, 11, -4, 13, -5, -2 }, its maximum subsequence is { 11, -4, 13 } with the largest sum being 20.

> Now you are supposed to find the largest sum, together with the first and the last numbers of the maximum subsequence.

> Input Specification:

> Each input file contains one test case. Each case occupies two lines. The first line contains a positive integer K (<= 10000). The second line contains K numbers, separated by a space.

> Output Specification:

> For each test case, output in one line the largest sum, together with the first and the last numbers of the maximum subsequence. The numbers must be separated by one space, but there must be no extra space at the end of a line. In case that the maximum subsequence is not unique, output the one with the smallest indices i and j (as shown by the sample case). If all the K numbers are negative, then its maximum sum is defined to be 0, and you are supposed to output the first and the last numbers of the whole sequence.

> Sample Input:
10
-10 1 2 3 4 -5 -23 3 7 -21

>Sample Output:
10 1 4

第一次超时了O(n^3)，后来改成O(n^2)了。

## Code

```
#include <cstdio>
#include <queue>
#include <vector>
#include <cstring>
#include <string>
#include <iostream>
#include <algorithm>
#include <climits>

using namespace std;

const int maxN = 1e4 + 10;

int q[maxN];
int n;
int tmp;
int flag = 0;
int ans = numeric_limits<int>::min();
int f, l;

int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
		cin >> q[i];
	for (flag = 0; flag < n && q[flag] < 0; flag++);
	if (flag == n )
	{
		cout << 0 << ' ' << q[0] << ' ' << q[n - 1];
		return 0;
	}
	for (int i = 0; i < n; i++)
	{
		int sum = 0;
		for (int j = i; j < n; j++)
		{
			if ((sum += q[j]) > ans)
			{
				ans = sum;
				f = q[i];
				l = q[j];
			}
		}
	}
	cout << ans << ' ' << f << ' ' << l;
	return 0;
}
```