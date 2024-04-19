---
title: Luogu P2886 [USACO07NOV]牛继电器Cow Relays 解题报告
categories: 解题报告
date: 2019-05-14 15:45:51
tags: 
  - 图论
  - 矩阵快速幂
  - Floyd   
top:
---

~~矩乘x图论~~

题目描述极其不清楚差评。

# 题目链接

[P2886 [USACO07NOV]牛继电器Cow Relays](<https://www.luogu.org/problemnew/show/P2886>)

# 题意简述

给定一张无向图，边号为T，求`S->E`的经过N条边的最短路。

<!--more-->

# 解题思想

看到边数可以想到使用`Floyd`，但是$n$过大，于是可以推测**点编号分布稀疏**，在读入时**离散化**一下。

如何维护**经过的边数**为N？考虑两个矩阵$A = x \times x,B = y \times y$分别维护了点数为$x, y$的最短路，我们将其**合并**，便可以得到一个点数为$x + y$的最短路矩阵。类似地，考虑将原**距离矩阵**合并**$n - 1$**次，我们便可以得到经过N条边的最短路。

但是观察题目中的$N$的范围，我们发现：太大了。又注意到每次合并的矩阵**相同**，都为**原距离矩阵**，不妨考虑**矩阵快速幂**，但是重载的乘法需要改为**合并的更新操作**而不是一般意义下的**矩阵乘法**，这样，就可以$O(\log n)$得到答案。

~~可以叫它广义矩阵快速幂嘛QAQ~~

# 参考代码

```c++
#include <cstdio>
#include <cstring>

#define add(x, y, z) d.m[x][y] = z;

const int N = 1000010;

template <typename T> T min(T a, T b) {return a < b ? a : b;}

int n, t, s, e, tot;
int num[N];

struct Mat {
    int m[501][501];
    Mat operator * (const Mat &a) const {
        Mat ans;
        memset(ans.m, 0x3f, sizeof(ans.m));
        for(register int k = 1; k <= tot; k++) {
            for(register int i = 1; i <= tot; i++) {
                for(register int j = 1; j <= tot; j++) {
                    ans.m[i][j] = min(ans.m[i][j], m[i][k] + a.m[k][j]);
                }
            }
        }
        return ans;
    }
}f, d;

int main() {
    memset(d.m, 0x3f, sizeof(d.m));
    scanf("%d %d %d %d", &n, &t, &s, &e);
    for(register int i = 1; i <= t; i++) {
        int x, y, z;
        scanf("%d%d%d", &z, &x, &y);
        if(!num[x]) num[x] = ++tot;
        if(!num[y]) num[y] = ++tot;
        add(num[x], num[y], z);
        add(num[y], num[x], z);
    }
    f = d;
    n--;
    for(; n; n >>= 1, d = d * d) if(n & 1) f = f * d;
    printf("%d", f.m[num[s]][num[e]]);
    return 0;
}
```

