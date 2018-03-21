---
title: PAT advanced 1112
tags:
  - PAT
categories: []
date: 2017-02-19 15:49:26
---

## Descritpion

> 1112.Stucked Keyboard

> On a broken keyboard, some of the keys are always stucked. So when you type some sentences, the characters corresponding to those keys will appear repeatedly on screen for k times.

> Now given a resulting string on screen, you are supposed to list all the possible stucked keys, and the original string.

> Notice that there might be some characters that are typed repeatedly. The stucked key will always repeat output for a fixed k times whenever it is pressed. For example, when k=3, from the string "thiiis iiisss a teeeeeest" we know that the keys "i" and "e" might be stucked, but "s" is not even though it appears repeatedly sometimes. The original string could be "this isss a teest".

> Input Specification:

> Each input file contains one test case. For each case, the 1st line gives a positive integer k ( 1<k<=100 ) which is the output repeating times of a stucked key. The 2nd line contains the resulting string on screen, which consists of no more than 1000 characters from {a-z}, {0-9} and "\_". It is guaranteed that the string is non-empty.

> Output Specification:

> For each test case, print in one line the possible stucked keys, in the order of being detected. Make sure that each key is printed once only. Then in the next line print the original string. It is guaranteed that there is at least one stucked key.

> Sample Input:
3
caseee1\_\_thiiis_iiisss\_a\_teeeeeest

> Sample Output:
ei
case1\_\_this\_isss\_a\_teest

注意：必须全部输入才能判断坏键，如`sss_s`已判断为坏键为错误。一旦确定为好键则不会修改为坏键。

## Code

```
#include<cstdio>
#include<iostream>
#include<map>
#include<string>

using namespace std;
typedef long long LL;

int k;
int cou = 0;
int sta = 0;
char pre;
string mes;
string ans = "";
string wrong = "";
map<char, int> m;

int main()
{
	scanf("%d", &k);
	cin >> mes;
	mes += '#';
	m.insert(pair<char, int>('_', -1));
	for (char i = 'a'; i <= 'z'; i++)
		m.insert(pair<char, int>(i, -1));
	for (char i = '0'; i <= '9'; i++)
		m.insert(pair<char, int>(i, -1));
	pre = mes[0];
	for (int i = 0; i < mes.length(); i++)
	{
		if (pre == mes[i])
		{
			sta = 0;
			cou++;
		}
		else
		{
			sta = 1;
			if (cou % k == 0 && m[pre] == -1)
			{
				m[pre] = 0;
				wrong += pre;
			}
			if (cou % k != 0)
			{
				m[pre] = 1;
			}
			pre = mes[i];
			cou = 1;
			continue;
		}
	}
	for (int i = 0; i < wrong.size(); i++)
		if (m[wrong[i]] == 0)
			cout << wrong[i];
	cout << endl;
	for (int i = 0; i < mes.size() - 1;)
		if (m[mes[i]])
		{ cout << mes[i]; i++; }
		else
		{ cout << mes[i]; i += k; }
	return 0;
}
```