---
title: PAT advanced 1002
tags: [PAT]
categories: []
date: 2017-02-03 00:30:58
---

## Description

> 1002.A+B for Polynomials (25)

> Input

> Each input file contains one test case. Each case occupies 2 lines, and each line contains the information of a polynomial: K N1 aN1 N2 aN2 ... NK aNK, where K is the number of nonzero terms in the polynomial, Ni and aNi (i=1, 2, ..., K) are the exponents and coefficients, respectively. It is given that 1 <= K <= 10，0 <= NK < ... < N2 < N1 <=1000.

> Output

> For each test case you should output the sum of A and B in one line, with the same format as the input. Notice that there must be NO extra space at the end of each line. Please be accurate to 1 decimal place.

> Sample Input
2 1 2.4 0 3.2
2 2 1.5 1 0.5

> Sample Output
3 2 1.5 1 2.9 0 3.2

注意保留1位精度，注意把0挑出来

## Code

```
#include <cstdio>
#include <queue>
#include <vector>
#include <cstring>
#include <string>
#include <iostream>
#include <algorithm>

typedef long long LL;
using namespace std;

int k1;
int k2;
int k3 = 0;
int flag = 0;
float a[1001] = {0};
float b[1001] = {0};
float c[1001] = {0};


int main()
{
	cin >> k1;
	for (int i = 0; i < k1; i++){ int tmp; cin >> tmp; cin >> a[tmp]; }
	cin >> k2;
	for (int i = 0; i < k2; i++){ int tmp; cin >> tmp; cin >> b[tmp]; }
	for (int i = 1000; i > -1; i--)
	{ 
		c[i] = a[i] + b[i]; 
		if (c[i])
			k3++;
	}
	if (k3) { cout << k3 << ' '; }
	else { cout << k3 << endl; return 0; }
	for (int i = 0; !c[i]; i++){ flag = i + 1; }
	for (int i = 1000; i > -1; i--)
	{
		if (c[i] && i != flag)
			printf("%d %.1f ", i, c[i]);
		else if (c[i] && i == flag)
			printf("%d %.1f\n", i, c[i]);
	}
	return 0;
}
```
