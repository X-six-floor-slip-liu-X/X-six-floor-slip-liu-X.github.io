---
comments: true
---

# 连通性相关不常用算法

本文包含三连通，耳分解，双极定向。

前置知识：点双，边双。

本来以为没用，学了感觉好像还是有点用的。

## 三连通

### 割集

在讨论无向图三连通前，我们先定义割集。

割集为一个边集，满足一个连通图中某个连通的点对删掉这个边集后就不连通了。我们熟悉的边双连通图即所有割集均不小于 $2$ 的连通图，同理，我们称两个点是三连通的当且仅当它们两个之间不存在大小小于 $3$ 的割集，边三连通图即所有割集均不小于 $3$ 的连通图。

我们不妨先来讨论如何判断一个边集是否为割集。有经典构造：

//// admonition | [P10778](https://www.luogu.com.cn/problem/P10778) BZOJ3569 DZY Loves Chinese II
    type: example

> 题意

- 给定一张 $n$ 个点 $m$ 条边的无向图。
- $q$ 次询问，每次询问一个边集是否为割集。
- $1\le n\le 10^5,1\le m\le 5\times 10^5,1\le q\le 5\times 10^4$，每次给定边集的大小 $k$ 不超过 $15$。

> 题解

经典构造。

考虑给每条边赋边权，使得一个点所有出边异或和为 $0$，显然这些边就是一个割集。

对于两个相邻的点，我们发现这个连通块向外连的边异或和也是 $0$，因为两点之间边的边权就是其它边的异或和。对于更多点组成的连通块也是同理。于是“边集存在一个异或和为 $0$ 的子集”是“这个边集是割集”的必要条件，当我们给边赋的权值尽可能随机时就不容易错了。

那么怎么赋边权才能让每个点所有出边异或和为 $0$ 呢？构造是找一棵 dfs 生成树，给非树边随机赋值 $[0,2^{64})$，树边赋值为所有包含它的非树边的异或和，正确性显然。

