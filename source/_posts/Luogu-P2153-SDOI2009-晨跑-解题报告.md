---
title: Luogu P2153 [SDOI2009]晨跑 解题报告
categories: 解题报告
date: 2019-04-30 07:47:23
tags:
	- 图论
	- 网络流
	- 最小费用最大流
	- 建图
top:
---

老是这种套路就没意思了。

# 题目链接

[P2153 [SDOI2009]晨跑](<https://www.luogu.org/problemnew/show/P2153>)

# 题意简述

求给定网络的最大流与最小费用，其中每个点仅允许经过一次。

<!--more-->

# 解题思想

看到限制，考虑拆点。

不妨将每个点拆成$i$和$i + n$两个点，在中间连接一条**流量为1**的边，并且点$i$出度为1，点$i + n$入度为1.

这样可以限制点的经过次数。

然后在新图中以$1 + n$为原点，$n$为汇点跑**EK+SPFA**就行了。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cstring>

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) r = r * 10 + ch - '0' , ch = getchar();
        return r * w;
    }
    void _write(int x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        putchar(' ');
    }
}

using namespace FastIO;

const int N = 10006;
const int M = 2000006;
const int INF = 0x3f3f3f3f;

template <typename T> T min(T a, T b) {return a < b ? a : b;}

template <typename T>
class queue {
    private:
    int head, tail;
    T q[N << 1];
    public:
    queue() {head = tail = 0;}
    inline bool empty() {return head >= tail;}
    inline void push(T x) {q[++tail] = x;}
    inline void pop() {if(!empty()) head++;}
    inline T front() {return q[head + 1];}
};

int head[N], ver[M], edge[M], cost[M], nxt[M], pre[N], incf[N], d[N];
int cnt = 1, n, m, s, t, maxflow, mincost;
bool v[N];

inline void add(int x, int y, int z, int w) {
    ver[++cnt] = y, edge[cnt] = z, cost[cnt] =  w, nxt[cnt] = head[x], head[x] = cnt;
    ver[++cnt] = x, edge[cnt] = 0, cost[cnt] = -w, nxt[cnt] = head[y], head[y] = cnt;
}

inline bool spfa() {
    for(register int i = 1; i <= n << 1; i++) v[i] = 0, d[i] = INF;
    queue <int> q;
    q.push(s);
    d[s] = 0;
    v[s] = 1;
    incf[s] = INF;
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 0;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i], w = cost[i];
            if(!z) continue;
            if(d[y] > d[x] + w) {
                d[y] = d[x] + w;
                incf[y] = min(incf[x], z);
                pre[y] = i;
                if(!v[y]) {
                    v[y] = 1;
                    q.push(y);
                }
            }
        }
    }
    return d[t] != INF;
}

inline void upd() {
    maxflow += incf[t];
    mincost += incf[t] * d[t];
    int x = t;
    while(x != s) {
        int i = pre[x];
        edge[i] -= incf[t];
        edge[i ^ 1] += incf[t];
        x = ver[i ^ 1];
    }
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read(), z = read();
        add(x + n, y, 1, z);
    }
    for(register int i = 1; i <= n; i++) {
        add(i, i + n, 1, 0);
    }
    s = 1 + n, t = n;
    while(spfa()) upd();
    write(maxflow);
    write(mincost);
    return 0;
}
```

~~为什么我的数组总是开小啊QAQ~~

