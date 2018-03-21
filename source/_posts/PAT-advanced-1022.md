---
title: PAT advanced 1022
tags:
  - PAT
categories: []
date: 2017-02-22 17:31:15
---

## Description

> 1022.Digital Library

> A Digital Library contains millions of books, stored according to their titles, authors, key words of their abstracts, publishers, and published years. Each book is assigned an unique 7-digit number as its ID. Given any query from a reader, you are supposed to output the resulting books, sorted in increasing order of their ID's.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains a positive integer N (<=10000) which is the total number of books. Then N blocks follow, each contains the information of a book in 6 lines:

> Line \#1: the 7-digit ID number;
Line \#2: the book title -- a string of no more than 80 characters;
Line \#3: the author -- a string of no more than 80 characters;
Line \#4: the key words -- each word is a string of no more than 10 characters without any white space, and the keywords are separated by exactly one space;
Line \#5: the publisher -- a string of no more than 80 characters;
Line \#6: the published year -- a 4-digit number which is in the range [1000, 3000].
It is assumed that each book belongs to one author only, and contains no more than 5 key words; there are no more than 1000 distinct key words in total; and there are no more than 1000 distinct publishers.

> After the book information, there is a line containing a positive integer M (<=1000) which is the number of user's search queries. Then M lines follow, each in one of the formats shown below:

> 1: a book title
2: name of an author
3: a key word
4: name of a publisher
5: a 4-digit number representing the year
Output Specification:

> For each query, first print the original query in a line, then output the resulting book ID's in increasing order, each occupying a line. If no book is found, print "Not Found" instead.

> Sample Input:
3
1111111
The Testing Book
Yue Chen
test code debug sort keywords
ZUCS Print
2011
3333333
Another Testing Book
Yue Chen
test code sort keywords
ZUCS Print2
2012
2222222
The Testing Book
CYLL
keywords debug book
ZUCS Print2
2011
6
1: The Testing Book
2: Yue Chen
3: keywords
4: ZUCS Print
5: 2011
3: blablabla

> Sample Output:
1: The Testing Book
1111111
2222222
2: Yue Chen
1111111
3333333
3: keywords
1111111
2222222
3333333
4: ZUCS Print
1111111
5: 2011
1111111
2222222
3: blablabla
Not Found

哇题目都没看清的我居然过了，是写了个更鲁棒的代码？（笑）
`map`+`set`的使用，第一遍超时，后来减少查找，增加映射过了- -

## Code

```
#include<cstdio>
#include<string>
#include<iostream>
#include<algorithm>
#include<map>
#include<set>

using namespace std;
typedef long long LL;

const int maxN = 1e4 + 10;

set<string> s;

int n, m;
string x, y, z;

map<string, set<string>> f;

void fin(string x)
{
	if (f.find(x) != f.end())
		for (set<string>::iterator it = f[x].begin(); it != f[x].end(); it++)
			cout << *it << endl;
	else
		cout << "Not Found" << endl;
}

int main()
{
	scanf("%d\n", &n);
	for (int i = 0; i < n; i++)
	{
		getline(cin, x);
		getline(cin, y);
		f[y].insert(x);
		getline(cin, y);
		f[y].insert(x);
		while (true)
		{
			cin >> y;
			f[y].insert(x);
			if (getchar() == '\n')
				break;
		}
		getline(cin, y);
		f[y].insert(x);
		getline(cin, y);
		f[y].insert(x);
	}
	scanf("%d\n", &m);
	for (int i = 0; i < m; i++)
	{
		getline(cin, y);
		cout << y << endl;
		z = y.substr(3, y.size());
		fin(z);
	}
	return 0;
}
```