求解时需要线性基，复杂度 $O(n+m+qk\log V)$，其中 $V$ 是赋的边权的值域。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(int i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(int i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using i64=long long;
using u64=unsigned long long;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=2e5+5,M=5e5+5,inf=0x3f3f3f3f;
int n,m,q,vis[N],dpt[N];
vector<pii> e[N],son[N];
u64 val[M],vv[N];
mt19937_64 mr(20071221);
void dfs(int x,int fa){
    vis[x]=1;
    dpt[x]=dpt[fa]+1;
    for(auto i:e[x]){
        int v=i.fi,p=i.se;
        if(v==fa) continue;
        if(vis[v]){
            if(dpt[v]<dpt[x]){
                val[p]=mr();
                vv[x]^=val[p];vv[v]^=val[p];
            }
            continue;
        }
        son[x].push_back(i);
        dfs(v,x);
        val[p]=vv[v];
        vv[x]^=vv[v];
    }
}
struct linear_basis{
    u64 c[64];
    void clear(){
        mem(c,0);
    }
    bool insert(u64 x){
        forup(i,0,63){
            if(!((x>>i)&1)) continue;
            if(!c[i]){
                c[i]=x;
                return true;
            }else{
                x^=c[i];
            }
        }
        return false;
    }
}mt;
signed main(){
    n=read();m=read();
    forup(i,1,m){
        int u=read(),v=read();
        e[u].push_back(mkp(v,i));
        e[v].push_back(mkp(u,i));
    }
    dfs(1,0);
    q=read();
    int lans=0;
    forup(i,1,q){
        int k=read();
        mt.clear();
        bool ans=true;
        forup(j,1,k){
            int p=read()^lans;
            if(!mt.insert(val[p])) ans=false;
        }
        puts(ans?"Connected":"Disconnected");
        lans+=ans;
    }
}
```

///

////

### 三连通分量

那么我们如何判断一个图是否是三连通的呢？

如果一张图不是三连通的，则必然存在大小不超过 $2$ 的割集，即在经过上文的赋值后存在两条边边权相等，这个是好做的，搞个哈希表（`std::unordered_map`）即可。

不难发现三连通是具有传递性的，即若 $a,b$ 三连通，$b,c$ 三连通，那么 $a,c$ 也是三连通的。那么我们也可以类似双连通分量地定义三连通分量。另外一个有趣的事实是边三连通分量不一定是连通的。

那么我们怎么求三连通分量呢？显然我们只要找到所有割集就能分出三连通分量，考虑一个大小不超过 $2$ 的割集无非三种情况：

1. 只有一条树边。
1. 一条树边和一条非树边。
1. 两条树边。

不难发现，前两种情况都会将若干个子树和整张图的其余部分分开。一个想法是再给每个点赋一个 $[0,2^{64})$ 的权值，权值相同表示在同一三连通分量。初始所有点都是 $0$，每次找到割集就将对应的部分全部异或一个随机数，最后将权值相同的点设为同一集合，这个可以树上前缀和。

对于第三种情况，必然是覆盖两条边的返祖边集合相同，那么这两条边 $(u,v)$ 和 $(x,y)$ 必定是祖先后代关系（不妨设深度关系为 $y < x < v < u$），则这两条边中间的部分会被划分为一个三连通分量，那么在 $x$ 处异或一下，再在 $u$ 处异或一下即可。

复杂度大常数 $O(n)$，瓶颈在哈希表处。

注意一开始图就不连通的情况。

/// details | [模板题](https://www.luogu.com.cn/problem/P6658) 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(int i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(int i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using i64=long long;
using u64=unsigned long long;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=5e5+5,inf=0x3f3f3f3f;
int n,m;
pii ed[N];
vector<pii> e[N],son[N];
u64 vv[N],val[N],id[N],srv[N];
int vis[N],dpt[N],ptr[N];
mt19937_64 mr(20071221);
void dfs(int x,int fp,u64 rv){
    vis[x]=1;
    srv[x]=rv;
    for(auto i:e[x]){
        int v=i.fi,p=i.se;
        if(p==fp) continue;
        if(vis[v]){
            if(dpt[v]<dpt[x]){
                val[p]=mr();
                vv[x]^=val[p];vv[v]^=val[p];
                ptr[p]=1;
            }
            continue;
        }
        son[x].push_back(i);
        dpt[v]=dpt[x]+1;
        dfs(v,p,rv);
        val[p]=vv[v];
        vv[x]^=vv[v];
    }
}
unordered_map<u64,vector<int>> tr;
unordered_set<u64> ntr;
void dfs2(int x,int fa){
    vis[x]=1;
    id[x]^=id[fa];
    for(auto i:son[x]){
        dfs2(i.fi,x);
    }
}
vector<int> ans[N];
unordered_map<u64,int> mp;
int cntn;
signed main(){
    n=read();m=read();
    forup(i,1,m){
        int u=read(),v=read();
        if(u==v) continue;
        e[u].push_back(mkp(v,i));
        e[v].push_back(mkp(u,i));
        ed[i]=mkp(u,v);
    }
    forup(i,1,n){
        if(!vis[i]){
            dfs(i,0,mr());
        }
    }
    forup(i,1,m){
        if(!ed[i].fi) continue;
        int u=ed[i].fi,v=ed[i].se;
        if(dpt[u]<dpt[v]) swap(u,v);
        if(ptr[i]){
            if(tr.count(val[i])){
                for(auto p:tr[val[i]]){
                    u64 nw=mr();
                    id[p]^=nw;
                    msg("%d|\n",p);
                }
            }
            ntr.insert(val[i]);
        }else{
            if(val[i]==0){
                u64 nw=mr();
                id[u]^=nw;
                msg("%d|\n",u);
                continue;
            }
            if(tr.count(val[i])){
                for(auto p:tr[val[i]]){
                    msg("%d %d|\n",u,p);
                    u64 nw=mr();
                    id[u]^=nw;id[p]^=nw;
                }
            }
            if(ntr.count(val[i])){
                u64 nw=mr();
                id[u]^=nw;
                msg("%d|\n",u);
            }
            tr[val[i]].push_back(u);
        }
    }
    forup(i,1,n) vis[i]=0;
    forup(i,1,n){
        if(!vis[i]){
            dfs2(i,0);
        }
    }
    forup(i,1,n){
        id[i]^=srv[i];
        if(!mp.count(id[i])){
            mp[id[i]]=++cntn;
        }
        int p=mp[id[i]];
        ans[p].push_back(i);
    }
    printf("%d\n",cntn);
    forup(i,1,cntn){
        for(auto j:ans[i]){
            printf("%d ",j);
        }
        puts("");
    }
}
```

///


## 耳分解

对于一张无向图 $G=(V,E)$，对于一张子图 $G'=(V',E')$。若一条路径 $v_1,v_2,v_2,\dots v_k$，满足 $v_1,v_k\in V'$，且其余点互不相同均不在 $V'$ 内（注意 $v_1$ 可以等于 $v_k$），则称这条路径是一个“耳”（很形象吧），特别地，若 $v_1\ne v_k$，则称这条路径是一个开耳。

耳分解直观的定义就是将一张无向图分为若干个耳。形式化地，设一个由 $G$ 的子图构成的序列 $G_0,G_1,\dots G_k$，满足：

- $G_k=G$，且 $G_0$ 是一个简单环或一个点。
- $\forall 1\le i\le k$，$G_i$ 相对 $G_{i-1}$ 多了一个 $G_{i-1}$ 的耳，或者说 $G_i$ 是 $G_{i-1}$ 上加了一个耳得到的。

类似地也可以定义开耳分解，即每次必须加一个开耳。

### 耳分解的条件

一张无向图存在耳分解的充要条件是这张图是边双连通的。

/// admonition | 证明
    type: note

$\Rightarrow$：首先初始的一个简单环或者点肯定是边双连通的，后面每加一个耳并不会破坏边双连通性。

$\Leftarrow$：考虑构造证明：

- 找到边双连通图的一棵 dfs 生成树，随便找一条以 $1$ 为端点的非树边 $(1,x)$，将这条边围出的环当作 $G_0$。
- 每次找到一个 $u$ 满足 $u\notin G_i,fa_u\in G_i$，根据边双的性质，$u$ 的子树必定存在一个点 $p$ 使得有一条从 $p$ 开始的非树边 $p,q$ 连向 $fa_u$ 的祖先（或者 $fa_u$ 自己），不难发现这就是一个耳，将其加入即可。
- 将所有点加入后，剩下的边可以每个都当成一个独立的耳加入。

///

一张无向图存在开耳分解的充要条件是这张图是点双连通的。

证明也是类似，唯一的区别是找 $p\to q$ 这一步时必定能找到 $q$ 是 $fa_u$ 的严格祖先（即不包含 $fa_u$ 自己）。

直接按照这个构造来求并不是很好写，后文会提到好一点的构造方法。另外显然耳分解的方案很可能不是唯一的。

### 应用/例题

~~猜猜为什么叫不常用算法~~

//// details | [P5776](https://www.luogu.com.cn/problem/P5776) [SNOI2013] Quare
    open: True
    type: note

> 题意

- 给定一张 $n$ 个点 $m$ 条边无向连通图，边带权，可能有重边，没有自环。
- 求出这张图的一个边双连通子图，要求包含所有点，使得边权和最小，无解输出 `impossible`。
- $1\le n\le 12,1\le m\le 40$，带多测，边权的范围不明，但实测不用开 `long long`

> 题解

我们可以用耳分解来进行一些双连通分量相关的 DP 状态设计。

因为耳分解是边双连通分量的充要条件，那么不妨设 $dp_{msk}$ 表示点集 $msk$ 的答案。转移就枚举 $msk$ 的一个耳 $S$，然后从 $dp_{msk}$ 转移到 $dp_{msk|S}$。复杂度 $O(3^npoly(n))$。

然后因为我们需要枚举一个耳，不可避免地需要枚举子集，考虑能不能把枚举的耳扔到状态里面。

设 $dp_{msk,u,t}$ 表示已经经过了点集 $msk$，现在正在刻画一个耳，目标是走到点 $t$，已经走到了点 $u$。特别地，设 $dp_{msk,0,0}$ 表示并非正在刻画一个耳的状态。考虑一些边界情况后 DP 是简单的，具体转移略。

复杂度 $O(2^nn^3)$，可能能压掉一个 $n$ 吧。

/// details
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(int i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(int i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using i64=long long;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=15,inf=0x3f3f3f3f;
int n,m;
int grp[N][N],sec[N][N];
int dp[1<<12][N][N];
void solve(){
    n=read();m=read();
    mem(grp,0x3f);mem(sec,0x3f);
    forup(i,1,m){
        int u=read(),v=read(),w=read();
        if(w<grp[u][v]){
            sec[u][v]=sec[v][u]=grp[u][v];
            grp[u][v]=grp[v][u]=w;
        }else if(w<sec[u][v]){
            sec[u][v]=sec[v][u]=w;
        }
    }
    mem(dp,0x3f);
    forup(i,1,n){
        dp[(1<<(i-1))][0][0]=0;
    }
    forup(msk,1,(1<<n)-1){
        forup(u,1,n){
            if(!(msk&(1<<(u-1)))) continue;
            forup(t,1,n){
                if(t==u||!(msk&(1<<(t-1)))) continue;
                if(dp[msk][u][t]==inf) continue;
                forup(p,1,n){
                    if(p==u||grp[u][p]==inf) continue;
                    if(p==t){
                        dp[msk][0][0]=min(dp[msk][0][0],dp[msk][u][t]+grp[u][p]);
                    }else if(!(msk&(1<<(p-1)))){
                        int nx=msk|(1<<(p-1));
                        dp[nx][p][t]=min(dp[nx][p][t],dp[msk][u][t]+grp[u][p]);
                    }
                }
            }
        }
        if(dp[msk][0][0]!=inf){
            forup(u,1,n){
                if(!(msk&(1<<(u-1)))) continue;
                forup(v,1,n){
                    if(msk&(1<<(v-1))||sec[u][v]==inf) continue;
                    int nx=msk|(1<<(v-1));
                    dp[nx][0][0]=min(dp[nx][0][0],dp[msk][0][0]+grp[u][v]+sec[u][v]);
                }
                forup(v,1,n){
                    if(msk&(1<<(v-1))||grp[u][v]==inf) continue;
                    forup(p,1,n){
                        if(p==u||p==v||grp[v][p]==inf) continue;
                        if(msk&(1<<(p-1))){
                            int nx=msk|(1<<(v-1));
                            dp[nx][0][0]=min(dp[nx][0][0],dp[msk][0][0]+grp[u][v]+grp[v][p]);
                        }else{
                            forup(t,1,n){
                                if(!(msk&(1<<(t-1)))) continue;
                                int nx=msk|(1<<(v-1))|(1<<(p-1));
                                dp[nx][p][t]=min(dp[nx][p][t],dp[msk][0][0]+grp[u][v]+grp[v][p]);
                            }
                        }
                    }
                }
            }
        }
    }
    printf(dp[(1<<n)-1][0][0]==inf?"impossible\n":"%d\n",dp[(1<<n)-1][0][0]);
}
signed main(){
    int t=read();
    while(t--){
        solve();
    }
}
```

///
    
////

然后还能给一张边双连通图的边定向变成强连通图，每次加耳的时候随便定一个向即可，没有找到例题。

## 双极定向

双极定向，即对于一张无向图 $G=(V,E)$，给定源汇点 $s,t$，求出一个 DAG 使得 $s$ 能到达所有点，且所有点都能到达 $t$（等价于 $s$ 没有出度，$t$ 没有入度，其余所有点出入度均非 $0$）。

### 引理及证明

引理：下面四个结论等价：

1. 图 $G$ 加上边 $(s,t)$ 是点双连通的。
2. 在由点双连通分量求出的广义园方树中，可以找到一条链，使得其两端点分别是 $s,t$，并且包含所有方点。
3. 存在给所有边定向的方案，使得 $s$ 没有出度，$t$ 没有入度，其余所有点出入度均非 $0$（你会发现我们其实不关心是不是 DAG，因为肯定能反转一些边让它变成 DAG）。
4. 存在一个点的排列 $p_1,p_2,\dots p_n$，使得 $p_1=s,p_n=t$，且任意一个前缀和任意一个后缀均是连通的。

考虑这样证明 ~~只要若干个命题连成强连通分量就行了~~ ：

1. $2\Rightarrow 1$：

    显然，这条边会把覆盖的所有方点合并。

2. $1\Rightarrow 2$：

    若添加之前是点双则显然正确。

    若添加之前不是点双，那么把所有割点删掉后只有两个连通块，并且 $s,t$ 分别在两个连通块内，否则加上这条边后仍然由割点，就不是点双连通图了。

3. $1\Rightarrow 3$：

    在添加 $(s,t)$ 之前先求出以 $s$ 为根的 dfs 生成树，则 $(s,t)$ 必是非树边，那么我们将 $(s,t)$ 及其覆盖的树边作为 $G_0$，做开耳分解。

    首先 $G_0$ 是一个环，易于进行双极定向，后面每个耳也是易于定向的。

4. $3\Rightarrow 4$：

    由于可以双极定向为 DAG，那么我们任取一拓扑序，那么对于任意一 $p_i$，必定和 $p_{1}\sim p_{i-1}$ 有至少一条边，也和 $p_{i+1}\sim p_n$ 有至少一条边，则前后缀必定连通。

5. $4\Rightarrow 1$：

    最困难的一步，考虑反证法。

    假设 $1$ 不成立，那么必定存在一个割点 $u$，使得删掉它后 $s,t$ 在同一连通块内（和 $1\Rightarrow 2$ 的证明中类似），设其余点集为 $S$。

    我们找到由命题 $4$ 得到的序列 $p$，设 $S$ 中最小的点为 $p_x$，最大的为 $p_y$。由于 $u$ 是个割点，则必定 $p_x\ne p_y$，那么不等的那边就不可能和前/后缀连通，与假设矛盾。

### 构造方案

最暴力的方法是直接暴力构造耳分解然后做，大概 $O(n^2)$ 吧。

显然，如果一张图可以双极定向，那么把所有度数为 $2$ 的点（除去 $s,t$）缩掉并不影响连通性。

观察 dfs 树，一个点的出边无非是树边和返祖边，而只保留最浅的返祖边（$low_u$）是不影响连通性的，那么我们就可以依次把叶子缩掉了。最后会得到一条 $s\to t$ 的链，就很简单了。中间缩掉的点递归处理即可，这个做法比较难写。

一个较为好写的做法是，考虑每个叶子有两个邻居 $low_u$ 和 $fa_u$，那么每当访问到这两者其一时下一个都能访问 $u$，那么对每个点维护一个后继序列 $next_u$，表示访问这个点后下一个就可以访问的点。然后由深到浅枚举叶子，按顺序加入 $next_{low_u}$ 和 $next_{fa_u}$ 即可，最后还是剩 $s\to t$ 的链，那么遍历过去，每访问到一个点就直接按深度优先的顺序访问它的后继序列（然后你会发现其实和递归做法差不多）。不难发现 $t$ 必然是最后遍历的（因为 $t$ 不是割点），这样就能保证 $s$ 没有入度，$t$ 没有出度，并且中间每个点都既有入度又有出度了。

据说这个方法改一下还能求耳分解，但是我没有试过，一般也不会有题专门喊你求耳分解吧。

### 例题

//// details | [P9394](https://www.luogu.com.cn/problem/P9394) 白鹭兰
    open: True
    type: example

> 题意

- 给定一张 $n$ 个点 $m$ 条边的无向连通图。
- 将其划分为若干个两两不交，且并集为全集的的集合 $S_1,S_2\dots S_p$，使得 $\forall 1\le x\le p$，$\bigcup_{i=1}^x S_i$ 和 $\bigcup_{i=k}^x S_i$ 都是连通的。
- 最小化最大集合的大小 $k$，输出方案。
- $1\le n,m\le 2.3\times 10^5$

> 题解

不难想到双极定向的结论。显然只有所有点双连成一条链时 $k$ 可以等于 $1$。

否则，我们找到 $S_1,S_p$（不妨各取一个代表点 $x,y$）所在的点双在圆方树上的简单路径，显然路径上的每个分叉都必须完全放在一个 $S_i$ 里面，否则就必定会让某个前缀/后缀不连通。

更准确地说，对于路径上的原点，它子集必须和它的分叉放在一起。对于方点，它的所有分叉必须放在一起。$k$ 即这些点集大小的最小值。

那么如何得到最小的 $k$ 呢？不难想到一个贪心。我们先假设 $y$ 是圆方树的根，考虑从 $x$ 开始到 $y$ 的链是 $x_0,x_1,\dots x_p$，对于每个 $i < p$，$x_i$ 都是 $x_{i+1}$ 的最大子树，否则显然不优，具体证明从 $x_{i+1}$ 是圆点还是方点分别考虑就行了，此处不展开。

那么设 $f_u$ 表示以 $u$ 为最顶上结点，链延伸到叶子的最小答案，统计答案就对于每个点假设它是 $x\to y$ 链的 $\mathrm{lca}$，将最大子树 $p$ 和次大子树 $q$ 的 $f$ 贡献 $\max(f_p,f_q,g_u)$ 即可，这里 $g_u$ 是点 $u$ 自己作为 $\mathrm{lca}$ 的贡献，注意特判一些边界情况。

这样我们就知道了 $S_1,S_p$，那么我们就能知道要把哪些点放在一起了，将它们缩成一个点后就变成了双极定向板题了。复杂度 $O(n)$。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(int i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(int i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using i64=long long;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=230005,inf=0x3f3f3f3f;
int n,m,cntn;
vector<vector<int>> res;
vector<int> e1[N],e2[N*2];
namespace Tarjan{
    int dfn[N],low[N],Tm;
    stack<int> stk;
    void Tarjan(int x){
        dfn[x]=low[x]=++Tm;
        stk.push(x);
        for(auto i:e1[x]){
            if(!dfn[i]){
                Tarjan(i);
                low[x]=min(low[x],low[i]);
                if(dfn[x]==low[i]){
                    int nw=++cntn;
                    for(int v=0;v!=i;stk.pop()){
                        v=stk.top();
                        e2[nw].push_back(v);
                        e2[v].push_back(nw);
                        msg("(%d %d)\n",nw,v);
                    }
                    e2[nw].push_back(x);
                    e2[x].push_back(nw);
                    msg("(%d %d)\n",nw,x);
                }
            }else{
                low[x]=min(low[x],dfn[i]);
            }
        }
    }    
}
int ans=inf,rt1=0,rt2=0;
int sz[N*2],f[N*2],ed[N*2];
void dfs(int x,int fa){
    sz[x]=(x<=n);
    int mx=0,sec=0;
    ed[x]=x;
    for(auto i:e2[x]){
        if(i==fa) continue;
        dfs(i,x);
        sz[x]+=sz[i];
        if(sz[i]>sz[mx]){
            sec=mx;mx=i;
        }else if(sz[i]>sz[sec]){
            sec=i;
        }
    }
    int val=0,g=0,tr=0;
    if(x<=n){
        g=sz[x]-sz[mx]-sz[sec]+(n-sz[x]);
        tr=sz[x]-sz[mx];
    }else{
        for(auto i:e2[x]){
            if(i==mx||i==sec||i==fa) continue;
            g=max(g,sz[i]);
            tr=max(tr,sz[i]);
        }
        g=max(g,n-sz[x]);
        tr=max(tr,sz[sec]);
    }
    f[x]=tr;
    if(mx){
        f[x]=max(f[mx],f[x]);
        ed[x]=ed[mx];
        val=max(val,f[mx]);
    }
    if(sec) val=max(val,f[sec]);
    val=max(val,g);
    if(val<=ans){
        ans=val;
        if(mx){
            rt1=ed[mx];
        }else{
            rt1=x;
        }
        if(sec){
            rt2=ed[sec];
        }else{
            rt2=x;
        }
    }
}
int ff[N*2],dfn[N*2],num[N*2],mp[N*2],Tm,inseq[N*2];
void dfs2(int x,int fa){
    ff[x]=fa;
    dfn[x]=++Tm;mp[dfn[x]]=x;
    num[x]=1;
    for(auto i:e2[x]){
        if(i==fa) continue;
        dfs2(i,x);
        num[x]+=num[i];
    }
}
int co[N],vis[N],dpt[N],tp[N];
vector<int> nxt[N];
int f2[N],low[N],mxd;
vector<int> seq[N];
void dfs3(int x,int fa){
    dpt[x]=dpt[fa]+1;
    vis[x]=1;f2[x]=fa;
    low[x]=0;
    for(auto i:e1[x]){
        if(i==fa||!co[i]) continue;
        if(vis[i]){
            if(dpt[i]<dpt[x]){
                if(!low[x]||dpt[i]<dpt[low[x]]){
                    low[x]=i;
                }
            }
            continue;
        }
        dfs3(i,x);
        if(!low[x]||dpt[low[i]]<dpt[low[x]]) low[x]=low[i];
    }
    seq[dpt[x]].push_back(x);
    mxd=max(mxd,dpt[x]);
}
void gans(int x){
    vector<int> vec;
    if(!tp[x]){
        for(int i=dfn[x];i<dfn[x]+num[x];++i){
            if(mp[i]<=n){
                vec.push_back(mp[i]);
            }
        }
    }else{
        vec.push_back(x);
        for(auto u:e2[x]){
            if(inseq[u]) continue;
            for(int i=dfn[u];i<dfn[u]+num[u];++i){
                if(mp[i]<=n){
                    vec.push_back(mp[i]);
                }
            }
        }
    }
    res.push_back(vec);
}
void dfs4(int x){
    vis[x]=1;
    gans(x);
    for(auto i:nxt[x]){
        if(vis[i]) continue;
        dfs4(i);
    }
    nxt[x].clear();
}
void work(int pn,int s,int t){
    msg("{%d %d %d}\n",pn,s,t);
    for(auto i:e2[pn]){
        co[i]=1;vis[i]=0;
    }
    mxd=0;
    dfs3(s,0);
    vector<int> vec;
    for(int i=t;i!=s;i=f2[i]){
        vis[i]=0;
        vec.push_back(i);
    }
    vis[s]=0;
    vec.push_back(s);
    reverse(vec.begin(),vec.end());
    fordown(i,mxd,1){
        for(auto j:seq[i]){
            if(!vis[j]) continue;
            vis[j]=0;
            msg("%d|%d %d|\n",j,f2[j],low[j]);
            nxt[f2[j]].push_back(j);
            if(low[j]) nxt[low[j]].push_back(j);
        }
        seq[i].clear();
    }
    for(auto u:vec){
        vis[u]=1;
    }
    for(auto u:vec){
        msg("(%d)",u);
        if(u!=s){
            dfs4(u);
        }else{
            for(auto i:nxt[u]){
                if(vis[i]) continue;
                dfs4(i);
            }
            nxt[u].clear();
        }
    }
    msg("|\n");
    for(auto i:e2[pn]){
        co[i]=0;
    }
}
// namespace CHK{
//     int vis[N],v2[N];
//     void dfs(int x){
//         v2[x]=1;
//         for(auto i:e1[x]){
//             if(!vis[i]||v2[i]) continue;
//             dfs(i);
//         }
//     }
//     bool lnk(){
//         int cnt=0;
//         mem(v2,0);
//         forup(i,1,n){
//             if(v2[i]!=vis[i]){
//                 dfs(i);
//                 ++cnt;
//             }
//         }
//         return cnt<=1;
//     }
//     void chk(){
//         forup(i,0,res.size()-1){
//             for(auto j:res[i]){
//                 assert(!vis[j]);
//                 vis[j]=1;
//             }
//             assert(lnk());
//         }
//         forup(i,1,n){
//             assert(vis[i]);
//         }
//         mem(vis,0);
//         fordown(i,res.size()-1,0){
//             for(auto j:res[i]){
//                 assert(!vis[j]);
//                 vis[j]=1;
//             }
//             assert(lnk());
//         }
//         msg("AC");
//     }    
// }
signed main(){
    n=read();m=read();
    forup(i,1,m){
        int u=read(),v=read();
        e1[u].push_back(v);
        e1[v].push_back(u);
    }
    cntn=n;
    Tarjan::Tarjan(1);
    dfs(1,0);
    msg("%d %d|\n",rt1,rt2);
    dfs2(rt1,0);
    res.push_back(vector<int>(1,rt2));
    for(int u=rt2;u;u=ff[u]){
        msg("%d ",u);
        inseq[u]=1;
        if(u<=n){
            tp[u]=1;
        }
    }
    msg("|\n");
    for(int p=0,u=rt2;u;p=u,u=ff[u]){
        if(u>n){
            work(u,p,ff[u]);
        }
    }
    printf("%d %d\n",ans,(int)res.size());
    for(auto i:res){
        printf("%d ",(int)i.size());
        for(auto j:i){
            printf("%d ",j);
        }
        puts("");
    }
    // CHK::chk();
}
```

///

////