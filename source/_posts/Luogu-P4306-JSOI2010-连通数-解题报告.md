---
title: Luogu P4306 [JSOI2010]连通数 解题报告
categories: 解题报告
date: 2019-08-01 20:36:04
tags:
	- Tarjan
	- 强连通分量
	- Floyd
	- 传递闭包
	- 拓扑排序
	- 图论
top:
---

~~高级暴力~~

翻了一下题解好像没人解法跟我一样

# 题目链接

[P4306 [JSOI2010]连通数](https://www.luogu.org/problem/P4306)

# 题意简述

给你一张图，求它的连通数。

连通数：指图中可达顶点对个的个数。

<!--more-->

# 解题思想

如果$n$不大的话，我们可以跑传递闭包，然后统计答案，这个比较暴力，是$O(n^3)$的，显然过不了2000~~但是数据水好像能过~~

我们考虑将范围缩小：
不妨对原图缩点，这样的话$n$的范围**期望**大大减小。（可能被卡）

然后在缩点后的图上传递闭包，就不会超时了。

但是答案统计的方法和以前要有所区别：
如果直接对原图传递闭包，我们仅需$O(n^2)$遍历一遍然后`ans++`就行了。

对于这个缩完的DAG，我们考虑每两个连通块联通对答案的贡献：
根据乘法原理，两个连通块贡献的连通数与块大小有关，假定两个连通块分别为$A,B$，那么这个贡献就是$size_A\times size_B$，其中$size_i$表示连通块$i$包含的节点个数。

那么，推广到所有连通块上，总连通数即为：
$$\sum \limits_{i, j\in DAG} size_i \times size_j$$

需要注意的是，当原图每个点各为一个强连通分量时，该算法会上界会跑满。

# 参考代码

```c++
#include <cstdio>
#include <cctype>

namespace FastIO  {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(int x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

const int N = 2010;
const int M = N * N;

inline int min(int a, int b) {return a < b ? a : b;}
inline int max(int a, int b) {return a > b ? a : b;}

int n, m, ans;
int head[N], nxt[M], ver[M], cnt;
int scc[N], num[N], dfn[N], low[N], idx, tot;
bool v[N];
int sta[N], stop;
bool f[N][N];

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
}

void tarjan(int x) {
    dfn[x] = low[x] = ++idx;
    sta[++stop] = x;
    v[x] = 1;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(!dfn[y]) {
            tarjan(y);
            low[x] = min(low[x], low[y]);
        }
        else if(v[y]) {
            low[x] = min(low[x], dfn[y]);
        }
    }
    if(dfn[x] == low[x]) {
        tot++;
        while(stop) {
            int y = sta[stop--];
            scc[y] = tot;
            v[y] = 0;
            num[tot]++;
            if(x == y) break;
        }
    }
}

void rebuild() {
    for(register int x = 1; x <= n; x++) {
        for(register int i = head[x]; i; i = nxt[i]) {
            if(scc[x] != scc[ver[i]]) f[scc[x]][scc[ver[i]]] = 1;
        }
    }
}

char s[N][N];

int main() {
    n = read();
    for(register int i = 1; i <= n; i++) scanf("%s", s[i] + 1);
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= n; j++) {
            if(s[i][j] == '1') add(i, j);
        }
    }
    for(register int i = 1; i <= n; i++) if(!dfn[i]) tarjan(i);
    rebuild();
    for(register int i = 1; i <= tot; i++) {
        f[i][i] = 1;
        for(register int j = 1; j <= tot; j++) {
            for(register int k = 1; k <= tot; k++) {
                f[i][j] |= f[i][k] & f[k][j];
            }
        }
    }
    for(register int i = 1; i <= tot; i++) {
        for(register int j = 1; j <= tot; j++) {
            if(f[i][j]) ans += num[i] * num[j];
        }
    }
    if(tot == 1) ans = num[1] * num[1];
    write(ans);
    return 0;
}
```

