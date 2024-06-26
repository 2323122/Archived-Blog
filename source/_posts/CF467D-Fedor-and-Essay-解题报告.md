---
title: CF467D Fedor and Essay 解题报告
categories: 解题报告
date: 2019-08-02 10:52:38
tags:
	- 图论
	- Tarjan
	- 字符串
	- DP
	- 树形DP
top:
---

字符串太菜了。

# 题目链接

[(Luogu) CF467D Fedor and Essay](https://www.luogu.org/problem/CF467D)

# 题意简述

给你一篇文章和n对单词，可以将文中的单词按对应对换。

求对换后文章含字母'r'的最小数量，以及此时文章中所有单词长度之和的最小值。

<!--more-->

# 解题思想

单词可以看成点，然后每个单词含'r'的数量可以理解为其点权。

我们按对应关系建图后，对该图缩点，然后每个节点跑一遍树形DP，统计一下自底向上统计最小值。

懒得写哈希了，所以用了map。没想到已经是最优解了。

用哈希理论上会跑更快，~~但是出题人可能闲得无聊构造了哈希冲突~~

对于文章长度最小值，我们可以在DP时判一下两个DP值相等的情况，然后取$\min$就行了。

设$minn1[i]$为对应图上到单词$i$为止所有单词'r'的最小值，$minn2[i]$为对应图上到单词$i$位置符合题目条件的文章长度的最小值。

那么两个答案分别为：$ans_1 = \sum\limits_{1\le i\le n} minn1[scc[i]], ans_2 = \sum\limits_{1\le i\le n}minn2[scc[i]]$

要开**long long**

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cstring>
#include <string>
#include <map>

std::map <std::string, int> mp;

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(long long x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(long long x) {
        _write(x);
        putchar(' ');
    }
}

using namespace FastIO;

const int N = 3e5 + 6;
const int M = N << 2;

template <typename T> inline T min(T a, T b) {return a < b ? a : b;}
template <typename T> inline T max(T a, T b) {return a > b ? a : b;}

char ch[M];
long long ans1, ans2;
int n, m, sz;
int ver[N], nxt[M], head[M], cnt, Cnt;
int Ver[N], Nxt[M], Head[M];
int scc[N], dfn[N], low[N], idx, tot;
int sta[N], stop;
int id[N], num[N], len[N];
int minn1[N], minn2[N];
bool v[N], vis[N];

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
}

inline void Add(int x, int y) {
    Ver[++cnt] = y, Nxt[cnt] = Head[x], Head[x] = cnt;
}

void tarjan(int x) {
    dfn[x] = low[x] = ++idx;
    v[x] = 1;
    sta[++stop] = x;
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
    int now = 0;
    if(low[x] == dfn[x]) {
        tot++;
        while(now != x) {
            now = sta[stop--];
            v[now] = 0;
            scc[now] = tot;
            if(num[now] < minn1[tot] || (num[now] == minn1[tot] && len[now] < minn2[tot])) {
                minn1[tot] = num[now], minn2[tot] = len[now];
            }
        }
    }
}

inline void rebuild() {
    cnt = 0;
    for(register int x = 1; x <= sz; x++) {
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i];
            if(scc[x] != scc[y]) Add(scc[x], scc[y]);
        }
    }
}

void dp(int x) {
    if(vis[x]) return;
    vis[x] = 1;
    for(register int i = Head[x]; i; i = Nxt[i]) {
        int y = Ver[i];
        dp(y);
        if(minn1[y] < minn1[x]) {
            minn1[x] = minn1[y];
            minn2[x] = minn2[y];
        }
        else if(minn1[y] == minn1[x]) {
            minn2[x] = min(minn2[x], minn2[y]);
        }
    }
}

int main() {
    n = read();
    for(register int i = 1; i <= n; i++) {
        scanf("%s", ch);
        int l = strlen(ch), k = 0;
        for(register int j = 0; j < l; j++) {
            if(ch[j] >= 'A' && ch[j] <= 'Z') ch[j] = ch[j] - 'A' + 'a';
            if(ch[j] == 'r') k++;
        }
        int t = mp[ch]; if(!t) t = mp[ch] = ++sz; len[t] = l; num[t] = k;
        id[i] = t;
    }
    m = read();
    for(register int i = 1; i <= m; i++) {
        scanf("%s", ch);
        int l = strlen(ch), k = 0;
        for(register int j = 0; j < l; j++) {
            if(ch[j] >= 'A' && ch[j] <= 'Z') ch[j] = ch[j] - 'A' + 'a';
            if(ch[j] == 'r') k++;
        }
        int x = mp[ch]; if(!x) x = mp[ch] = ++sz; len[x] = l; num[x] = k;
        scanf("%s", ch);
        l = strlen(ch), k = 0;
        for(register int j = 0; j < l; j++) {
            if(ch[j] >= 'A' && ch[j] <= 'Z') ch[j] = ch[j] - 'A' + 'a';
            if(ch[j] == 'r') k++;
        }
        int y = mp[ch]; if(!y) y = mp[ch] = ++sz; len[y] = l; num[y] = k;
        add(x, y);
    }
    memset(minn1, 0x3f, sizeof(minn1));
    memset(minn2, 0x3f, sizeof(minn2));
    for(register int i = 1; i <= sz; i++) if(!dfn[i]) tarjan(i);
    rebuild();
    for(register int i = 1; i <= tot; i++) dp(i);
    for(register int i = 1; i <= n; i++) {
        ans1 += minn1[scc[id[i]]];
        ans2 += minn2[scc[id[i]]];
    }
    write(ans1);
    write(ans2);
    return 0;
}
```

