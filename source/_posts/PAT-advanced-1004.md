---
title: PAT advanced 1004
tags:
  - PAT
categories: []
date: 2017-02-08 16:02:14
---

## Description

>  1004.Counting Leaves(30)

> A family hierarchy is usually presented by a pedigree tree. Your job is to count those family members who have no child.
Input

> Each input file contains one test case. Each case starts with a line containing 0 < N < 100, the number of nodes in a tree, and M (< N), the number of non-leaf nodes. Then M lines follow, each in the format:

> ID K ID[1] ID[2] ... ID[K]
where ID is a two-digit number representing a given non-leaf node, K is the number of its children, followed by a sequence of two-digit ID's of its children. For the sake of simplicity, let us fix the root ID to be 01.
Output

> For each test case, you are supposed to count those family members who have no child for every seniority level starting from the root. The numbers must be printed in a line, separated by a space, and there must be no extra space at the end of each line.

> The sample case represents a tree with only 2 nodes, where 01 is the root and 02 is its only child. Hence on the root 01 level, there is 0 leaf node; and on the next level, there is 1 leaf node. Then we should output "0 1" in a line.

> Sample Input
2 1
01 1 02
Sample Output
0 1

参考了排行榜里的一位前辈，学习了很多的东西，比如`vector`真的是一个很方便的东西；又比如`std::max(int a ,int b)`这个很方便的函数。很多在学数据结构的时候没有好好学习，数据结构还是要补一补。

不过这种不系统的学习方法真的好么？

还是要抽时间好好学一遍的。

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

typedef long long LL;
using namespace std;
const int maxN = 1e2 + 10;

int n;
int m;
int ans[maxN];
int deep = 0;

vector<int> tree[maxN];

void dfs(int i, int d)
{
	deep = max(deep, d);
	if (!tree[i].size()) ans[d]++;
	for (int j = 0; j < tree[i].size(); j++)
	{
		dfs(tree[i][j], d + 1);
	}
}

int main()
{
	scanf("%d%d", &n, &m);
	while (m--)
	{
		int id, d;
		scanf("%d%d", &id, &d);
		while (d--)
		{
			int y;
			scanf("%d", &y);
			tree[id].push_back(y);
		}
	}
	dfs(1, 0);
	for (int i = 0; i <= deep; i++)
		printf("%s%d", i ? " " : "", ans[i]);
	system("pause");
	return 0;
}
```