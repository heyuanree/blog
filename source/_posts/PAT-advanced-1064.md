---
title: PAT advanced 1064
tags:
  - PAT
categories: []
date: 2017-02-21 22:53:48
---

## Descripton

> 1064.Complete Binary Search Tree

> A Binary Search Tree (BST) is recursively defined as a binary tree which has the following properties:

> The left subtree of a node contains only nodes with keys less than the node's key.
The right subtree of a node contains only nodes with keys greater than or equal to the node's key.
Both the left and right subtrees must also be binary search trees.
A Complete Binary Tree (CBT) is a tree that is completely filled, with the possible exception of the bottom level, which is filled from left to right.

> Now given a sequence of distinct non-negative integer keys, a unique BST can be constructed if it is required that the tree must also be a CBT. You are supposed to output the level order traversal sequence of this BST.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains a positive integer N (<=1000). Then N distinct non-negative integer keys are given in the next line. All the numbers in a line are separated by a space and are no greater than 2000.

> Output Specification:

> For each test case, print in one line the level order traversal sequence of the corresponding complete binary search tree. All the numbers in a line must be separated by a space, and there must be no extra space at the end of the line.

> Sample Input:
10
1 2 3 4 5 6 7 8 9 0

> Sample Output:
6 3 8 1 5 7 9 0 2 4

先建完全二叉树，然后中序遍历存放数值，最后bfs输出。
开心，好像一次过的情况变得多起来了~
可是为什么我的代码这么丑啊QAQ

## Code

```
#include<cstdio>
#include<iostream>
#include<algorithm>
#include<queue>

using namespace std;
typedef long long LL;

const int maxN = 1e3 + 10;

struct t{
	int d;
	int l;
	int r;
	int flag = 0;
}t[maxN];

int n;
int b[maxN];
int k = 1;

void inorder(int a)
{
	if (t[a].flag)
	{
		inorder(t[a].l);
		t[a].d = b[k++];
		inorder(t[a].r);
	}
}

void bfs(int root)
{
	queue<int> q;
	q.push(root);
	while (!q.empty())
	{
		if (t[t[q.front()].l].flag)
			q.push(t[q.front()].l);
		if (t[t[q.front()].r].flag)
			q.push(t[q.front()].r);
		(q.size() != 1) ? printf("%d ", t[q.front()].d) : printf("%d\n", t[q.front()].d);
		q.pop();
	}
}

int main()
{
	scanf("%d", &n);
	for (int i = 1; i <= n; i++)
		scanf("%d", b+i);
	sort(b+1, b+n+1);
	for (int i = 1; i <= n; i++)
	{
		t[i].flag = 1;
		if (n % 2 == 0 && i * 2 <= n)
		{
			if (i != n / 2)
			{
				t[i].l = i * 2;
				t[i].r = i * 2 + 1;
			}
			else
			{
				t[i].l = i * 2;
			}
		}
		if (n % 2 == 1 && i * 2 + 1 <= n)
		{
			t[i].l = i * 2;
			t[i].r = i * 2 + 1;
		}
	}
	inorder(1);
	bfs(1);
	return 0;
}
```