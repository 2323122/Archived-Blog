---
title: Luogu P1896 [SCOI2005]互不侵犯 解题报告
categories: 解题报告
toc: true
date: 2019-09-04 16:43:05
tags:
	- DP
	- 状压DP
top:
thumbnail: https://i.loli.net/2019/09/06/BCsbRpJOUiSf3ta.png
---

状压DP入门题~~但是我早都入门了为啥还要写这个~~

# 题目链接

[P1896 [SCOI2005]互不侵犯](https://www.luogu.org/problem/P1896)

# 题意简述

在N×N的棋盘里面放K个国王，使他们互不攻击，共有多少种摆放方案。国王能攻击到它上下左右，以及左上左下右上右下八个方向上附近的各一个格子，共8个格子。

<!--more-->

# 解题思想

考虑状压每一行，然后以行为阶段DP。

预处理出每一个状态所需的国王数，然后转移的时候枚举两行的状态判一下就行了。

这里给出一个调试状压DP很好用的函数：

```c++
void print(ll x, ll base, ll len, char const *s = NULL) {
	int tmp[100] = {0}, tot = 0;
	while(x) {
		tmp[++tot] = x % base;
		x /= base;
	}
	for(register int i = len; i >= 1; i--) printf("%d", tmp[i]);
	if(s) printf("%s", s);
}
```

将一个数转为 `base` 进制然后输出，空位输出 $0$

便于打印状态。

当然，如果是二进制状压则有一个好写的多的函数：

```c++
void print_bin(int x, int len) {
    if(len) {
        print_bin(x >> 1, len - 1);
     	putchar('0' + x & 1);
    }
}
```



# 参考代码

```c++
#include <cstdio>
#include <cctype>

#define ll long long

namespace FastIO {
	inline ll read() {
		char ch = getchar(); ll r = 0, w = 1;
		while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
		while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
		return r * w;
	}
	void _write(ll x) {
		if(x < 0) putchar('-'), x = -x;
		if(x > 9) _write(x / 10);
		putchar(x % 10 + '0');
	}
	inline void write(ll x, char const *s = "\n") {
		_write(x);
		printf("%s", s);
	}
}

using namespace FastIO;

const int N = 10;

ll ans = 0;
ll n, m, cnt;
ll dp[N][1 << N][N * N];
ll sta[1 << N], num[1 << N];

void print(ll x, ll base, ll len, char const *s = NULL) {
	int tmp[100] = {0}, tot = 0;
	while(x) {
		tmp[++tot] = x % base;
		x /= base;
	}
	for(register int i = len; i >= 1; i--) printf("%d", tmp[i]);
	if(s) printf("%s", s);
}

void dfs(int sit, int sum, int now) {
	if(now >= n) {sta[++cnt] = sit, num[cnt] = sum; return;}
	dfs(sit, sum, now + 1);
	dfs(sit + (1 << now), sum + 1, now + 2);
}

int main() {
	n = read(), m = read();
	dfs(0, 0, 0);
	for(register int i = 1; i <= cnt; i++) {
		dp[1][sta[i]][num[i]] = 1;
	}
	#ifdef DEBUG
	for(register int i = 1; i <= cnt; i++) {
		printf("num[");
		print(sta[i], 2, n);
		printf("] = %d\n", num[i]);
	}
	#endif
	for(register int i = 2; i <= n; i++) {
		for(register int j = 1; j <= cnt; j++) {
			for(register int k = 1; k <= cnt; k++) {
//				print(sta[j], 2, n, "\n");
//				print(sta[k], 2, n, "\n");
				if((sta[j] & sta[k]) || ((sta[j] << 1) & sta[k]) || ((sta[k] << 1) & sta[j])) {
					continue;
				}
				for(register int s = m; s >= num[j]; s--) {
					dp[i][sta[j]][s] += dp[i - 1][sta[k]][s - num[j]];
				}
			}
		}
	}
	for(register int i = 1; i <= cnt; i++) ans += dp[n][sta[i]][m];
	write(ans);
	return 0;
}
```

