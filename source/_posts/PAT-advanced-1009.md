---
title: PAT advanced 1009
tags:
  - PAT
categories: []
date: 2017-02-09 22:20:20
---

## Description

> 1009.Product of Polynomials (25)

> Input Specification:

> Each input file contains one test case. Each case occupies 2 lines, and each line contains the information of a polynomial: K N1 aN1 N2 aN2 ... NK aNK, where K is the number of nonzero terms in the polynomial, Ni and aNi (i=1, 2, ..., K) are the exponents and coefficients, respectively. It is given that 1 <= K <= 10, 0 <= NK < ... < N2 < N1 <=1000.

> Output Specification:

> For each test case you should output the product of A and B in one line, with the same format as the input. Notice that there must be NO extra space at the end of each line. Please be accurate up to 1 decimal place.

> Sample Input
2 1 2.4 0 3.2
2 2 1.5 1 0.5

> Sample Output
3 3 3.6 2 6.0 1 1.6

居然一次过了。

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
typedef long long LL;

const int maxN = 1e3 + 10;
float ans[maxN + maxN];
float p1[maxN];
float p2[maxN];
int k1, k2, k3;

int main()
{
	cin >> k1;
	while (k1--)
	{
		int x; cin >> x; cin >> p1[x];
	}
	cin >> k2;
	while (k2--)
	{
		int x; cin >> x; cin >> p2[x];
	}
	for (int i = 0; i < maxN; i++)
	{
		if (p2[i])
			for (int j = 0; j < maxN; j++)
			{
				if (p1[j])
					ans[i + j] += p1[j] * p2[i];
			}
	}
	for (int i = 0; i < 2 * maxN; i++)
		if (ans[i])
			k3++;
	cout << k3;
	for (int i = 2 * maxN - 1; i >= 0; i--)
	{
		if (ans[i])
			printf(" %d %.1f", i, ans[i]);
	}
	return 0;
}
```