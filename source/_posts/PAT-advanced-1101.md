---
title: PAT advanced 1101
tags:
  - PAT
categories: []
date: 2017-02-20 20:24:51
---

## Description

> 1101.Quick Sort

> There is a classical process named partition in the famous quick sort algorithm. In this process we typically choose one element as the pivot. Then the elements less than the pivot are moved to its left and those larger than the pivot to its right. Given N distinct positive integers after a run of partition, could you tell how many elements could be the selected pivot for this partition?

> For example, given N = 5 and the numbers 1, 3, 2, 4, and 5. We have:

> 1 could be the pivot since there is no element to its left and all the elements to its right are larger than it;
3 must not be the pivot since although all the elements to its left are smaller, the number 2 to its right is less than it as well;
2 must not be the pivot since although all the elements to its right are larger, the number 3 to its left is larger than it as well;
and for the similar reason, 4 and 5 could also be the pivot.
Hence in total there are 3 pivot candidates.

> Input Specification:

> Each input file contains one test case. For each case, the first line gives a positive integer N (<= 105). Then the next line contains N distinct positive integers no larger than 109. The numbers in a line are separated by spaces.

> Output Specification:

> For each test case, output in the first line the number of pivot candidates. Then in the next line print these candidates in increasing order. There must be exactly 1 space between two adjacent numbers, and no extra space at the end of each line.

> Sample Input:
5
1 3 2 4 5

> Sample Output:
3
1 4 5

用两个数组维护最大最小值

## Code

```
#include<cstdio>
#include<cmath>
#include<iostream>
#include<algorithm>
#include<limits>

using namespace std;
typedef long long LL;

const int maxN = 1e5 + 10;

int n;
int num[maxN];
int flag;
int minn[maxN];
int maxn[maxN];
int ans[maxN];
int tmp;

int main()
{
	scanf("%d", &n);
	num[0] = numeric_limits<int>::min();
	num[n + 1] = numeric_limits<int>::max();
	for (int i = 1; i <= n; i++)
		scanf("%d", &num[i]);
	for (int i = 1; i <= n; i++)
		maxn[i] = max(maxn[i - 1], num[i]);
	minn[n + 1] = num[n + 1];
	for (int i = n; i >= 1; i--)
		minn[i] = min(minn[i + 1], num[i]);
	for (int i = 1; i <= n; i++)
	{
		if (num[i] > maxn[i - 1] && num[i] < minn[i + 1])
		{
			ans[flag] = num[i];
			flag++;
		}
	}
	printf("%d\n", flag);
	for (int i = 0; i < flag; i++)
		(!i) ? printf("%d", ans[i]) : printf(" %d", ans[i]);
	printf("\n");
	return 0;
}
```