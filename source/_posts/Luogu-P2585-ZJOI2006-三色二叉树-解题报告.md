---
title: Luogu P2585 [ZJOI2006]三色二叉树 解题报告
categories: 解题报告
toc: true
date: 2019-09-05 15:04:16
tags:
	- DP
	- 树形DP
top:
thumbnail: https://i.loli.net/2019/09/05/BvxO9nPoE7jamN6.png
---

$3kb$ 写着玩的，没想到1A了...

$24$ 行转移方程（大雾

# 题目链接

[P2585 [ZJOI2006]三色二叉树](https://www.luogu.org/problem/P2585)

# 题意简述

读入一颗二叉树，将其所有节点染成三种颜色，其中一个点左儿子右儿子和它自己的颜色都不能相同。求某种颜色最大和最小存在数量。

<!--more-->

# 解题思想

裸的树形 $\mathrm {DP}$，我们设 `dp[x][i]` 表示已经染完了 $x$ 的子树，并将 $x$ 染成颜色 $i$ 时问题的解，然后讨论左儿子与右儿子的颜色转移。

建树的话，可以一个 `dfs` 完成，有儿子再建节点，可以省空间。

# 参考代码

```c++
#include <cstdio>
#include <cstring>
#include <cctype>

const int N = 5e5 + 6;

inline int max(int a, int b) {return a > b ? a : b;}
inline int min(int a, int b) {return a < b ? a : b;}

int ch[N][2], tot = 1, rt = 1, ans1 = 0xcfcfcfcf, ans2 = 0x7fffffff;
int maxx[N][3], minn[N][3];

void readin(int x) {
    int son = getchar() - '0';
    if(son == 0) return;
    if(son == 1) {
        ch[x][0] = ++tot;
        readin(ch[x][0]);
    }
    else {
        ch[x][0] = ++tot;
        ch[x][1] = ++tot;
        readin(ch[x][0]);
        readin(ch[x][1]);
    }
}

void dfs(int x) {
    if(!ch[x][0]) {
        maxx[x][0] = 1, maxx[x][1] = maxx[x][2] = 0;
        minn[x][0] = 1, minn[x][1] = minn[x][2] = 0;
    }
    else if(!ch[x][1]) {
        dfs(ch[x][0]);
        maxx[x][0] = max(maxx[x][0], maxx[ch[x][0]][1] + 1);
        maxx[x][0] = max(maxx[x][0], maxx[ch[x][0]][2] + 1);
        maxx[x][1] = max(maxx[x][1], maxx[ch[x][0]][0]);
        maxx[x][1] = max(maxx[x][1], maxx[ch[x][0]][2]);
        maxx[x][2] = max(maxx[x][2], maxx[ch[x][0]][0]);
        maxx[x][2] = max(maxx[x][2], maxx[ch[x][0]][1]);
        minn[x][0] = min(minn[x][0], minn[ch[x][0]][1] + 1);
        minn[x][0] = min(minn[x][0], minn[ch[x][0]][2] + 1);
        minn[x][1] = min(minn[x][1], minn[ch[x][0]][0]);
        minn[x][1] = min(minn[x][1], minn[ch[x][0]][2]);
        minn[x][2] = min(minn[x][2], minn[ch[x][0]][0]);
        minn[x][2] = min(minn[x][2], minn[ch[x][0]][1]);
    }
    else {
        dfs(ch[x][0]);
        dfs(ch[x][1]);
        maxx[x][0] = max(maxx[x][0], maxx[ch[x][0]][1] + maxx[ch[x][1]][2] + 1);
        maxx[x][0] = max(maxx[x][0], maxx[ch[x][0]][2] + maxx[ch[x][1]][1] + 1);
        maxx[x][1] = max(maxx[x][1], maxx[ch[x][0]][0] + maxx[ch[x][1]][2]);
        maxx[x][1] = max(maxx[x][1], maxx[ch[x][0]][2] + maxx[ch[x][1]][0]);
        maxx[x][2] = max(maxx[x][2], maxx[ch[x][0]][0] + maxx[ch[x][1]][1]);
        maxx[x][2] = max(maxx[x][2], maxx[ch[x][0]][1] + maxx[ch[x][1]][0]);
        minn[x][0] = min(minn[x][0], minn[ch[x][0]][1] + minn[ch[x][1]][2] + 1);
        minn[x][0] = min(minn[x][0], minn[ch[x][0]][2] + minn[ch[x][1]][1] + 1);
        minn[x][1] = min(minn[x][1], minn[ch[x][0]][0] + minn[ch[x][1]][2]);
        minn[x][1] = min(minn[x][1], minn[ch[x][0]][2] + minn[ch[x][1]][0]);
        minn[x][2] = min(minn[x][2], minn[ch[x][0]][0] + minn[ch[x][1]][1]);
        minn[x][2] = min(minn[x][2], minn[ch[x][0]][1] + minn[ch[x][1]][0]);
    }
}

int main() {
    readin(rt);
    memset(maxx, 0xcf, sizeof maxx);
    memset(minn, 0x3f, sizeof minn);
    dfs(rt);
    for(register int i = 0; i < 3; i++) {
    	ans1 = max(ans1, maxx[rt][i]);
    	ans2 = min(ans2, minn[rt][i]);
	}
	printf("%d %d", ans1, ans2);
	return 0;
}
```

