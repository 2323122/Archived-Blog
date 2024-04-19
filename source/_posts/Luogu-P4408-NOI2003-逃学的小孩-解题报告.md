---
title: Luogu P4408 [NOI2003]逃学的小孩 解题报告
categories: 解题报告
date: 2019-06-05 20:08:40
tags:
  - 树上问题
  - 树的直径
  - 贪心
top:
---

# 题目链接

[P4408/NOI2003 逃学的小孩](https://www.luogu.org/problemnew/show/P4408)

# 题意简述

给你一棵树，求树上的三个点$A,B,C$，满足$AB + BC$最长，且$BC<AC$

<!--more-->

# 解题思想

其实三个点位置的**顺序**不重要，我们可以两两交换得到一个**固定的顺序**，为了方便我就使用了$AB + BC$。

首先，图中**两点之间有且仅有一条简单路径**，说明这张图是**一棵树**。

要求三个点间的**最大距离**的话，我们不妨钦定有两个点间的路径长度为**树的直径**。可以通过**微扰**证明无法构造更优的方案。

对于剩下的点，我们在计算树的直径时顺便计算直径的顶点到树上每一个点的距离，然后**枚举图中的每一个点**，考虑这个点到直径**两端**的距离，用两者的**较小值**更新答案。

注意：要开`long long`

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#define int long long

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(int x) {
        if(x < 0) {putchar('-'), x = -x;}
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

const int N = 200020;
const int M = N << 2;

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

template <typename T> inline T max(T a, T b) {return a > b ? a : b;}
template <typename T> inline T min(T a, T b) {return a < b ? a : b;}

int head[N], nxt[M], ver[M], edge[M];
int dl[N], dr[N];
bool v[N];
int n, m, cnt, l, r, ans;

inline void add(int x, int y, int z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
}

signed main() {
    n = read(), m = read();
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read(), z = read();
        add(x, y, z);
        add(y, x, z);
    }
    queue <int> q;
    q.push(1);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 1;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i];
            if(v[y]) continue;
            dl[y] = dl[x] + z;
            q.push(y);
        }
    }
    for(register int i = 1; i <= n; i++) if(dl[i] > dl[l]) l = i;
    for(register int i = 1; i <= n; i++) v[i] = 0;
    dl[l] = 0;
    q.push(l);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 1;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i];
            if(v[y]) continue;
            dl[y] = dl[x] + z;
            q.push(y);
        }
    }
    for(register int i = 1; i <= n; i++) if(dl[i] > dl[r]) r = i;
    for(register int i = 1; i <= n; i++) v[i] = 0;
    q.push(r);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 1;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i];
            if(v[y]) continue;
            dr[y] = dr[x] + z;
            q.push(y);
        }
    }
    for(register int i = 1; i <= n; i++) {
        ans = max(ans, min(dl[i], dr[i]));
    }
    ans += dl[r];
    write(ans);
    return 0;
}
```

