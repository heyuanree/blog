---
title: PAT advanced 1100
tags: [PAT]
categories: []
date: 2017-02-01 22:10:39
---

## Description

>  1100.Mars Numbers(20)

> Zero on Earth is called "tret" on Mars.
The numbers 1 to 12 on Earch is called "jan, feb, mar, apr, may, jun, jly, aug, sep, oct, nov, dec" on Mars, respectively.
For the next higher digit, Mars people name the 12 numbers as "tam, hel, maa, huh, tou, kes, hei, elo, syy, lok, mer, jou", respectively.
For examples, the number 29 on Earth is called "hel mar" on Mars; and "elo nov" on Mars corresponds to 115 on Earth. In order to help communication between people from these two planets, you are supposed to write a program for mutual translation between Earth and Mars number systems.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains a positive integer N (< 100). Then N lines follow, each contains a number in [0, 169), given either in the form of an Earth number, or that of Mars.

> Output Specification:

> For each number, print in a line the corresponding number in the other language.

> Sample Input:
4
29
5
elo nov
tam

>Sample Output:
hel mar
may
115
13

学习了`sscanf(&src, format, &dest)`这个函数，从`&src`输入字符串到`&dest`。
而且还有个坑，`13`在这里不是`tam tret`而是`tam`

## Code

```
#include <cstdio>
#include <queue>
#include <vector>
#include <cstring>
#include <string>
#include <iostream>
#include<algorithm>

using namespace std;

string low[13] = { "tret", "jan", "feb", "mar", "apr", "may", "jun", "jly", "aug", "sep", "oct", "nov", "dec" };
string high[13] = { "" ,"tam", "hel", "maa", "huh", "tou", "kes", "hei", "elo", "syy", "lok", "mer", "jou" };

const int maxSize = 50;
char s[maxSize];
int n, x, ans;
string current;

int main()
{
	scanf("%d", &n);
	getchar();
	while (n--)
	{
		gets(s);
		if (s[0] <= '9' && s[0] >= '0')
		{
			sscanf(s, "%d", &x);
			if (!(x / 13)){ cout << low[x] << endl; continue; }
			if (x / 13 && x % 13){ cout << high[x / 13] << ' ' << low[x % 13] << endl; continue; }
			if (x / 13 && !(x % 13)){ cout << high[x / 13] << endl; continue; }
		}
		else
		{
			ans = 0;
			if (strlen(s) != 4)
			{
				for (int i = 0; !i || s[i - 1]; i += 4)
				{
					current = "";
					for (int j = i; j < i + 3; j++)
						current += s[j];
					for (int i = 0; i < 13; i++)
					{
						if (current == high[i]) ans += i * 13;
						if (current == low[i]) ans += i;
					}
				}
			}
			cout << ans << endl;
		}
	}
	return 0;
}
```
