---
title: PAT advanced 1063
tags:
  - PAT
categories: []
date: 2017-02-18 18:32:49
---

## Description

> 1063.Set Similarity

>Given two sets of integers, the similarity of the sets is defined to be Nc/Nt\*100%, where Nc is the number of distinct common numbers shared by the two sets, and Nt is the total number of distinct numbers in the two sets. Your job is to calculate the similarity of any given pair of sets.

> Input Specification:

> Each input file contains one test case. Each case first gives a positive integer N (<=50) which is the total number of sets. Then N lines follow, each gives a set with a positive M (<=104) and followed by M integers in the range [0, 109]. After the input of sets, a positive integer K (<=2000) is given, followed by K lines of queries. Each query gives a pair of set numbers (the sets are numbered from 1 to N). All the numbers in a line are separated by a space.

> Output Specification:

> For each query, print in one line the similarity of the sets, in the percentage form accurate up to 1 decimal place.

> Sample Input:
3
3 99 87 101
4 87 101 5 87
7 99 101 18 5 135 18 99
2
1 2
1 3

> Sample Output:
50.0%
33.3%


学习了几点简单的优化吧：
+ `int`比`float`运算快
+ `scnaf`和`printf`比`cin`和`cout`运算快
+ 一段程序比多段程序运算快（是不是把编译时间算上去了？还是`call`是一个耗时的指令？）

## Code

```
#include<cstdio>
#include<iostream>
#include<set>

using namespace std;
typedef long long LL;

const int maxN = 5e1 + 10;

set<int> s[maxN];
int n, ni;
int a, b;
int x, o;
int inN = 0;
int allN = 0;
float ans = 0;
set<int> ::iterator it;

int main()
{
	scanf("%d", &n);
	for (int i = 1; i <= n; i++)
	{
		scanf("%d", &ni);
		while (ni--)
		{
			scanf("%d", &x);
			s[i].insert(x);
		}
	}
	scanf("%d", &o);
	while (o--)
	{
		scanf("%d%d", &a, &b);
		inN = 0; allN = 0; ans = 0;
		for (it = s[a].begin(); it != s[a].end(); it++)
			if (s[b].find(*it) != s[b].end())inN++;
		allN = s[a].size() + s[b].size() - inN;
		ans = (float)inN / allN;
		printf("%.1f%%\n", ans * 100);
	}
	return 0;
}
```