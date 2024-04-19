---
title: Luogu P2272 [ZJOI2007]最大半联通子图 解题报告
categories: 解题报告
date: 2019-08-01 10:53:03
tags:
	- 图论
	- DP
	- Tarjan
top:
---

题面成那样了都没人管....也是服

~~NOI Linux 真好用~~

# 题目链接

[P2272 [ZJOI2007]最大半连通子图](https://www.luogu.org/problem/P2272)

# 题意简述

最大半联通子图计数。

<!--more-->

# 解题思想

联通子图缩点后DP最长链，然后计数的话在SCC上DP的时候记录一下相等的情况就行了。

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
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        putchar('\n');
    }
}

using namespace FastIO;

const int N = 1e5 + 6;
const int M = 2e6 + 6;

template <typename T>
class stack {
    private:
    int stop;
    T s[N];
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

stack <int> s;

int n, m, mod;
int head[N], nxt[M], ver[M], cnt;
int scc[N], dfn[N], low[N], num[N], idx, tot;
bool v[N];
int Head[N], Nxt[M], Ver[M], pre[N], Cnt;
int in[N], dp[N], ans[N];

inline int min(int a, int b) {return a < b ? a : b;}
inline int max(int a, int b) {return a > b ? a : b;}

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
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
        else if(v[y]) low[x] = min(low[x], dfn[y]);
    }
    if(dfn[x] == low[x]) {
        tot++;
        while(!s.empty()) {
            int y = s.top();
            s.pop();
            v[y] = 0;
            scc[y] = tot;
            num[tot]++;
            if(x == y) break;
        }
    }
}

inline void Add(int x, int y) {
    Ver[++Cnt] = y, Nxt[Cnt] = Head[x], Head[x] = Cnt;
    in[y]++;
}

inline void rebuild() {
    for(register int x = 1; x <= n; x++) {
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i];
            if(scc[x] != scc[y]) Add(scc[x], scc[y]); 
        }
    }
}

inline void TopSort() {
    queue <int> q;
    for(register int i = 1; i <= tot; i++) {
        if(!in[i]) q.push(i);
        dp[i] = num[i], ans[i] = 1;
    }
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        for(register int i = Head[x]; i; i = Nxt[i]) {
            int y = Ver[i];
            in[y]--;
            if(!in[y]) q.push(y);
            if(pre[y] == x) continue;
            if(dp[y] < dp[x] + num[y]) {
                dp[y] = dp[x] + num[y];
                ans[y] = ans[x];
            }
            else if(dp[y] == dp[x] + num[y]) {
                ans[y] = (ans[y] + ans[x]) % mod;
            }
            pre[y] = x;
        }
    }
}

int main() {
    n = read(), m = read(), mod = read();
    for(register int i = 1; i <= m; i++) add(read(), read());
    for(register int i = 1; i <= n; i++) if(!dfn[i]) tarjan(i);
    rebuild();
    TopSort();
    int count = 0, maxx = 0;
    for(register int i = 1; i <= tot; i++) {
        if(dp[i] > maxx) maxx = dp[i], count = ans[i];
        else if(dp[i] == maxx) count = (count + ans[i]) % mod;
    }
    write(maxx);
    write(count);
    return 0;
}
```

