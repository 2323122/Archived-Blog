---
title: Luogu P3629 [APIO2010]巡逻 解题报告
categories: 解题报告
date: 2019-06-06 09:26:49
tags:
  - 树上问题
  - 树的直径
  - 分类讨论
  - dfs
  - 树上dp
top:
---

# 题目链接

[P3629 [APIO2010]巡逻](https://www.luogu.org/problemnew/show/P3629)

# 题意简述

给你一张无向图，有$n$个点和$n - 1$条边，**无重边和自环**，边长都为$1$，给定源点$1$，新建$k$条道路，要求从源点出发，每条道路**必须**经过**至少一次**，且**新建的道路必须仅经过一次**，求可能的新建道路的方案，使**遍历路径最短**。

<!--more-->

# 解题思想

暴力就不讲了，应该都能想到。

我们直接考虑$100\%$的数据。

注意原题中的$1\le k \le 2$，我们考虑**分类讨论**。

## 先来考虑$k == 1$的情况。

在`k == 1`时，这道题和[草鉴定](https://www.luogu.org/problemnew/show/P3119)有点像，但是读完题目后我们会发现，两道题的方向是相反的，要求也不同。

那要怎么做呢？

先来看题目：

>  有$n$个点和$n - 1$条边，**无重边和自环**

这句话告诉我们，题中的图实质上是一颗**无根树**。

看到题目中描述的**新道路**甚至可以是**自环**（题意简述中没写），我们先来考虑这种情况：

当新道路是一个自环时，我们不能通过它来从一个点到另一个点。而又因题目要求，新道路必须经过一次，那么我们只会在新道路上浪费$1$巡逻距离，如图：（这是**样例1**）

![](https://cdn.luogu.org/upload/pic/60199.png)

根据`DFS`的思想，我们要想遍历途中的每一个节点，需要访问每一条边两次（访问一次，回溯一次）。可以得到巡逻距离为$2 \times (8 - 1)  = 14$。

如果我们添加的新道路为节点$2$的自环，我们就会在访问节点$2$时被**强制**经过新道路一次，但是自环访问完后，依然要从$2$回溯至$3$，因此添加**任意的自环**都仅会使**最小访问距离变大**，舍弃。

那么我们考虑在**两个不同的节点之间**连边。

在一棵树上连一条边的话，原树就会变成一颗**基环树**。

如图：连接了$7,2$两点。
![](https://i.loli.net/2019/06/06/5cf87f5b6abe427205.png)

那么如果必须经过新建道路，我们就必须经过**基环树的环**，这样，环上的道路都只需要**遍历一次**（因为我们可以通过环来回到某一个点，这样就可以遍历完环后**直接回溯**，而环上的边就不需要再增加回溯时的费用）。于是考虑让这个环的长度**最大**，也就是说，让连边前的**链**最长，于是可以求出原树的直径`L`，然后将直径的两端连边，那么答案就是$2 * (n - 1) - L + 1$。

其中$2 * (n - 1)$为遍历原树的费用，L - 1为环的长度（不明白的动手在上面的图中数一数）。

~~30分到手了不做了~~

## 再来考虑$k == 2$的情况

和$k == 1$时相似，自环依然是被舍弃的，这个自己理解。

我们的第一条边依然连在直径两端，接着考虑第二条边的位置：
不难看出有如下两种可能：

- 环重叠
- 环不重叠

先来连接直径的**两端**，如图：

![](https://i.loli.net/2019/06/06/5cf87f7daad8544803.png)

这时的点$1 -  2 - 6 - 5 - 3$构成了一个环。我们考虑让环不重叠，连接点$7,8$，如图：

![](https://i.loli.net/2019/06/06/5cf87f8ec23b499834.png)

这时，新的环上的边也仅需经过一次，看上去非常的~~实惠~~。

别急，我们再来考虑一下**重叠**的情况，如图：

![](https://i.loli.net/2019/06/06/5cf87f9e7c8d375395.png)

这次连接了$4,7$两个点。看上去形成了一个更大的环，**节省**了中间重叠部分的费用，但是是这样嘛？

再次回去看一遍题目，发现了一个要求：

> 每条道路**必须**经过**至少一次**

这样，中间的道路就**不能不走**，如果走的话，不仅没有节省费用，相反会因为两个环都要经过而增加了**重叠长度**的费用。

结果就很显然了，需要连接两个**不会导致重环**的点。

那么要连接**哪两个**？怎么样才会导致**不重环**？

考虑$k == 1$时的做法，我们尝试找一条**次长链**，而且不与原链**重复**。有了第二个要求，我们反而会比较好做：

只需要将原直径上的边**毁掉**，然后建立一颗新的树，再在这棵树上求出直径就好了。

但是可能会出现某些~~dl~~数据，让连边后不得不重环，其实计算后我们就会发现，**不论是否重环**，都需要用原图长度减去两个环的大小，而重叠的部分被减了两次（不需要容斥的），这点我们通过求出直径后把直径上的边权值取反就能达到。

取反的实现类似于`EK`算法的`upd()`操作，需要在第一遍`bfs()`求直径时维护一个`pre[]`数组。

第二条直径可以**树上dp**来求。

# 参考代码

>  恰当的函数使用可以极大减少码量

```c++
//Author: zgjjj
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
        puts("");
    }
}

using namespace FastIO;

const int N = 100010;
const int M = N << 2;

template <typename T> inline T max(T a, T b) {return a > b ? a : b;}

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

int head[N], nxt[M], edge[M], ver[M], d[N], pre[N];
int cnt = 1, n, k, l, r, l1, l2;
bool v[N];

inline void add(int x, int y, int z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
}

inline int bfs(int s) {
    for(register int i = 1; i <= n; i++) d[i] = 0xcfcfcfcf, pre[i] = v[i] = 0;
    int ans = 0;
    d[s] = 0;
    queue <int> q;
    q.push(s);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 1;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i];
            if(v[y]) continue;
            d[y] = d[x] + z;
            pre[y] = i;
            q.push(y);
        }
    }
    for(register int i = 1; i <= n; i++) if(d[ans] < d[i]) ans = i;
    return ans;
}

void dp(int x) {
    v[x] = 1;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i], z = edge[i];
        if(v[y]) continue;
        dp(y);
        l2 = max(l2, d[y] + d[x] + z);
        d[x] = max(d[x], d[y] + z);
    }
}

int main() {
    n = read(), k = read();
    for(register int i = 1; i < n; i++) {
        int x = read(), y = read();
        add(x, y, 1);
        add(y, x, 1);
    }
    l = bfs(1);
    r = bfs(l);
    l1 = d[r];
    if(k == 1)
        return write(2 * (n - 1) - l1 + 1), 0;
    int x = r;
    while(x != l) {
        int i = pre[x];
        edge[i] = edge[i ^ 1] = -1;
        x = ver[i ^ 1];
    }
    for(register int i = 1; i <= n; i++) d[i] = v[i] = 0;
    dp(1);
    return write(2 * n - l1 -l2), 0;
}
```

