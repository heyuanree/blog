---
title: PAT advanced 1102
tags:
  - PAT
categories: []
date: 2017-03-03 20:11:10
---

## Description

> 1102.Invert a Binary Tree

> Google: 90% of our engineers use the software you wrote (Homebrew), but you can't invert a binary tree on a whiteboard so fuck off.

> Now it's your turn to prove that YOU CAN invert a binary tree!

> Input Specification:

> Each input file contains one test case. For each case, the first line gives a positive integer N (<=10) which is the total number of nodes in the tree -- and hence the nodes are numbered from 0 to N-1. Then N lines follow, each corresponds to a node from 0 to N-1, and gives the indices of the left and right children of the node. If the child does not exist, a "-" will be put at the position. Any pair of children are separated by a space.

> Output Specification:

> For each test case, print in the first line the level-order, and then in the second line the in-order traversal sequences of the inverted tree. There must be exactly one space between any adjacent numbers, and no extra space at the end of the line.

> Sample Input:
8
1 -
\- -
0 -
2 7
\- -
\- -
5 -
4 6

> Sample Output:
3 7 2 6 4 0 5 1
6 5 7 4 3 2 0 1

dfs+inorder，考前练手。。

## Code

```
#include<cstdio>
#include<iostream>
#include<algorithm>
#include<vector>
#include<queue>

using namespace std;
typedef long long LL;

const int maxN = 10;

int n;
int node[maxN][2];
char x;
int y;
int z;
int root[maxN];
int flag;

void dfs(int root)
{
	queue<int> q;
	q.push(root);
	while (!q.empty())
	{
		int tmp;
		tmp = q.front();
		q.pop();
		tmp == root ? printf("%d", tmp) : printf(" %d", tmp);
		if (node[tmp][1] != -1)
			q.push(node[tmp][1]);
		if (node[tmp][0] != -1)
			q.push(node[tmp][0]);
	}
	printf("\n");
}

void inorder(int root)
{
	if (node[root][1] != -1) inorder(node[root][1]);
	++z != n ? printf("%d ", root): printf("%d\n", root);
	if (node[root][0] != -1) inorder(node[root][0]);
}

int main()
{
	scanf("%d", &n);
	getchar();
	for (int i = 0; i < n; i++)
	{
		((x = getchar()) != 45) ? (node[i][0] = x - '0') : (node[i][0] = -1);
		getchar();
		((x = getchar()) != 45 )? (node[i][1] = x - '0') : (node[i][1] = -1);
		getchar();
	}
	for (int i = 0; i < n; i++)
	{
		y = node[i][0];
		if (y != -1)
			root[y] = 1;
		y = node[i][1];
		if (y != -1)
			root[y] = 1;
	}
	for (int i = 0; i < n; i++)
		if (root[i] != 1)
		{
			flag = i;
			break;
		}
	dfs(flag);
	inorder(flag);
	return 0;
}
```