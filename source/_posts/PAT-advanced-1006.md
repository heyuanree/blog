---
title: PAT advanced 1006
tags:
  - PAT
categories: []
date: 2017-02-09 00:10:16
---

## Description

> 1006.Sign In and Sign Out (25)

> Input Specification:

> Each input file contains one test case. Each case contains the records for one day. The case starts with a positive integer M, which is the total number of records, followed by M lines, each in the format:

> ID_number Sign_in_time Sign_out_time
where times are given in the format HH:MM:SS, and ID number is a string with no more than 15 characters.

> Output Specification:

> For each test case, output in one line the ID numbers of the persons who have unlocked and locked the door on that day. The two ID numbers must be separated by one space.

> Note: It is guaranteed that the records are consistent. That is, the sign in time must be earlier than the sign out time for each person, and there are no two persons sign in or out at the same moment.

>Sample Input:
3
CS301111 15:30:28 17:00:10
SC3021234 08:00:00 11:25:25
CS301133 21:45:00 21:58:40

>Sample Output:
SC3021234 CS301133

被自己蠢到了。开锁和上锁的输出顺序弄错了，瞎忙活了一个小时。

`cin`遇到`Space`,`Tab`,`Enter`会结束读取。

`||`优先级比`?`，长逻辑不确定记得加`()`

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

int m;
string first;
string last;
string id;

struct id
{
	int h, m, s;
}maxi, mini, tmp;

int main()
{
	scanf("%d", &m);
	for (int i = 0; i < m;i++)
	{
		cin >> id;
		scanf("%d:%d:%d", &tmp.h, &tmp.m, &tmp.s);
		if (!i || (tmp.h == mini.h ? tmp.m == mini.m ? tmp.s < mini.s : tmp.m < mini.m : tmp.h < mini.h))
		{
			mini = tmp; first = id;
		}
		scanf("%d:%d:%d", &tmp.h, &tmp.m, &tmp.s);
		if (!i || (tmp.h == maxi.h ? tmp.m == maxi.m ? tmp.s > maxi.s : tmp.m > maxi.m : tmp.h > maxi.h))
		{
			maxi = tmp; last = id;
		}
	}
	cout << first << " " << last << endl;
	return 0;
}
```