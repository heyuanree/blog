---
title: PAT advanced 1078
tags:
  - PAT
categories: []
date: 2017-02-20 12:30:58
---

## Description

> 1078.Hashing

> The task of this problem is simple: insert a sequence of distinct positive integers into a hash table, and output the positions of the input numbers. The hash function is defined to be "H(key) = key % TSize" where TSize is the maximum size of the hash table. Quadratic probing (with positive increments only) is used to solve the collisions.

> Note that the table size is better to be prime. If the maximum size given by the user is not prime, you must re-define the table size to be the smallest prime number which is larger than the size given by the user.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains two positive numbers: MSize (<=104) and N (<=MSize) which are the user-defined table size and the number of input numbers, respectively. Then N distinct positive integers are given in the next line. All the numbers in a line are separated by a space.

> Output Specification:

> For each test case, print the corresponding positions (index starts from 0) of the input numbers in one line. All the numbers in a line are separated by a space, and there must be no extra space at the end of the line. In case it is impossible to insert the number, print "-" instead.

> Sample Input:
4 4
10 6 4 15

> Sample Output:
0 1 4 -

看书好长一段，看看被人的代码一下就懂了，QAQ
二次方探查就是用`(x + i * i) % Tsize`这么个东西吧。。
哦，想起来个注释小技巧，这样写`/*···············//*/`，去掉注释删除前一个`/*`即可

## Code

```
#include<cstdio>
#include<cmath>
#include<iostream>
#include<map>
#include<string>

using namespace std;
typedef long long LL;

const int maxS = 1e4 + 10;

int m, n;
int t[maxS] = {0};
int x;

bool prime(int x)
{
	if (x == 1)
		return false;
	if (x == 2)
		return true;
	if (x == 3)
		return true;
	for (int i = 2; i < sqrt(x) + 1; i++)
		if (x % i == 0)
			return false;
	return true;
}

int findp(int x)
{
	for (int i = x + 1;; i++)
		if (prime(i))
			return i;
}
//*/

int main()
{
	scanf("%d%d", &m, &n);
	
	if (!prime(m))
		m = findp(m);


	for (int i = 0; i < n; i++)
	{
		scanf("%d", &x);
		int flag = 0;
		if (i) printf(" ");
		for (int i = 0; i < m; i++)
		{
			int y = (x + i * i) % m;
			if (!t[y])
			{
				printf("%d", y);
				t[y] = 1;
				flag = 1;
				break;
			}
		}
		if (!flag) printf("-");
	}
	//*/
	return 0;
}
```