---
title: Luogu P2503 [HAOI2006]均分数据 解题报告
categories: 解题报告
toc: true
date: 2019-09-24 16:18:29
tags:
	- 模拟退火
	- DP
top:
thumbnail: https://i.loli.net/2019/09/24/HlsYeFm9ZEiqN3h.jpg
---

卡时黑科技 $get√$

# 题目链接

[P2503 [HAOI2006]均分数据](https://www.luogu.org/problem/P2503)

# 题意简述

把 $n$ 个元素分成 $m$ 组，求每组元素和的最小均方差。

<!--more-->

# 解题思想

这种给出贡献的题直接 $SA$。

对于单组分配方案最小均方差的计算，考虑 $DP$，设 $dp[i][j]$ 表示前 $i$ 个元素分成 $j$ 组的最小均方差，枚举最后一组的断点转移就行了。

剩下的交给 $SA$。

-   黑科技：卡时

我们知道，$SA$ 的次数越多，答案精准的概率越大。所以而退火又是一个~~很高效~~的算法，所以我们考虑利用 “未被利用的时限”，而不是循环固定次数退火。

首先需要一个查询当前运行时间的函数 `Time()`。

```c++
double Time() {return (double)clock() / CLOCKS_PER_SEC;}
```

该函数返回当前程序运行了多少**秒**。

在实际退火时，我们仅需要对着时间卡退火次数就行了。

比如 `while(Time() < 0.8) SA();`。

注意给后面的语句留一定的时间。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <ctime>
#include <cmath>
#include <algorithm>
#include <cstdlib>
#include <cstring>

const double eps = 1e-3;
const double k = 0.99;
const int N = 21;

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

double a[N], s[N];
double f[N][N];

int n, m;
double ans, ave;

double dp() {
	for(register int i = 1; i <= n; i++) s[i] = s[i - 1] + a[i];
	memset(f, 127, sizeof f);
	f[0][0] = 0;
	for(register int i = 1; i <= n; i++)
		for(register int j = 1; j <= i; j++)
			for(register int k = 0; k <= i - 1; k++)
				f[i][j] = std::min(f[i][j], f[k][j - 1] + (s[i] - s[k] - ave) * (s[i] - s[k] - ave));
	ans = std::min(ans, f[n][m]);
	return f[n][m];
}

void SA() {
	double T = 10000;
	while(T > eps) {
		int x = 0, y = 0;
		while(x == y) {
			x = rand() % n + 1;
			y = rand() % n + 1;
		}
		std::swap(a[x], a[y]);
		double now = dp();
		double delta = now - ans;
		if(delta < 0) ans = now;
		else if(exp(-delta / T) * RAND_MAX < rand()) std::swap(a[x], a[y]);
		T *= k;
	}
}

double Time() {return (double)clock() / CLOCKS_PER_SEC;}

int main() {
	srand(time(NULL));
	n = read(), m = read();
	for(register int i = 1; i <= n; i++) scanf("%lf", a + i);
	for(register int i = 1; i <= n; i++) ave += a[i] / m;
	ans = dp();
	while(Time() < 0.75) SA();
	printf("%.2lf\n", sqrt(ans / m));
	return 0;
}
```

