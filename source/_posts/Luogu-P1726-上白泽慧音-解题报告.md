---
title: Luogu P1726 上白泽慧音 解题报告
categories: 解题报告
date: 2019-05-02 20:32:58
tags: 
  - 图论
  - Tarjan
  - 强连通分量
top:
---

又双叒叕写挂了TarjanQAQ

# 题目链接

[**P1726** 上白泽慧音](https://www.luogu.org/problemnew/show/P1726) 

# 题意简述

给你一张有向图，求出最大的**强连通子图**，并输出**字典序**最小的一个。

<!--more-->

# 解题思想

emmmm挺裸的题。

只需要**Tarjan**求强连通分量就行了。

至于**字典序**的处理：

不妨在**缩点**时**更新**答案，然后**从小到大**枚举每个点，第一个**所属强连通分量大小与答案相同**的**强连通分量**就是字典序**最小**的那个。

这里是枚举的代码：

```c++
for(register int i = 1; i <= n; i++) {
    if(scc[i] == ans) {
        ans = i;
        break;
    } 
}
```

输出答案：

```c++
for(register int i = ans; i <= n; i++) {
    if(scc[i] == scc[ans]) write(i);
}
```

注意一下细节就好了。

至于写挂的Tarjan...

```c++
if(low[x] == dfn[x]) {
    tot++;
    while(s.top()) {
        int y = s.top();
        s.pop();
        scc[y] = tot;
        v[y] = 0;//这里要清零！！！！！！！
        num[tot]++;
        if(x == y) break;
    }
    if(num[tot] > num[ans]) ans = tot;
}
```

**`v[]`要清零....**

# 参考代码

```c++
#include <cstdio>
#include <cctype>

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0'; ch = getchar();}
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

using namespace FastIO;

const int N = 5010;
const int M = 100010;

template <typename T> T min(T a, T b) {return a < b ? a : b;}

template <typename T>
class stack {
    private:
    int stop;
    T s[N];
    public:
    stack() {stop = 0;}
    inline bool empty() {return !stop;}
    inline void push(T x) {s[++stop] = x;}
    inline void pop() {if(!empty()) stop--;}
    inline T top() {return s[stop];}
};

stack <int> s;

int head[N], ver[M], nxt[M], scc[N], num[N], dfn[N], low[N];
int cnt, idx, tot, ans, n, m;
bool v[N];

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
            v[y] = 0;
            num[tot]++;
            if(x == y) break;
        }
        if(num[tot] > num[ans]) ans = tot;
    }
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read(), opt = read();
        add(x, y);
        if(opt == 2) add(y, x);
    }
    for(register int i = 1; i <= n; i++) {
        if(!dfn[i]) tarjan(i);
    }
    printf("%d\n", num[ans]);
    for(register int i = 1; i <= n; i++) {
        if(scc[i] == ans) {
            ans = i;
            break;
        } 
    }
    for(register int i = ans; i <= n; i++) {
        if(scc[i] == scc[ans]) write(i);
    }
    return 0;
}
```

