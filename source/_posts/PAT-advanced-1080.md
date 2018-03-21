---
title: PAT advanced 1080
tags:
  - PAT
categories: []
date: 2017-02-23 12:42:47
---

## Description

> 1080.Graduate Admission

> It is said that in 2013, there were about 100 graduate schools ready to proceed over 40,000 applications in Zhejiang Province. It would help a lot if you could write a program to automate the admission procedure.

> Each applicant will have to provide two grades: the national entrance exam grade GE, and the interview grade GI. The final grade of an applicant is (GE + GI) / 2. The admission rules are:

> The applicants are ranked according to their final grades, and will be admitted one by one from the top of the rank list.
If there is a tied final grade, the applicants will be ranked according to their national entrance exam grade GE. If still tied, their ranks must be the same.
Each applicant may have K choices and the admission will be done according to his/her choices: if according to the rank list, it is one's turn to be admitted; and if the quota of one's most preferred shcool is not exceeded, then one will be admitted to this school, or one's other choices will be considered one by one in order. If one gets rejected by all of preferred schools, then this unfortunate applicant will be rejected.
If there is a tied rank, and if the corresponding applicants are applying to the same school, then that school must admit all the applicants with the same rank, even if its quota will be exceeded.
Input Specification:

> Each input file contains one test case. Each case starts with a line containing three positive integers: N (<=40,000), the total number of applicants; M (<=100), the total number of graduate schools; and K (<=5), the number of choices an applicant may have.

> In the next line, separated by a space, there are M positive integers. The i-th integer is the quota of the i-th graduate school respectively.

> Then N lines follow, each contains 2+K integers separated by a space. The first 2 integers are the applicant's GE and GI, respectively. The next K integers represent the preferred schools. For the sake of simplicity, we assume that the schools are numbered from 0 to M-1, and the applicants are numbered from 0 to N-1.

> Output Specification:

> For each test case you should output the admission results for all the graduate schools. The results of each school must occupy a line, which contains the applicants' numbers that school admits. The numbers must be in increasing order and be separated by a space. There must be no extra space at the end of each line. If no applicant is admitted by a school, you must output an empty line correspondingly.

> Sample Input:
11 6 3
2 1 2 2 2 3
100 100 0 1 2
60 60 2 3 5
100 90 0 3 4
90 100 1 2 0
90 90 5 1 3
80 90 1 0 2
80 80 0 1 2
80 80 0 1 2
80 70 1 3 2
70 80 1 2 3
100 100 0 2 4

> Sample Output:
0 10
3
5 6 7
2 8

> 1 4

一个点超时了没过，以后再改

## Code

```
#include<cstdio>
#include<iostream>
#include<algorithm>
#include<vector>

using namespace std;
typedef long long LL;

const int maxN = 4e4 + 10;
const int maxM = 1e2 + 10;
const int maxK = 5;

int n, m, k;
struct stu
{
	int id;
	int e, i;
	int a[maxK];
	int g;
	int s;
}stu[maxN];
vector<int> ans[maxM];

bool cmp(int a, int b)
{
	return stu[a].id < stu[b].id;
}

int main()
{
	scanf("%d%d%d", &n, &m, &k);
	for (int i = 0; i < m; i++)
	{
		int x; scanf("%d", &x);
		ans[i].push_back(x);
	}
	for (int i = 0; i < n; i++)
	{
		stu[i].id = i;
		scanf("%d%d", &stu[i].e, &stu[i].i);
		stu[i].s = stu[i].e + stu[i].i;
		for (int j = 0; j < k; j++)
			scanf("%d", &stu[i].a[j]);
	}
	//sort
	for (int i = 0; i < n; i++)
		for (int j = i; j < n; j++)
			if (stu[i].s < stu[j].s)
				swap(stu[i], stu[j]);
			else if (stu[i].s == stu[j].s && stu[i].e < stu[j].e)
				swap(stu[i], stu[j]);
	stu[0].g = 0;
	for (int i = 1; i < n; i++)
		stu[i].g = (stu[i].s == stu[i - 1].s && stu[i].e == stu[i - 1].e) ? stu[i - 1].g : i;
	/***********************************************************/
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < k; j++)
		{
			if (ans[stu[i].a[j]][0] > 0)
			{
				ans[stu[i].a[j]][0]--;
				ans[stu[i].a[j]].push_back(i); // 压入录取学生排名
				break;
			}
			else if (ans[stu[i].a[j]][0] <= 0 && stu[ans[stu[i].a[j]].back()].g == stu[i].g)
			{
				ans[stu[i].a[j]][0]--;
				ans[stu[i].a[j]].push_back(i);
				break;
			}
		}
	}
	/*********************************************************/
	for (int i = 0; i < m; i++)
		sort(ans[i].begin() + 1, ans[i].end(), cmp);
	for (int i = 0; i < m; i++)
	{
		for (int j = 1; j < ans[i].size(); j++)
			(j == 1) ? printf("%d", stu[ans[i][j]].id) : printf(" %d", stu[ans[i][j]].id);
		printf("\n");
	}
	return 0;
}
```