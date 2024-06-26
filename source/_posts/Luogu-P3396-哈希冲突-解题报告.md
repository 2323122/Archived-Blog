---
title: Luogu P3396 哈希冲突 解题报告
categories: 解题报告
toc: true
date: 2019-08-26 08:35:24
tags: 根号分治
top:
thumbnail: https://i.loli.net/2019/08/26/4KhpnH65UmMNErg.jpg
---

根号分治入门题

# 题目链接

[P3396 哈希冲突](https://www.luogu.org/problem/P3396)

# 题意简述

给你一个序列，每次查询模`p`意义下某数总和，或者更改某一个数。

<!--more-->

# 解题思想

先考虑暴力怎么打，显然有两种操作：
一是先$O(n ^ 2)$预处理出模$i$意义下余数为$j$的答案，然后可以做到$O(1)$查询，$O(n)$修改。

二是对于每一次询问，暴力$O(n)$统计，修改是$O(1)$的。

两种做法的总复杂度都是$O(n ^ 2)$的，且第一种做法空间也会炸。

观察到两种做法查询修改互补，我们可以考虑结合一下：
对于模数较小的情况，我们可以预处理出来，这样空间不会炸，修改范围也会变小。

对于模数较大的情况，我们暴力统计，这样结果不会过多，查询不会炸。

这个较大较小的界，我们可以定为$O(\sqrt n)$，所以总复杂度为$O(n\sqrt n)$

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cmath>

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

const int N = 150010;
const int SIZE = 400;

int n, m;
int a[N];
int f[SIZE][SIZE];

int main() {
    n = read(), m = read();
    int size = sqrt(n);
    for(register int i = 1; i <= n; i++) {
        a[i] = read();
        for(register int j = 1; j <= size; j++) {
            f[j][i % j] += a[i];
        }
    }
    while(m--) {
        char s[3];
        scanf("%s", s);
        int x = read(), y = read();
        if(s[0] == 'A') {
            if(x <= size) write(f[x][y]);
            else {
                int ans = 0;
                for(register int i = y; i <= n; i += x) {
                    ans += a[i];
                }
                write(ans);
            }
        }
        else {
            for(register int i = 1; i <= size; i++) {
                f[i][x % i] += y - a[x];
            }
            a[x] = y;
        }
    }
    return 0;
}
```

