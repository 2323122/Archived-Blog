---
title: Luogu P2746 [USACO5.3]校园网Network of Schools 解题报告
categories: 解题报告
date: 2019-04-30 09:57:54
tags: 
	- 图论
	- Tarjan
	- 强连通分量
top:
---

`for(register int j = 1; j <= n; i++)`

Farmer John太屑了

# 题目链接

[P2746 [USACO5.3]校园网Network of Schools](<https://www.luogu.org/problemnew/show/P2746>)

# 题意简述

给你一张图，~~你得看出来题目让你~~找出**入度为0**的强连通分量的个数以及它与**出度为0**的强连通分量的个数较大者

<!--more-->

# 解题思想

先来解读一下为什么要向题意简述那样做：

## 第一问

要想分发给更多的学校，必须从入度为0的点开始分发。

不妨**考虑一个入度不为0的点**：
对于一个**入度不为0**的点，我们总能从**可以抵达它的点**开始分发，这样接收数量显然更优。

## 第二问

要使原图**强连通**，则构造后的图**不存在**入度**与**出度为0的点。我们仅需求出两者中的**数量较大**者，

可以通过**较小者**向其连**较大数量的边**使原图不存在入度与出度为0的点，此时原图强连通。

具体实现就是缩点后统计一下每个节点的**度**就行了。

# 参考代码

```c++
#include <cstdio>
#include <cctype>

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0'; ch = getchar();}
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

const int N = 110;
const int M = N * N;

using namespace FastIO;

template <typename T> T max(T a, T b) {return a > b ? a : b;}
template <typename T> T min(T a, T b) {return a < b ? a : b;}

template <typename T>
class stack {
    private:
    int stop;
    T s[N << 1];
    public:
    stack() {stop = 0;}
    inline bool empty() {return !stop;}
    inline void push(T x) {s[++stop] = x;}
    inline void pop() {if(!empty()) stop--;}
    inline T top() {return s[stop];}
};

stack <int> s;

int head[N], ver[M], nxt[M], scc[N], dfn[N], low[N];
int in[N], out[N];
int idx, cnt, ans1, ans2, n, tot;
bool v[N];

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
}

void tarjan(int x) {
    dfn[x] = low[x] = ++idx;
    v[x] = 1;
    s.push(x);
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
        while(s.top()) {
            int y = s.top();
            s.pop();
            v[y] = 0;
            scc[y] = tot;
            if(x == y) break;
        }
    }
}

int main() {
    n = read();
    for(register int i = 1; i <= n; i++) {
        int x = read();
        while(x != 0) add(i, x), x = read();
    }
    for(register int i = 1; i <= n; i++) {
        if(!dfn[i]) tarjan(i);
    }
    for(register int i = n; i >= 1; i--) {
        for(register int j = head[i]; j; j = nxt[j]) {
            if(scc[i] != scc[ver[j]]) {
                in[scc[ver[j]]]++;
                out[scc[i]]++;
            }
        }
    }
    for(register int i = 1; i <= tot; i++) {
        if(in[i] == 0) ans1++;
        if(out[i] == 0) ans2++;
    }
    if(tot == 1) {
        puts("1");
        puts("0");
        return 0;
    }
    write(ans1);
    write(max(ans1, ans2));
    return 0;
}
```

