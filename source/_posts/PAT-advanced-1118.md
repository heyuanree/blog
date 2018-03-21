---
title: PAT advanced 1118
tags:
  - PAT
categories: []
date: 2017-02-19 10:37:08
---

## Description

> 1118.Birds in Forest

> Some scientists took pictures of thousands of birds in a forest. Assume that all the birds appear in the same picture belong to the same tree. You are supposed to help the scientists to count the maximum number of trees in the forest, and for any pair of birds, tell if they are on the same tree.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains a positive number N (<= 104) which is the number of pictures. Then N lines follow, each describes a picture in the format:
K B1 B2 ... BK
where K is the number of birds in this picture, and Bi's are the indices of birds. It is guaranteed that the birds in all the pictures are numbered continuously from 1 to some number that is no more than 104.

> After the pictures there is a positive number Q (<= 104) which is the number of queries. Then Q lines follow, each contains the indices of two birds.

> Output Specification:

> For each test case, first output in a line the maximum possible number of trees and the number of birds. Then for each query, print in a line "Yes" if the two birds belong to the same tree, or "No" if not.

> Sample Input:
4
3 10 1 2
2 3 4
4 1 5 7 8
3 9 6 4
2
10 5
3 7

> Sample Output:
2 10
Yes
No


查并集问题，知道了就好做了。

## Code

```
#include<cstdio>
#include<iostream>
#include<set>

using namespace std;
typedef long long LL;

const int maxN = 1e4 + 10;
const int maxQ = 1e4 + 10;

int n, q;
int x, y, z;
int a, b;
int pre[maxN + 1];
int fa[maxN + 1];
bool e[maxN + 1];
int bn, tn;

int find(int x)
{
	int r = x;
	while (pre[r] != r)
		r = pre[r];
	int i = x, j;
	while (i != r)
	{
		j = pre[i];
		pre[i] = r;
		i = j;
	}
	return r;
}

void join(int x, int y)
{
	int fx = find(x), fy = find(y);
	if (fx != fy)
		pre[fy] = fx;
}

int main()
{
	scanf("%d", &n);
	for (int i = 1; i <= maxN; i++)
		pre[i] = i;
	while (n--)
	{
		scanf("%d", &x);
		scanf("%d", &y);
		e[y] = true;
		for (int i = 0; i < x - 1; i++)
		{
			scanf("%d", &z);
			e[z] = true;
			if (z != find(z))
			{
				join(y, z);
				fa[z] = find(y);
			}
			else
			{
				join(z, y);
				fa[y] = find(z);
			}
		}
	}
	for (int i = 1; i <= maxN + 1; i++)
		if (e[i])
		{
			bn++;
			if (pre[i] == i)tn++;
		}
	printf("%d %d\n", tn, bn);
	scanf("%d", &q);
	while (q--)
	{
		scanf("%d%d", &a, &b);
		(find(a) == find(b)) ? printf("Yes\n") : printf("No\n");
	}
	return 0;
}
```