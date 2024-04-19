---
title: CF1207E XOR Guessing 解题报告
categories: 解题报告
toc: true
date: 2019-08-28 08:44:58
tags:
top:
thumbnail: https://i.loli.net/2019/08/28/FONmI38P5w7obpB.jpg
---

交互太好van了，~~可惜辣鸡样例欺诈~~

# 题目链接

[(Luogu)CF1207E XOR Guessing](https://www.luogu.org/problem/CF1207E)

# 题意简述

**这是一道交互题**

询问两次，每次询问100个$[0, 2^{14} - 1]$的数，输出答案与其中一个数的异或值。

求出答案

<!--more-->

# 解题思想

考虑异或的性质，本质上是不进位加法，所以每一位仅对当前位有影响。于是我们可以七位七位地确定这个数，先询问一次，前七位都是1，再询问一次，后七位都是1，这样就可以唯一确定这个数了。

交互题一定要记得`fflush(stdout);`！！！

# 参考代码

```c++
#include <cstdio>
#include <cctype>

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 110;

int a[N], b[N], ans, tmp;

int main() {
	for(register int i = 1; i <= 100; i++) {
		a[i] = i;
		b[i] = i << 7;
	}
	printf("?");
	for(register int i = 1; i <= 100; i++) {
		printf(" %d", a[i]);
	}
	puts("");
	fflush(stdout);
	ans = read();
	printf("?");
	for(register int i = 1; i <= 100; i++) {
		printf(" %d", b[i]);
	}
	puts("");
	fflush(stdout);
	ans = (ans ^ (tmp = read())) >> 7;
	printf("! %d\n", (ans << 7) ^ tmp);
	return 0;
}
```

