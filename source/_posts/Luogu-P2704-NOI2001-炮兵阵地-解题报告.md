---
title: Luogu P2704 [NOI2001]炮兵阵地 解题报告
categories: 解题报告
toc: true
date: 2019-09-05 09:58:34
tags:
	- DP
	- 状压DP
	- 字符串处理
top:
thumbnail: https://i.loli.net/2019/09/05/CGDkBNfMF69gAUS.jpg
---

学不来 `getchar()`

# 题目链接

[P2704 [NOI2001]炮兵阵地](https://www.luogu.org/problem/P2704)

# 题意简述

给你一个棋盘，让你放置一些棋子，攻击范围是上下左右 $2$ 格。有一些格不允许放置棋子。求最多能放置多少棋子互不攻击。

<!--more-->

# 解题思想

攻击范围是 $2$ 格，所以我们考虑状压两行。

然后以行为阶段，状态为两行，转移一行，更新最大值就行了。

很多人的状态棋子数统计是 $O(2^m\log m)$ 的，其实可以 $\mathrm{dfs}$ 做到 $O(2 ^ m)$。

具体实现烦请看一下代码。

# 参考代码

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <cctype>

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

void print2(int x, int len) {
	if(len) {
		print2(x >> 1, len - 1);
		printf("%d", x & 1);
	}
}

const int N = 11;

char ch;
int n, m, ans = 0;
int cant[101], dp[1 << N][1 << N][3], s[1 << N];

void dfs(int sit, int sum, int now) {
	if(now >= m) {
		s[sit] = sum;
		return;
	}
	dfs(sit + (1 << now), sum + 1, now + 1);
	dfs(sit, sum, now + 1);
}

inline int max(int a, int b) {return a > b ? a : b;}

int main() {
	n = read(), m = read();
	dfs(0, 0, 0);
	for(register int i = 0; i < n; i++) {
	    for(register int j = 0; j < m; j++) {
			std::cin >> ch;
	        cant[i] = (cant[i] << 1) + (ch == 'H');
	    }
	}
	for(register int i = 0; i < (1 << m); i++) 
		if(!((i & cant[0]) || (i & (i << 1)) || (i & (i << 2))))
			dp[0][i][0] = s[i];
	for(register int i = 0; i < (1 << m); i++) {
		for(register int j = 0; j < (1 << m); j++) {
			if(!((i & j) || (i & cant[0]) || (i & (i << 1)) || (i & (i << 2)) || (j & cant[1]) || (j & (j << 1)) || (j & (j << 2)))) {
				dp[i][j][1] = s[i] + s[j];
			}
		}
	}
	for(register int i = 2; i < n; i++) {
		for(register int j = 0; j < (1 << m); j++) {
			if((j & (cant[i - 1])) || (j & (j << 1)) || (j & (j << 2))) continue;
			for(register int k = 0; k < (1 << m); k++) {
				if((k & cant[i]) || (j & k) || (k & (k << 1)) || (k & (k << 2))) continue;
				for(register int S = 0; S < (1 << m); S++) {
					if((S & j) || (S & k) || (S & cant[i - 2]) || (S & (S << 1)) || (S & (S << 2))) continue;
					dp[j][k][i % 3] = max(dp[j][k][i % 3], dp[S][j][(i - 1) % 3] + s[k]);
				}
			}
		}
	}
	for(register int i = 0; i < (1 << m); i++) {
		for(register int j = 0; j < (1 << m); j++) {
			ans = max(ans, dp[i][j][(n - 1) % 3]);
		}
	}
	printf("%d\n", ans);
	return 0;
}
```

