---
title: Luogu P4742 [Wind Festival]Running In The Sky 解题报告
categories: 解题报告
date: 2019-05-07 16:38:51
tags:
	- 图论
	- Tarjan
	- 强连通分量
top:
---

屑题卡我两天，这可能是我写过的最长的蓝题了。

# 题目链接

[P4742 [Wind Festival]Running In The Sky](<https://www.luogu.org/problemnew/show/P4742>)

# 题意简述

给你一张**有向图**，有点权，一个点的费用可被收取**当仅当第一次**经过该点，求出**最长路**和该路径上**点权最大值**。

<!--more-->

# 解题思想

缩点没什么好说的，但是在缩点时我们维护**每个强连通分量**的**两个**信息：

- 点权之和`info[0][i]`
- 点权最大值`info[1][i]`

缩完点后建立新图，就不用多说了。

然后在**新图**上**拓扑排序**的同时**DP**，维护到每个点的最长路，同时**有依赖性地**更新点权最大值。

这个**有依赖性**就是难点。

我们考虑**独立DP极大值**，可以很轻易地被Hack掉，考虑一条路径，上面有**一个**点**权值极大**，其余的点权值**全为0**，再构造另一条路径，每个点的权值**仅**比上条路径**权值极大**点**小1**，这样我们DP出的结果很明显不在**最长路**上。

那要怎么做呢？

考虑**仅在最长路径转移的时候**转移**极大值的状态**，即**单独考虑**两条路**长度相等**的情况。

那么**拓扑排序时**就有**两种情况**：

- 一是**最长路**更新，这时需要更新最大点权，与**新加入的点的权值**进行对比。
- 二是发现**等长路径**，这是也需要更新最大点权，**但是**与**新加入点的状态**进行对比。

很好想通，发现等长路径时，我们需要将**原状态**与**整条路径上的点权**进行对比，这就是新加入点的状态了。

# 参考代码

写的比较工程化，所以很长。

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
        putchar(' ');
    }
}

namespace Data {
    static const int N = 2e5 + 6;
    static const int M = 5e5 + 6;

    template <typename T> T max(T a, T b) {return a > b ? a : b;}
    template <typename T> T min(T a, T b) {return a < b ? a : b;}

    template <typename T>
    class queue {
        private:
        int head, tail;
        T q[N];
        public:
        queue() {head = tail = 0;}
        inline bool empty() {return head >= tail;}
        inline void push(T x) {q[++tail] = x;}
        inline void pop() {if(!empty()) head++;}
        inline T front() {return q[head + 1];}
    };

    template <typename T>
    class stack {
        private:
        int stop;
        T s[N];
        public:
        inline bool empty() {return stop == 0;}
        inline void push(T x) {s[++stop] = x;}
        inline void pop() {if(!empty()) stop--;}
        inline T top() {return s[stop];}
    };

    int n, m;
    int head[N], nxt[M], ver[M], k[N], cnt;

    int scc[N], low[N], dfn[N], info[2][N], dp[2][N], idx, tot;
    bool v[N];
    stack <int> s;

    int Head[N], Ver[M], Nxt[M], in[N], Cnt;
}

using namespace FastIO;
using namespace Data;

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
    if(low[x] == dfn[x]) {
        tot++;
        while(s.top()) {
            int y = s.top();
            s.pop();
            v[y] = 0;
            scc[y] = tot;
            info[0][tot] += k[y];
            info[1][tot] = max(info[1][tot], k[y]);
            if(x == y) break;
        }
    }
}

inline void Add(int x, int y) {
    Ver[++Cnt] = y, Nxt[Cnt] = Head[x], Head[x] = Cnt, in[y]++;
}

inline void rebuild() {
    for(register int x = 1; x <= n; x++) {
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i];
            if(scc[y] != scc[x]) Add(scc[x], scc[y]);
        }
    }
}

inline void TopSort() {
    queue <int> q;
    for(register int i = 1; i <= n; i++) {
        dp[0][i] = info[0][i];
        dp[1][i] = info[1][i];
        if(in[i] == 0) q.push(i);
    }
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        for(register int i = Head[x]; i; i = Nxt[i]) {
            int y = Ver[i];
            if(dp[0][y] < dp[0][x] + info[0][y]) {
                dp[0][y] = dp[0][x] + info[0][y];
                dp[1][y] = max(dp[1][x], info[1][y]);
            }
            else if(dp[0][y] == dp[0][x] + info[0][y]) {
                dp[1][y] = max(dp[1][y], dp[1][x]);
            }
            in[y]--;
            if(in[y] == 0) q.push(y);
        }
    }
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i <= n; i++) {
        k[i] = read();
    }
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read();
        add(x, y);
    }
    for(register int i = 1; i <= n; i++) {
        if(!dfn[i]) tarjan(i);
    }
    rebuild();
    TopSort();
    int las = 1;
    for(register int i = 2; i <= tot; i++) {
        if(dp[0][i] > dp[0][las] || (dp[0][i] == dp[0][las] && dp[1][i] >= dp[1][las])) las = i;
    }
    write(dp[0][las]);
    write(dp[1][las]);
    return 0;
}
```

