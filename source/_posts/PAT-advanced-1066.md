---
title: PAT advanced 1066
tags:
  - PAT
categories: []
date: 2017-02-17 14:09:58
---

## Description

> An AVL tree is a self-balancing binary search tree. In an AVL tree, the heights of the two child subtrees of any node differ by at most one; if at any time they differ by more than one, rebalancing is done to restore this property. Figures 1-4 illustrate the rotation rules.

> ![1](PAT-advanced-1066/1.jpg) ![2](PAT-advanced-1066/2.jpg)

> ![3](PAT-advanced-1066/3.jpg) ![4](PAT-advanced-1066/4.jpg)

> Now given a sequence of insertions, you are supposed to tell the root of the resulting AVL tree.

> Input Specification:

> Each input file contains one test case. For each case, the first line contains a positive integer N (<=20) which is the total number of keys to be inserted. Then N distinct integer keys are given in the next line. All the numbers in a line are separated by a space.

> Output Specification:

> For each test case, print ythe root of the resulting AVL tree in one line.

> Sample Input 1:
5
88 70 61 96 120
Sample Output 1:
70

> Sample Input 2:
7
88 70 61 96 120 90 65
Sample Output 2:
88

照着数据结构的书打的，等会再做1123复习下。。
照着书还打错一遍Orz

## Code

```
#include<stack>
#include<cstdio>  
#include<string>  
#include<cstring>  
#include<vector>  
#include<iostream>  
#include<queue>  
#include<algorithm>  
using namespace std;
typedef long long LL;
struct avlndoe{
	int bf;
	int data;
	avlndoe *left;
	avlndoe *right;
	avlndoe(int d, avlndoe *l = NULL, avlndoe *r = NULL) : left(l), right(r), data(d), bf(0){}
}* root;

int n, data;

void rotateL(avlndoe * & ptr){
	avlndoe * subL = ptr;
	ptr = subL->right;
	subL->right = ptr->left;
	ptr->left = subL;
	ptr->bf = subL->bf = 0;
}

void rotateR(avlndoe * & ptr){
	avlndoe * subR = ptr;
	ptr = subR->left;
	subR->left = ptr->left;
	ptr->right = subR;
	ptr->bf = subR->bf = 0;
}

void rotateLR(avlndoe * & ptr){
	avlndoe * subR = ptr, *subL = subR->left;
	ptr = subL->right;
	subL->right = ptr->left;
	ptr->left = subL;
	ptr->bf <= 0 ? subL->bf = 0 : subL->bf = -1;
	subR->left = ptr->right;
	ptr->right = subR;
	ptr->bf == -1 ? subR->bf = 1 : subR->bf = 0;
	ptr->bf = 0;
}

void rotateRL(avlndoe * & ptr){
	avlndoe * subL = ptr, *subR = subL->right;
	ptr = subR->left;
	subR->left = ptr->right;
	ptr->right = subR;
	ptr->bf >= 0 ? subR->bf = 0 : subL->bf = 1;
	subL->right = ptr->left;
	ptr->left = subL;
	ptr->bf == 1 ? subL->bf = -1 : subL->bf = 0;
	ptr->bf = 0;
}

bool Insert(avlndoe * & ptr , int &el){
	avlndoe * pr = NULL, *p = ptr, *q; int d;
	stack<avlndoe *> st;
	while (p != NULL)
	{
		if (el == p->data) return false;
		pr = p; st.push(pr);
		(el < p->data) ? p = p->left : p = p->right;
	}
	p = new avlndoe(el);
	if (pr == NULL) { ptr = p; return true; }
	(el < pr->data) ? pr->left = p : pr->right = p;
	while (!st.empty())
	{
		pr = st.top();st.pop();
		(p == pr->left) ? pr->bf-- : pr->bf++;
		if (pr->bf == 0) break;
		if (pr->bf == 1 || pr->bf == -1)
			p = pr;
		else
		{
			d = (pr->bf < 0) ? -1 : 1;
			if (p->bf == d)
				(d == -1) ? rotateR(pr) : rotateL(pr);
			else
				(d == -1) ? rotateLR(pr) : rotateRL(pr);
			break;
		}
	}
	if (st.empty())ptr = pr;
	else
	{
		q = st.top();
		(q->data > pr->data) ? q->left = pr : q->right = pr;
	}
	return true;
}

int main()
{
	cin >> n;
	while (n--)
	{
		cin >> data;
		Insert(root, data);
	}
	cout << root->data << endl;
	return 0;
}
```