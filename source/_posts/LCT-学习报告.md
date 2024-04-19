---
title: LCT 学习报告
date: 2019-07-15 16:27:57
tags:
    - LCT
    - 数据结构
categories: 学习报告
top:
toc: true
---

~~LCT是常数巨大的树剖~~

<!--more-->

# 问题引入

~~[P4383 [八省联考2018]林克卡特树lct](https://www.luogu.org/problemnew/show/P4383)~~

emmm，不是这个。

给你一棵树，有点权。

## Subtask 1

> 询问x到y最短路径上的点权和。

这个简单，前缀和就好了。

## Subtask 2

> x到y最短路径上每个点点权加k

树上差分！

我还会树剖！

## Subtask 3

> 改变树的结构

emmmm...

由这个问题，我们提出了$LCT$。

# 定义、基本操作及原理

我们先来回顾一下**树剖**。

在树剖中，我们使用**线段树**来维护链上的信息，其实也可以使用$Splay$的，但是它常数太大了，~~而且难写~~，没有必要。

考虑一下树剖的弊病：一旦一棵树确定了，它的重链也就确定了。

所以对于动态问题，树剖就没什么办法了。

这时需要用到$LCT$

## 一些定义

**辅助树**：以$Splay$作为辅助树，原树被划分成若干个$Splay$。

**重儿子**：重儿子与父亲节点同在一棵$Splay$中，**一个节点最多**
**只能有一个重儿子**。

注意这里的重儿子的定义与树剖不同。

**轻儿子**：其他儿子。

**重边**：指向重儿子的边。

**轻边**：其他边。

**重链**：由重儿子连接构成的链。

**轻链**：其他链。

比如这是一棵树：
![](https://i.loli.net/2019/07/15/5d2c3ea7487d866400.png)

我们规定其中的一些节点为重儿子：
![](https://i.loli.net/2019/07/15/5d2c408d382d462450.png)

红色的是重儿子，蓝色的是轻儿子。

加粗的边为重边。

然后把它扭一扭（？），把$Splay$划分出来：

![](https://i.loli.net/2019/07/15/5d2c4c448047b42931.png)

这棵$LCT$中的$Splay$被**红笔**圈了出来，为了满足$Splay$的性质，每棵$Splay$都在旋转后由轻边重新连接。

可以发现，对一棵$Splay$中序遍历，可以得到该$Splay$维护的重链自链顶向下访问的顺序。

在每条重链在一棵$Splay$上的同时，每个**轻儿子**单独构成一棵$Splay$。

同时，为了方便维护树的结构，我们将每棵$Splay$**根**的父亲节点设为该链**链顶**的父亲节点。

对比原树，我们发现，在此操作的过程中，一些边被断开，一些边被连上。

## 基本操作

### 访问（Access）

#### 简介及原理

LCT的**核心操作**。

顾名思义，访问就是从**根节点**出发，访问到某个节点。在访问的过程中，每个**路径上的点**都被设置为**重儿子**，即**访问路径**被设置为**重链**。

由于每个节点**仅能有一个**重儿子，所以我们访问路上的节点时，需要**断掉**它与**之前的重儿子**的连边，并加入到$Splay$中。

我们考虑自**访问节点**向**根**处理，对于访问到的每个节点，我们进行$Splay$操作，此时，其右子树即为原重链上需要断掉的一段。切断后，将节点右儿子设置为当前重儿子即可。

在LCT中，`fa[x]`记录**全局**父节点信息，与所在$Splay$无关，但是`ch[x][i]`记录的则是当前$Splay$的信息，因此，**轻儿子不会被记录**，我们只需要更改每个节点的右儿子信息，不需要更新右儿子的父节点信息。

#### 代码实现

``` c++
inline void access(int p) {
    for(register int q = 0; p; p = tr[p].fa) {
        splay(p);
        tr[p].ch[1] = q;
        upd(p);
    }
}
```

### 换根（MakeRoot）

#### 简介及原理

可以将某个节点设置为原树的根。

考虑如下性质：

> 对一棵$Splay$中序遍历，可以得到该$Splay$维护的重链自链顶向下访问的顺序。

在访问操作后，**该节点至根**的路径形成一条**重链**，我们将该节点$Splay$至根，此时，链顶节点即为原树的根，且该节点存在于链底，考虑$Splay$反转操作，即可将其转至链顶，即为原树根。

#### 代码实现

```c++
inline void makert(int p) {
    access(p);
    splay(p);
    reverse(p);
}
```

### 找根（FindRoot）

#### 简介及原理

找到原树的树根。

由于$LCT$可能维护了一个**森林**，所以有时候需要找到某个节点属于原森林中哪棵树。

考虑访问，此时自原树根节点到该节点的路径构成了一条重链，将该点伸展至根，此时**链底**即为原树根。

由于伸展操作，此时的$Splay$以**深度**为秩有序，所以**最左的左儿子**即为链底。

#### 代码实现

```c++
inline int findrt(int p) {
    access(p);
    splay(p);
    while(tr[p].ch[0]) p = tr[p].ch[0];
    return p;
}
```

### 分离树链（Split）

#### 简介及原理

~~树链剖分~~

将原树中两节点间的简单路径分离出来。

考虑将两个节点间的简单路径构造为**一条重链**。

不妨先将树根**设置**为一个节点，并**访问**另一个，这样两个节点就在**同一条重链**上了。

此时再对访问的节点进行**伸展**操作，操作后的链表现为$Splay$的**根及其左子树**。

#### 代码实现

```c++
inline void split(int u, int v) {
    makert(u);
    access(v);
    splay(v);
}
```





### 林克（Link）

#### 简介及原理

连接两棵原树。

考虑在根与链底之间连**轻边**，因此我们需要将一个节点设为根，然后连在另一个节点上就行了。

#### 代码实现

```c++
inline void link(int u, int v) {
    makert(u);
    tr[u].fa = v;
}
```

### 卡特（Cut）

#### 简介及原理

切断原树。

考虑断除两个节点之间的重边。

先将边**分离**出来，然后**切断父子关系**即可。

如果左儿子不是$u$或者左儿子有右儿子，那么表示不存在$(u, v)$这条边。

#### 代码实现

```c++
inline void cut(int u, int v) {
    split(u, v);
    if(tr[v].ch[0] != u || tr[tr[v].ch[0]].ch[1]) return;
    tr[u].fa = tr[v].ch[0] = 0;
}
```

### 树（Tree）

~~没有这个东西~~

### 单点修改（Modify）

#### 简介及原理

改变单点权值。

为避免更新操作带来的大常数~~（虽然本来常数就够大了）~~，我们先将待修节点**伸展至根**，再进行修改。

#### 代码实现

```c++
inline void modify(int u, int w) {
    access(u);
    splay(u);
    tr[u].val = w;
    upd(u);
}
```

### 判根（Isroot）

#### 简介及原理

判断当前节点是否为一棵$Splay$的根。

上文中提到了`ch[x][i]`与`fa[x]`的差异性，我们可以利用这个差异来检查某个节点与父亲的连边是否为重边。

若不是重边，则`ch[fa[x]][0]  != x && ch[fa[x]][1] != x`

代码很自然。

#### 代码实现

```c++
inline bool isroot(int x) {
    return tr[fa[x]].ch[0] != x && tr[fa[x]].ch[1] != x;
}
```

### 旋转（Rotate）

#### 简介及原理

平衡树基操。

不过判断$Splay$根的方式有所改变，旋转的方式也有所改变。

主要是将判断爷爷节点是否存在改为判断父节点是否为根。

区别不大。

#### 代码实现

```c++
inline void rotate(int p) {
    bool t = !isleft(p);
    int fa = tr[p].fa, ffa = tr[fa].fa;
    if(!isroot(fa)) tr[ffa].ch[!isleft(fa)] = p; // 这句一定要写在最前面
    tr[fa].ch[t] = tr[p].ch[!t];
    tr[p].ch[!t] = fa;
    tr[tr[fa].ch[t]].fa = fa;
    tr[p].fa = ffa;
    tr[fa].fa = p;
    upd(p), upd(fa); // 这里的upd(p)可以省略，但是upd(fa)绝对不能省，不放心可以都写上
}
```

### 伸展（Splay）

#### 简介及原理

$Splay$基操

反转标记的存在会使LCT出锅，所以Splay前要将标记**下放至**待伸展节点。

#### 代码实现

```c++
//反转
inline void reverse(int p) {
    swap(tr[p].ch[0], tr[p].ch[1]);
    tr[p].rev ^= 1;
}

//下放
inline void pushdown(int p) {
    if(tr[p].rev) {
        if(tr[p].ch[0]) reverse(tr[p].ch[0]);
        if(tr[p].ch[1]) reverse(tr[p].ch[1]);
        tr[p].rev ^= 1;
    }
}

//手写栈可换为STL，也可递归调用函数
int sta[MAXN], stop;

//从根下放到p节点
inline void pushto(int p) {
    stop = 0;
    while(!isroot(p)) {
        sta[stop++] = p;
        p = tr[p].fa;
    }
    sta[stop++] = p;
    while(stop) {
        pushdown(sta[--stop]);
    }
}

//伸展
inline void splay(int p) {
    pushto(p);
    for(int fa = tr[p].fa; !isroot(p); rotate(p), fa = tr[p].fa) {
        if(!isroot(fa)) rotate(isleft(fa) == isleft(p) ? fa : p);
    }
    upd(p);
}
```

# 应用

## 直接运用

### 操作及原理

单点修改，动态加边、断边，保证结构是一棵树，查询两点间简单路径的信息。

用LCT直接维护即可。

### 例题1

[**P3690** 【模板】Link Cut Tree （动态树）](https://www.luogu.org/problemnew/show/P3690) 

维护两点间最短路径的异或和即可。

比树剖板子题和蔼多了。

```c++
#include <cstdio>
#include <cctype>

#define LL long long

namespace FastIO {
    inline LL read() {
        char ch = getchar(); LL r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(LL x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(LL x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

template <typename T> inline void swap(T &x, T &y) {T tmp = x; x = y, y = tmp;}

const int N = 300005;

struct Node {
    int ch[2], fa, val, xsum;
    bool rev;
} tr[N];

inline bool isleft(int p) {
    return tr[tr[p].fa].ch[0] == p;
}

inline bool isroot(int p) {
    return tr[tr[p].fa].ch[0] != p && tr[tr[p].fa].ch[1] != p;
}

inline void upd(int p) {
    tr[p].xsum = tr[p].val ^ tr[tr[p].ch[0]].xsum ^ tr[tr[p].ch[1]].xsum;
}

inline void reverse(int p) {
    swap(tr[p].ch[0], tr[p].ch[1]);
    tr[p].rev ^= 1;
}

inline void pushdown(int p) {
    if(tr[p].rev) {
        if(tr[p].ch[0]) reverse(tr[p].ch[0]);
        if(tr[p].ch[1]) reverse(tr[p].ch[1]);
        tr[p].rev ^= 1;
    }
}

int sta[N], stop;

inline void pushto(int p) {
    stop = 0;
    while(!isroot(p)) {
        sta[stop++] = p;
        p = tr[p].fa;
    }
    sta[stop++] = p;
    while(stop) {
        pushdown(sta[--stop]);
    }
}

inline void rotate(int p) {
    bool t = !isleft(p); int fa = tr[p].fa, ffa = tr[fa].fa;
    tr[p].fa = ffa; if(!isroot(fa)) tr[ffa].ch[!isleft(fa)] = p;
    tr[fa].ch[t] = tr[p].ch[!t]; tr[tr[fa].ch[t]].fa = fa;
    tr[p].ch[!t] = fa; tr[fa].fa = p;
    upd(fa);
}

inline void splay(int p) {
    pushto(p);
    for(int fa = tr[p].fa; !isroot(p); rotate(p), fa = tr[p].fa) {
        if(!isroot(fa)) rotate(isleft(fa) == isleft(p) ? fa : p);
    }
    upd(p);
}

inline void access(int p) {
    for(int q = 0; p; q = p, p = tr[p].fa) {
        splay(p);
        tr[p].ch[1] = q;
        upd(p);
    }
}

inline void makert(int p) {
    access(p);
    splay(p);
    reverse(p);
}

inline int findrt(int p) {
    access(p);
    splay(p);
    while(tr[p].ch[0]) p = tr[p].ch[0];
    return p;
}

inline void split(int u, int v) {
    makert(u);
    access(v);
    splay(v);
}

inline void link(int u, int v) {
    makert(u);
    tr[u].fa = v;
}

inline void cut(int u, int v) {
    split(u, v);
    if(tr[v].ch[0] != u || tr[u].ch[1]) return;
    tr[u].fa = tr[v].ch[0] = 0;
}

inline void modify(int u, int w) {
    access(u);
    splay(u);
    tr[u].val = w;
    upd(u);
}

int n, m, op, x, y;

int main() {
    n = read(); m = read();
    for(int i = 1; i <= n; i++) {
        tr[i].val = tr[i].xsum = read();
    }
    while(m--) {
        op = read(); x = read(); y = read();
        switch(op) {
            case 0:
                split(x, y);
                write(tr[y].xsum);
                break;
            case 1:
                link(x, y);
                break;
            case 2:
                cut(x, y);
                break;
            case 3:
                modify(x, y);
        }
    }
    return 0;
}
```

### 例题2

[**P3203** [HNOI2010]弹飞绵羊](https://www.luogu.org/problemnew/show/P3203) 

题目没有那么直白了，我们考虑建图。

观察到$k$仅能为正，考虑把每个点与它能抵达的所有点连边，这样就建成了一颗树。

对于可能跳过的情况，直接连接到一个超级汇点$n + 1$上。

建完图后，我们可以发现，每个点的答案即为它在以$n + 1$为根的有根树中的深度。

对于改$k$这个操作，我们发现连边的依据是$k$，那么这个操作相当于断旧边、连新边，考虑LCT。

此时我们再想想答案（即节点深度）怎么在LCT中体现。

对于每个被询问的点，不妨将它到$n + 1$节点的链$Split$出来，此时，这个节点的子树大小即为节点深度。

LCT维护即可。

```c++
#include <cstdio>
#include <cctype>

#define LL long long

namespace FastIO {
    inline LL read() {
        char ch = getchar(); LL r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(LL x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(LL x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

const int N = 200005;

template <typename T> inline void swap(T &x, T &y) {T tmp = x; x = y, y = tmp;}
template <typename T> inline T min(T a, T b) {return a < b ? a : b;}

struct Node {
    int ch[2], fa, sum;
    bool rev;
} tr[N];

inline bool isleft(int p) {
    return tr[tr[p].fa].ch[0] == p;
}

inline bool isroot(int p) {
    int fa = tr[p].fa;
    return tr[fa].ch[0] != p && tr[fa].ch[1] != p;
}

inline void upd(int p) {
    int ls = tr[p].ch[0], rs = tr[p].ch[1];
    tr[p].sum = tr[ls].sum + tr[rs].sum + 1;
}

inline void reverse(int p) {
    swap(tr[p].ch[0], tr[p].ch[1]);
    tr[p].rev ^= 1;
}

inline void pushdown(int p) {
    int ls = tr[p].ch[0], rs = tr[p].ch[1];
    if(tr[p].rev) {
        if(ls) reverse(ls);
        if(rs) reverse(rs);
        tr[p].rev ^= 1;
    }
}

int sta[N], stop;

inline void pushto(int p) {
    stop = 0;
    while(!isroot(p)) {
        sta[stop++] = p;
        p = tr[p].fa;
    } 
    pushdown(p);
    while(stop) {
        pushdown(sta[--stop]);
    }
}

inline void rotate(int p) {
    bool t = !isleft(p); int fa = tr[p].fa, ffa = tr[fa].fa;
    tr[p].fa = ffa; if(!isroot(fa)) tr[ffa].ch[!isleft(fa)] = p;
    tr[fa].ch[t] = tr[p].ch[!t]; tr[tr[fa].ch[t]].fa = fa;
    tr[p].ch[!t] = fa; tr[fa].fa = p;
    upd(fa);
}

inline void splay(int p) {
    pushto(p);
    for(int fa = tr[p].fa; !isroot(p); rotate(p), fa = tr[p].fa) {
        if(!isroot(fa)) rotate(isleft(fa) == isleft(p) ? fa : p);
    }
    upd(p);
}

inline void access(int p) {
    for(int q = 0; p; q = p, p = tr[p].fa) {
        splay(p);
        tr[p].ch[1] = q;
        upd(p);
    }
}

inline void makert(int p) {
    access(p);
    splay(p);
    reverse(p);
}

inline int findrt(int p) {
    access(p);
    splay(p);
    while(tr[p].ch[0]) p = tr[p].ch[0];
    return p;
}

inline void split(int u, int v) {
    makert(u);
    access(v);
    splay(v);
}

inline void link(int u, int v) {
    makert(u);
    tr[u].fa = v;
}

inline void cut(int u, int v) {
    split(u, v);
    if(tr[v].ch[0] != u || tr[tr[v].ch[0]].ch[1]) return;
    tr[u].fa = tr[v].ch[0] = 0;
}

int n, m, k[N], op, ut, wt;

int main() {
    n = read();
    tr[n + 1].sum = 1;
    for(int i = 1; i <= n; i++) {
        k[i] = read();
        tr[i].sum = 1;
    }
    for(int i = 1; i <= n; i++) {
        int x = i, y = i + k[i];
        y = min(y, n + 1);
        link(x, y);
    }
    m = read();
    while(m--) {
        op = read(); ut = read() + 1;
        if(op == 1) {
            split(ut, n + 1);
            write(tr[n + 1].sum - 1);
        }
        else {
            wt = read();
            cut(ut, min(ut + k[ut], n + 1));
            k[ut] = wt;
            link(ut, min(ut + k[ut], n + 1));
        }
    }
    return 0;
}
```

## 处理子树信息

### 操作及原理

在树剖里，我们利用子树节点的`dfs`序的**连续性**建立**线段树**解决了子树信息的处理。

但是在LCT中，**根是会变的**，维护`dfs`序似乎并不靠谱。我们考虑**直接维护**子树的信息：

不妨对每一个点维护一个**虚**子树的信息，因为只有`access()`和`link()`会导致其改变，而`cut()`断开的是一条重边，不会影响一个虚子树的信息，所以只需要`upd()`一下就行了。

### 例题

太菜了只找到了大融合。

[**P4219** [BJOI2014]大融合](https://www.luogu.org/problemnew/show/P4219) 

首先考虑一条边的负载与什么有关。通过排列组合的知识我们可以知道，负载等于起点个数乘以终点个数。

起点个数与终点个数体现在树上就是断边后两个节点子树大小。

由于动态连边，我们要用LCT维护这个。

照着上面讲的做就行了。

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
        puts("");
    }
}

using namespace FastIO;

const int N = 1e5 + 5;

template <typename T> inline void swap(T &x, T &y) {T tmp = x; x = y, y = tmp;}

struct Node {
    int ch[2], fa, tsz, sz;
    bool rev;
} tr[N];

inline void upd(int p) {
    tr[p].sz = tr[tr[p].ch[0]].sz + tr[tr[p].ch[1]].sz + tr[p].tsz + 1;
}

inline bool isleft(int p) {
    return tr[tr[p].fa].ch[0] == p;
}

inline bool isroot(int p) {
    return tr[tr[p].fa].ch[0] != p && tr[tr[p].fa].ch[1] != p;
}

inline void reverse(int p) {
    tr[p].rev ^= 1;
    swap(tr[p].ch[0], tr[p].ch[1]);
}

inline void pushdown(int p) {
    if(tr[p].rev) {
        if(tr[p].ch[0]) reverse(tr[p].ch[0]);
        if(tr[p].ch[1]) reverse(tr[p].ch[1]);
        tr[p].rev ^= 1;
    }
}

int sta[N], stop;

inline void pushto(int p) {
    stop = 0;
    while(!isroot(p)) {
        sta[stop++] = p;
        p = tr[p].fa;
    }
    sta[stop++] = p;
    while(stop) {
        pushdown(sta[--stop]);
    }
}

inline void rotate(int p) {
    bool t = !isleft(p); int fa = tr[p].fa, ffa = tr[fa].fa;
    tr[p].fa = ffa;
    if(!isroot(fa)) tr[ffa].ch[!isleft(fa)] = p;
    tr[fa].ch[t] = tr[p].ch[!t];
    tr[tr[fa].ch[t]].fa = fa;
    tr[p].ch[!t] = fa;
    tr[fa].fa = p;
    upd(fa);
}

inline void splay(int p) {
    pushto(p);
    for(int fa = tr[p].fa; !isroot(p); rotate(p), fa = tr[p].fa) {
        if(!isroot(fa)) rotate(isleft(fa) == isleft(p) ? fa : p);
    }
    upd(p);
}

inline void access(int p) {
    for(int q = 0; p; q = p, p = tr[p].fa) {
        splay(p);
        tr[p].tsz += tr[tr[p].ch[1]].sz;
        tr[p].tsz -= tr[q].sz;
        tr[p].ch[1] = q;
        upd(p);
    }
}

inline void makert(int p) {
    access(p);
    splay(p);
    reverse(p);
}

inline int findrt(int p) {
    access(p);
    splay(p);
    while(tr[p].ch[0]) p = tr[p].ch[0];
    return p;
}

inline void split(int u, int v) {
    makert(u);
    access(v);
    splay(v);
}

inline void link(int u, int v) {
    split(u, v);
    tr[u].fa = v;
    tr[v].tsz += tr[u].sz;
    upd(v);
}

inline void cut(int u, int v) {
    split(u, v);
    if(tr[v].ch[0] != u || tr[u].ch[1]) return;
    tr[v].ch[0] = tr[u].fa = 0;
}

int n, q;

int main() {
    n = read(), q = read();
    while(q--) {
        char op = getchar();
        int x = read(), y = read();
        if(op == 'A') link(x, y);
        else {
            split(x, y);
            write(tr[x].sz * (tr[y].sz - tr[x].sz));
        }
    }
    return 0;
}
```

~~太菜了把`rotate()`里的`upd(fa)`写成了`upd(p)`，调了好久，自己打自己脸qwq~~

## LCT动态加边求最大生成树

### 操作及原理

还没学会咕咕咕咕咕...