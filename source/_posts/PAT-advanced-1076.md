---
title: PAT advanced 1076
tags:
  - PAT
categories: []
date: 2017-02-21 14:54:07
---

## Description

> 1076.Forwards on Weibo

> Weibo is known as the Chinese version of Twitter. One user on Weibo may have many followers, and may follow many other users as well. Hence a social network is formed with followers relations. When a user makes a post on Weibo, all his/her followers can view and forward his/her post, which can then be forwarded again by their followers. Now given a social network, you are supposed to calculate the maximum potential amount of forwards for any specific user, assuming that only L levels of indirect followers are counted.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains 2 positive integers: N (<=1000), the number of users; and L (<=6), the number of levels of indirect followers that are counted. Hence it is assumed that all the users are numbered from 1 to N. Then N lines follow, each in the format:

> M[i] user_list[i]

> where M[i] (<=100) is the total number of people that user[i] follows; and user_list[i] is a list of the M[i] users that are followed by user[i]. It is guaranteed that no one can follow oneself. All the numbers are separated by a space.

> Then finally a positive K is given, followed by K UserID's for query.

> Output Specification:

> For each UserID, you are supposed to print in one line the maximum potential amount of forwards this user can triger, assuming that everyone who can view the initial post will forward it once, and that only L levels of indirect followers are counted.

> Sample Input:
7 3
3 2 3 4
0
2 5 6
2 3 1
2 3 4
1 4
1 5
2 2 6

> Sample Output:
4
5

哇，难得的一次过233。。。BFS

## Code

```
#include<cstdio>
#include<cmath>
#include<iostream>
#include<algorithm>
#include<limits>
#include<queue>

using namespace std;
typedef long long LL;

const int maxN = (1e3 + 10) + 1;
int n, l;
int m[maxN][maxN];
int lay[maxN];
bool vis[maxN];
int x, y;
int p;
int ans[maxN];
queue<int> q;

int bfs(int root, int l)
{
	int w; int flag = 1; int ans = 0;
	for (int i = 1; i < maxN; i++)
		vis[i] = false;
	q.push(root);
	vis[root] = true;
	lay[root] = 0;
	while (!q.empty())
	{
		flag = 1;
		root = q.front();
		q.pop();
		w = m[root][flag];
			while (w)
			{
				if (!vis[w])
				{
					vis[w] = true;
					lay[w] = lay[root] + 1;
					q.push(w);
					if (lay[w] <= l )
						ans++;
				}
				flag++;
				w = m[root][flag];
			}
	}
	return ans;
}

int main()
{
	scanf("%d%d", &n, &l);
	for (int i = 1; i <= n; i++)
	{
		scanf("%d", &x);
		for (int j = 1; j <= x; j++)
		{
			scanf("%d", &y);
			m[y][++m[y][0]] = i;
			//m[i][j] = y;
		}
	}
	scanf("%d", &p);
	for (int i = 1; i <= p; i++)
		scanf("%d", &ans[i]);
	for (int i = 1; i <= p; i++)
	{
		printf("%d\n", bfs(ans[i], l));
	}
	return 0;
}
```