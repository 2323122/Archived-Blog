---
title: Luogu P1337 [JSOI2004]平衡点 解题报告
categories: 解题报告
toc: true
date: 2019-09-24 08:28:34
tags:
	- 模拟退火
top:
thumbnail: https://i.loli.net/2019/09/24/xpsDWnvRgy5JCte.jpg
---

物理⑧太好，不会正解，写了模拟退火 ![](https://i.loli.net/2019/09/24/wIYgVPRM8aXnrZ4.gif)。

# 题目链接

[P1337 [JSOI2004]平衡点 / 吊打XXX](https://www.luogu.org/problem/P1337)

# 题意简述

并不会概括..

有n个重物，每个重物系在一条足够长的绳子上。每条绳子自上而下穿过桌面上的洞，然后系在一起。图中X处就是公共的绳结。假设绳子是完全弹性的（不会造成能量损失），桌子足够高（因而重物不会垂到地上），且忽略所有的摩擦。

问绳结X最终平衡于何处。

注意：桌面上的洞都比绳结X小得多，所以即使某个重物特别重，绳结X也不可能穿过桌面上的洞掉下来，最多是卡在某个洞口处。

<!--more-->

# 解题思想

考虑将绳结系在点 $A(x, y)$ 时的 “势能”： $E = \sum\limits_{i = 1} ^ n \mathrm {dis}(A, A_i)\times m_i$ 。

其中 $A_i$ 指重物 $i$ 所在点，$m_i$ 指重物 $i$ 的重量。

有了这个东西，我们就可以很好地决定一个解的优劣程度。显然，当一个解的 $E$ 越小，答案就越优。

剩下的按退火来就行了。

rp太好了没调参就过了qwq。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cstdlib>
#include <cmath>
#include <ctime>

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 1010;
const double eps = 1e-15;
const double k = 0.999;

struct data {
	double x, y, m;
} a[N];

int n;
double ansx, ansy;

double dis(double x1, double y1, double x2, double y2) {
	return sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2));
}

double res(double x, double y) {
	double ans = 0;
	for(register int i = 1; i <= n; i++) ans += dis(x, y, a[i].x, a[i].y) * a[i].m;
	return ans;
}

double rdm() {
	return (rand() * 2 - RAND_MAX);
}

void SA() {
	double T = 200;
	while(T > eps) {
		double nowx = ansx + rdm() * T;
		double nowy = ansy + rdm() * T;
		double delta = res(nowx, nowy) - res(ansx, ansy);
		if(delta < 0) ansx = nowx, ansy = nowy;
		else if(exp(-delta / T) * RAND_MAX > rand()) ansx = nowx, ansy = nowy;
		T *= k;
	}
}

int main() {
	srand(time(NULL));
	n = read();
	for(register int i = 1; i <= n; i++) {
		scanf("%lf%lf%lf", &a[i].x, &a[i].y, &a[i].m);
		ansx += a[i].x, ansy += a[i].y;
	}
	ansx /= n, ansy /= n;
	SA();
	printf("%.3lf %.3lf\n", ansx, ansy);
	return 0;
}

```

