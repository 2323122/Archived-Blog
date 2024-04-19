---
title: Luogu P3119 草鉴定 解题报告
categories: 解题报告
date: 2019-05-04 11:08:11
tags:
	- 图论
	- 强连通分量
	- Tarjan
top:
---

~~草鉴定太艹了。~~

# 题目链接

[**P3119** 草鉴定Grass Cownoisseur](https://www.luogu.org/problemnew/show/P3119) 

# 题意简述

给你一张图，允许加一条**已知边**的**反向边**，求从**点1**出发**并回到点1**最多经过多少个点。

<!--more-->

# 解题思想

首先在原图上**缩点**，缩成**DAG**，点权**相加**。

然后在DAG上**拓扑排序**或者**SPFA**求出**以1为源**的**最长路**。

然后建**反图**（可以在缩点后就搞）。

再在**反图**上求出**以`scc[1]为源`**的**最长路**，即**原DAG**上**以`scc[1]`为汇**的**最长路**。

最后，枚举每一个**原图**上的顶点`i`，若`scc[i]`可达，枚举每一条**反向边**，同时更新答案。

注意**反向边未使用**的情况，初始化`ans = num[scc[1]]`就行了。

# 参考代码

```c++
#include <cstdio>
#include <cctype>

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(int x) {
        if(x < 0) putchar('-');
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

const int N = 1e5 + 6;
const int M = N << 1;
const int INF = 0xcfcfcfcf;

template <typename T> T max(T a, T b) {return a > b ? a : b;}
template <typename T> T min(T a, T b) {return a < b ? a : b;}

template <typename T>
class stack {
    private:
    int stop;
    T s[N << 1];
    public:
    stack() {stop = 0;}
    inline bool empty() {return stop == 0;}
    inline void push(T x) {s[++stop] = x;}
    inline void pop() {if(!empty()) stop--;}
    inline T top() {return s[stop];}
};

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

int n, m, st, ans;
int head[N], ver[M], nxt[M], cnt;
int dfn[N], low[N], scc[N], num[N], tot, idx;
int head1[N], ver1[M], nxt1[M], cnt1;
int head2[N], ver2[M], nxt2[M], cnt2;
int d1[N], d2[N];
bool v[N];
stack <int> s;

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
}

inline void add1(int x, int y) {
    ver1[++cnt1] = y, nxt1[cnt1] = head1[x], head1[x] = cnt1;
}

inline void add2(int x, int y) {
    ver2[++cnt2] = y, nxt2[cnt2] = head2[x], head2[x] = cnt2;
}

void tarjan(int x) {
    dfn[x] = low[x] = ++idx;
    s.push(x);
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
    if(low[x] == dfn[x]) {
        tot++;
        while(s.top()) {
            int y = s.top();
            s.pop();
            scc[y] = tot;
            num[tot]++;
            v[y] = 0;
            if(x == y) break;
        }
    }
}

inline void spfa1() {
    queue <int> q;
    v[st] = 1;
    d1[st] = num[st];
    q.push(st);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 0;
        for(register int i = head1[x]; i; i = nxt1[i]) {
            int y = ver1[i];
            if(d1[y] < d1[x] + num[y]) {
                d1[y] = d1[x] + num[y];
                if(!v[y]) {
                    v[y] = 1;
                    q.push(y);
                }
            }
        }
    }
}

inline void spfa2() {
    queue <int> q;
    v[st] = 1;
    d2[st] = num[st];
    q.push(st);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 0;
        for(register int i = head2[x]; i; i = nxt2[i]) {
            int y = ver2[i];
            if(d2[y] < d2[x] + num[y]) {
                d2[y] = d2[x] + num[y];
                if(!v[y]) {
                    v[y] = 1;
                    q.push(y);
                }
            }
        }
    }
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i <= m; i++) {
        add(read(), read());
    }
    for(register int i = 1; i <= n; i++) {
        if(!dfn[i]) tarjan(i);
    }
    st = scc[1];
    for(register int x = 1; x <= n; x++) {
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i];
            if(scc[x] != scc[y]) add1(scc[x], scc[y]), add2(scc[y], scc[x]);
        }
    }
    spfa1();
    spfa2();
    int ans = num[st];
    for(register int x = 1; x <= n; x++) {
        if(!v[scc[x]] && d1[scc[x]]) {
            v[scc[x]] = 1;
            for(register int i = head2[scc[x]]; i; i = nxt2[i]) {
                int y = ver2[i];
                if(!d2[y]) continue;
                ans = max(ans, d1[scc[x]] + d2[y] - num[st]);
            }
        }
    }
    write(ans);
    return 0;
}
```

