---
title: Trie与01树 学习报告
categories: 学习报告
date: 2019-04-09 17:33:10
tags: 
    - Trie
top:
toc: true
---

借着[异或粽子](<http://hboj.icu/problem/1064>)的契机学了一下**Trie**(~~zgjjj:原来这玩意不止能搞字符串~~)

<!--more-->

# 概述

Trie，字典树，显而易见是一个数据结构。它是解决字符串问题~~[不是这个](<http://hboj.icu/problem/1065>)~~的有力工具。

它的衍生物**01树**则可以很方便的处理**异或和最大**问题。

# Trie

这玩意很奇怪，不同于其他常见的数据结构，它的节点信息保存在数组下标中。

举个例子：`trie[p][ch - 'a'] != 0`代表字符`ch`存在于字典树，p是什么我们待会看。

## 插入字符串

```cpp
inline void ins(char *s) {
    int p = 0;
    for(register int i = 0; s[i]; i++) {
        char ch = s[i] - 'a';
        if(trie[p][ch] == 0) trie[p][ch] = ++tot;
        p = trie[p][ch];
    }
    end[p] = 1;
} 
```

这里的p就是当前节点的编号， tot存储的则是子节点之一的编号。

`end[p] `记录当前字符串是否结束，若**不维护**则树上串的前缀会被错误的判为存在。

## 查询字符串是否存在

```c++
inline bool check(char *s) {
    int p = 0;
    for(register int i = 0; s[i]; i++) {
        char ch = s[i] - 'a';
        p = trie[p][ch];
        if(p == 0) return 0;
    }
    return end[p];
}
```

注意判断该串是否完结。

# 01树

## 思想

和`Trie`相似，但是**01串的节点信息只有0和1**

我们将一个数进行**二进制划分**，从大到小考虑每一位，依次插入字典树，那么在寻找与已知串异或和最大的串，只需**贪心**地找与当前位**相反**的节点。

## 例题

[**P4551** 最长异或路径](https://www.luogu.org/problemnew/show/P4551)

树上统计，需要先**dfs**出到每个节点的异或和，然后将权值插入**01树**。

### 代码

```c++
#include <cstdio>
#include <algorithm>

using std::max;

const int N = 300010;
const int M = N << 1;

int head[N], ver[M], edge[M], nxt[M];
int cnt = 0, tot = 0, p;
int trie[N * 31][2], d[N], ans, n;

inline void add(int x, int y, int z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
    ver[++cnt] = x, edge[cnt] = z, nxt[cnt] = head[y], head[y] = cnt;
}

inline void ins(int x, int p) {
    for(register int i = 1 << 30; i; i >>= 1) {
        bool ch = x & i;
        if(!trie[p][ch]) trie[p][ch] = ++tot;
        p = trie[p][ch];
    }
}

inline int query(int x, int p) {
    int ans = 0;
    for(register int i = 1 << 30; i; i >>= 1) {
        bool ch = x & i;
        if(trie[p][ch ^ 1]) ans += i, p = trie[p][ch ^ 1];
        else p = trie[p][ch];
    }
    return ans;
}

void dfs(int x, int fa) {
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i], z = edge[i];
        if(y != fa) {
            d[y] = d[x] ^ z;
            dfs(y, x);
        }
    }
}

int main() {
    scanf("%d", &n);
    for(register int i = 1; i < n; i++) {
        int x, y, z;
        scanf("%d%d%d", &x, &y, &z);
        add(x, y, z);
    }
    dfs(1, 0);
    for(register int i = 1; i <= n; i++) {
        ins(d[i], p);
    }
    for(register int i = 1; i <= n; i++) {
        ans = max(ans, query(d[i], p));
    }
    printf("%d", ans);
    return 0;
}
```



