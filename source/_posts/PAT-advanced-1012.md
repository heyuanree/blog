---
title: PAT advanced 1012
tags:
  - PAT
categories: []
date: 2017-02-13 18:53:14
---

## Description

> 1012.The Best Rank (25)

> To evaluate the performance of our first year CS majored students, we consider their grades of three courses only: C - C Programming Language, M - Mathematics (Calculus or Linear Algebra), and E - English. At the mean time, we encourage students by emphasizing on their best ranks -- that is, among the four ranks with respect to the three courses and the average grade, we print the best rank for each student.

 > For example, The grades of C, M, E and A - Average of 4 students are given as the following:

> StudentID  C  M  E  A
310101     98 85 88 90
310102     70 95 88 84
310103     82 87 94 88
310104     91 91 91 91
Then the best ranks for all the students are No.1 since the 1st one has done the best in C Programming Language, while the 2nd one in Mathematics, the 3rd one in English, and the last one in average.

> Input

> Each input file contains one test case. Each case starts with a line containing 2 numbers N and M (<=2000), which are the total number of students, and the number of students who would check their ranks, respectively. Then N lines follow, each contains a student ID which is a string of 6 digits, followed by the three integer grades (in the range of [0, 100]) of that student in the order of C, M and E. Then there are M lines, each containing a student ID.

> Output

> For each of the M students, print in one line the best rank for him/her, and the symbol of the corresponding rank, separated by a space.

> The priorities of the ranking methods are ordered as A > C > M > E. Hence if there are two or more ways for a student to obtain the same best rank, output the one with the highest priority.

> If a student is not on the grading list, simply output "N/A".

> Sample Input
5 6
310101 98 85 88
310102 70 95 88
310103 82 87 94
310104 91 91 91
310105 85 90 90
310101
310102
310103
310104
310105
999999

> Sample Output
1 C
1 M
1 E
1 A
3 A
N/A

注意排名相同情况下，取相同的最高名次。

分数要`int(sum / 3 + 0.5)`，但其实不四舍五入也能过。。

## Code

```
#include <cstdio>
#include <cmath>
#include <queue>
#include <vector>
#include <cstring>
#include <string>
#include <iostream>
#include <algorithm>
#include <climits>

using namespace std;
typedef unsigned long long uLL;
const int maxN = 2e3 + 10;

struct S{
	string name;
	int a[4];
	int b[4];
	int r[4];
	int g;
}stu[maxN];

int n, m;
char sym[] = "ACME";
int tmp[maxN];

int main()
{
	cin >> n >> m;
	for (int i = 0; i < n; i++)
	{
		cin >> stu[i].name;
		float tmp = 0;
		for (int j = 1; j < 4; j++)
		{
			cin >> stu[i].a[j];
			tmp += stu[i].a[j];
			stu[i].b[j] = stu[i].a[j];
		}
		stu[i].a[0] = tmp / 3.0;
		stu[i].b[0] = tmp / 3.0;
	}
	for (int i = 0; i < 4; i++)
	{
		for (int j = 0; j < n; j++) tmp[j] = j;
		for (int p = 0; p < n - 1; p++)
			for (int q = p + 1; q < n; q++)
				if (stu[q].a[i] > stu[p].a[i])
				{
					swap(stu[q].a[i], stu[p].a[i]);
					swap(tmp[p], tmp[q]);
				}
		for (int j = 0; j < n; j++)
			stu[tmp[j]].r[i] = j;
		for (int j = 0; j < n - 1; j++)
		{
			if (stu[tmp[j]].b[i] == stu[tmp[j + 1]].b[i])
				stu[tmp[j + 1]].r[i] = stu[tmp[j]].r[i];
		}
	}
	for (int i = 0; i < n; i++)
	{
		int tmp = maxN;
		for (int j = 0; j < 4; j++)
			if (stu[i].r[j] < tmp)
			{
				tmp = stu[i].r[j];
				stu[i].g = j;
			}
	}
	//
	for (int i = 0; i < m; i++)
	{
		string in;
		cin >> in;
		int j = 0;
		for (j = 0; j < n; j++)
			if (stu[j].name == in)
			{
				cout << stu[j].r[stu[j].g] + 1 << ' ' << sym[stu[j].g] << endl;
				break;
			}
		if (j == n)
			cout << "N/A" << endl;
	}
	return 0;
}
```