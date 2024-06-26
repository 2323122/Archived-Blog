---
title: CF1039D You Are Given a Tree 解题报告
categories: 解题报告
toc: true
date: 2019-08-26 11:16:23
tags: 
	- 树上问题
	- 根号分治
	- 贪心
	- 图论
top:
thumbnail: https://i.loli.net/2019/08/26/AvRNQehbfGFHLDx.png
---

神仙题，写了一上午

# 题目链接

[(Luogu)CF1039D You Are Given a Tree](https://www.luogu.org/problem/CF1039D)

# 题意简述

给你一棵树，分别求出划分成包含点数$k\in[1,n]$的链时最大的链的个数。

<!--more-->

# 解题思想

感谢小粉兔教我这道题QwQ

OwenOwl说要整体二分，~~那是啥~~

首先考虑暴力怎么打，对于每一个$k$，可以**贪心地**去做，一直找能连接的点，直到长度**第一次**$\geq k$，这时`链数++`，继续找下一段。时间复杂度为$O(n ^ 2)$

然后分析一波样例，我们发现对于$k\in(\sqrt n, n]$，答案是**单减**的，并且与**整除分块**相似，中间的某一段答案是一样的，所以我们可以二分找到一个段的右端点，然后暴力`check()`，更新整段的答案。这里的复杂度是$O(n\sqrt n \log n)$（log二分，根号区间，O(n)check）。

所以我们对于剩下的区间$[1, \sqrt n]$，直接暴力做，复杂度也是$O(n\sqrt n)$的，不会拖后腿。

总复杂度为$O(n\sqrt n\log n)$

这个复杂度已经可以过了

但是我们发现分治的两个区间复杂度并不均衡，所以考虑通过分配不同的分治范围来均衡掉两个不同的操作时间。

我们发现，对于大段区间，二分的复杂度是$O(\log n)$的，而暴力部分区间是$O(n)$的，于是我们可以通过设置区块分界点为$O(\sqrt{(n \times \log n)}) = O(\sqrt{n \log n})$来均衡两个复杂度。

`60`个点一共快了`10s`。。。

[根号nlogn断点](https://www.luogu.org/record/23329587)

[根号n断点](https://www.luogu.org/record/23329370)

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cmath>

namespace FastIO{
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
const int M = N << 1;

template <typename T> inline T max(T a, T b) {return a > b ? a : b;}

int n, size;
int head[N], ver[M], nxt[M], cnt;
int fa[N], ptn[N], idx;
int d[N], ans[N];

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
    ver[++cnt] = x, nxt[cnt] = head[y], head[y] = cnt;
}

void dfs(int x) {
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x]) continue;
        fa[y] = x;
        dfs(y);
    }
    ptn[++idx] = x;
}

inline int find(int k) {
    int ans = 0;
    for(register int i = 1; i <= n; i++) d[i] = 1;
    for(register int i = 1; i <= n; i++) {
        int x = ptn[i];
        if(fa[x] && (~d[fa[x]]) && (~d[x])) {
            if(d[x] + d[fa[x]] >= k) {
                ans++;
                d[fa[x]] = -1;
            }
            else {
                d[fa[x]] = max(d[fa[x]], d[x] + 1);
            }
        }
    }
    return ans;
}

int main() {
    n = read();
    size = sqrt(n * (log(n) / log(2)));
    for(register int i = 1; i < n; i++) {
        int x = read(), y = read();
        add(x, y);
    }
    dfs(1);
    ans[1] = n;
    for(register int i = 2; i <= size; i++) {
        ans[i] = find(i);
    }
    for(register int i = size + 1, l, r, k; i <= n; i = l + 1) {
        l = i, r = n, k = find(i);
        while(l + 1 < r) {
            int mid = (l + r) >> 1;
            if(find(mid) == k) l = mid;
            else r = mid;
        }
        for(register int j = i; j <= l; j++) ans[j] = k;
    }
    for(register int i = 1; i <= n; i++) {
        write(ans[i]);
    }
    return 0;
}
```

