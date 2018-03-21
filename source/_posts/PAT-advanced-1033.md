---
title: PAT advanced 1033
tags:
  - PAT
categories: []
date: 2017-02-16 18:27:31
---

## Description

> 1033.To Fill or Not to Fill (25)

> With highways available, driving a car from Hangzhou to any other city is easy. But since the tank capacity of a car is limited, we have to find gas stations on the way from time to time. Different gas station may give different price. You are asked to carefully design the cheapest route to go.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains 4 positive numbers: Cmax (<= 100), the maximum capacity of the tank; D (<=30000), the distance between Hangzhou and the destination city; Davg (<=20), the average distance per unit gas that the car can run; and N (<= 500), the total number of gas stations. Then N lines follow, each contains a pair of non-negative numbers: Pi, the unit gas price, and Di (<=D), the distance between this station and Hangzhou, for i=1,...N. All the numbers in a line are separated by a space.

> Output Specification:

> For each test case, print the cheapest price in a line, accurate up to 2 decimal places. It is assumed that the tank is empty at the beginning. If it is impossible to reach the destination, print "The maximum travel distance = X" where X is the maximum possible distance the car can run, accurate up to 2 decimal places.

> Sample Input 1:
50 1300 12 8
6.00 1250
7.00 600
7.00 150
7.10 0
7.20 200
7.50 400
7.30 1000
6.85 300
Sample Output 1:
749.17

> Sample Input 2:
50 1300 12 2
7.10 0
7.00 600
Sample Output 2:
The maximum travel distance = 1200.00

贪心算法： 遇到的最近的比当前加油站便宜的加油站，则加上足够到此车站的油；没有比当前车站便宜的，在当前车站加满油，到次便宜的车站；否则加满油，输出最远距离。

**注意可能始发点没有加油站QAQ**

## Code

```
#include<cstdio>  
#include<string>  
#include<cstring>  
#include<vector>  
#include<iostream>  
#include<queue>  
#include<algorithm>  
using namespace std;
typedef long long LL;
const int maxN = 5e2 + 10;

int m, d, p, s;
// the maximum capacity of the tank
// the distance between Hangzhou and the destination city
// the average distance per unit gas that the car can run   21
// the total number of gas stations

struct sta
{
	float x;
	float y;
	sta(float x = 0, float y = 0) :x(x), y(y){}
}stat[maxN];

float price;
float dis = 0;

float x;float y;

bool gre()
{
	int z = 0;
	float h = 0;
	while (dis != d)
	{
		int flag = -1;
		int maxs = m * p + dis;
		if (stat[0].y != 0)
			return 0;
		for (int i = z + 1; i <= s; i++)
		{
			if (stat[i].x < stat[z].x && stat[i].y <= maxs)
			{
				flag = i;
				price += (stat[flag].y - dis - h * p) / p * stat[z].x;
				h += (stat[flag].y - dis - h * p) / p;
				dis = stat[flag].y;
				h -= (stat[flag].y - stat[z].y) / p;
				z = flag;
				break;
			}
		}
		if (flag == -1)
		{
			if (stat[z + 1].y > maxs)
			{
				dis = maxs;
				return 0;
			}
			else
			{
				float tmp = stat[z + 1].x;
				flag = z + 1;
				for (int i = z + 1; i <= s; i++)
					if (stat[i].y <= maxs && stat[i].x < tmp)
					{
						flag = i;
						tmp = stat[i].x;
					}
				price += (m - h) * stat[z].x;
				h += m - h;
				dis = stat[flag].y;
				h -= (stat[flag].y - stat[z].y) / p;
				z = flag;
			}
		}
	}
	return 1;
}
int main()
{
	scanf("%d%d%d%d", &m, &d, &p, &s);
	for (int i = 0; i < s; i++)
	{
		scanf("%f", &x);
		scanf("%f", &y);
		stat[i] = sta(x, y);
	}
	stat[s] = sta(0, d);
	for (int i = 0; i < s - 1; i++)
		for (int j = i + 1; j < s; j++)
			if (stat[i].y > stat[j].y)
				swap(stat[i], stat[j]);
	gre() ? printf("%.2f\n", price) : printf("The maximum travel distance = %.2f\n", dis);
	return 0;
}
```