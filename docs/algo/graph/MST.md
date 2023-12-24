---
comments: true
---

# 最小生成树与 Kruskal 重构树

## 前言

改完错就开始卷算法了。

## 引入

首先，生成树的定义是很自然的。

很多时候我们会使用搜索算法遍历某一张无向图上的 $n$ 个点。那么为了遍历这些点我们会经过 $n-1$ 条边（若图连通）。而稍微有点图论直觉的人都会发现，$G=(V,E),|V|=n,|E|=n-1$ 这样的定义很容易让我们联想到树，那么生成树的定义就呼之欲出了。

用数学语言就是：若 $G'=(V,E'),G=(V,E),E'\subseteq E$，且 $G'$ 是一棵树。那么称 $G'$ 为 $G$ 的生成树（**Spanning Tree**）。

而显然，每个连通图必然有生成树，考虑有环就断掉，最后必然是无环连通图。

而最小生成树（MST，**Minimum Spanning Tree**）即图 $G$ 的所有生成树中边权和**最小**的。容易发现最小生成树**不唯一**。

## 求解

### Prim 算法

Prim 的思路和求解最短路的 dijkstra 算法类似，把图分为两个集合 $S,T$，$S$ 最初为任意一个点 $u$，$T$ 最初为 $V\setminus \begin{Bmatrix}u\end{Bmatrix}$，然后每次从 $T$ 集合中取出距离最小的点加入 $S$ 集合，经过的这条边就是树边。

代码很简单，就不贴了。

/// admonition | 正确性证明
    type: quote

考虑说明在每一步，都存在一棵最小生成树包含已选边集。

首先只有一个结点的时候，显然成立。

然后考虑归纳证明，如果某一步成立，当前边集为 $F$，属于 $T$ 这棵 MST，接下来要加入边 $e$。

如果 $e$ 属于 $T$，那么成立。

否则考虑 $T+e$ 中环上另一条可以加入当前边集的边 $f$。

首先，$f$ 的权值一定不小于 $e$ 的权值，否则就会选择 $f$ 而不是 $e$ 了。

然后，$f$ 的权值一定不大于 $e$ 的权值，否则 $T+e-f$ 就是一棵更小的生成树了。

因此，$e$ 和 $f$ 的权值相等，$T+e-f$ 也是一棵最小生成树，且包含了当前边集 $F$。

摘自 [OI Wiki](https://oi.wiki/graph/mst/#%E8%AF%81%E6%98%8E_1)

///

直接暴力复杂度是 $O(n^2+m)$ 的，若使用优先队列复杂度是 $O((n+m)\log m)$ 的。如果使用斐波那契堆复杂度是 $O(n\log n+m)$ 的，但是代码会比较难写。

### Kruskal 算法

与 Prim 不同，Kruskal 算法的核心思想在于加边而不是加点。具体来说，我们从小到大考虑每条边。若不会形成环就加入 MST，否则跳过。判定会不会形成环可以使用并查集维护，把每个连通块维护为一个集合即可。

代码也不贴了。

/// admonition | 正确性证明
    type: quote

仍然考虑说明在每一步，都存在一棵最小生成树包含已选边集。

考虑使用归纳法，证明任何时候算法选择的边集都被某棵 MST 所包含。

对于算法刚开始时，显然成立（最小生成树存在，边集为空集）。

假设某时刻成立，当前边集为 $F$，令 $T$ 为这棵 MST，考虑下一条加入的边 $e$。

如果 $e$ 属于 $T$，那么成立。

否则，$T+e$ 一定存在一个环，考虑这个环上不属于 $F$ 的另一条边 $f$（一定只有一条）。

首先，$f$ 的权值一定不会比 $e$ 小，不然 $f$ 会在 $e$ 之前被选取。

然后，$f$ 的权值一定不会比 $e$ 大，不然 $T+e-f$ 就是一棵比 $T$ 还优的生成树了。

所以，$T+e-f$ 包含了 $F$，并且也是一棵最小生成树，归纳成立。

摘自 [OI Wiki](https://oi.wiki/graph/mst/#%E8%AF%81%E6%98%8E_1)

///

若排序的复杂度是 $O(m\log m)$，并查集使用路径压缩+按秩合并，复杂度为 $O(m\alpha(n))$，那么总复杂度就是 $O(m\log m)$，在相同的代码难度（优先队列写法）下复杂度优于 Prim。且由于常数问题在大部分图下使用斐波那契堆的 Prim 也不一定能跑过 Kruskal。

另外，上述两个算法也可以求解**最大生成树**问题，证明略。

## 瓶颈路

那么最小生成树有什么用呢？

考虑这样一个问题，某地区的路径规划为一张无向图，每条边有一个限制 $w$，仅有权限大于 $w$ 的车才能经过，问权限为 $h$ 的车能否从 $A$ 地到 $B$ 地。

容易发现，这个存在性只与 $AB$ 两地间限制**最大值最小**的路径有关。我们把这条路称为瓶颈路。

那么类似的，我们可以定义瓶颈生成树，即 $G$ 所有生成树中边权最大值最小的。

容易发现，瓶颈生成树上的简单路径是瓶颈路的充分不必要条件。证明略。

另外还有一个结论：最小生成树是瓶颈生成树的充分不必要条件。

/// admonition | 证明
    type: quote

考虑使用反证法。

我们设最小生成树中的最大边权为 $w$，如果最小生成树不是瓶颈生成树的话，则瓶颈生成树的所有边权都小于 $w$，我们只需删去原最小生成树中的最长边，用瓶颈生成树中的一条边来连接删去边后形成的两棵树，得到的新生成树一定比原最小生成树的权值和还要小，这样就产生了矛盾。

///

很多时候，树上信息会比图上信息更好维护，那么就可以用最小生成树算法把一般图上问题转化为树上问题求解。

例题不太能找到。我现在只能想到一道校内 OJ 的题目。

## Kruskal 重构树

但是有些时候，只转化为一棵生成树上的问题还是不好维护。怎么办呢？这时候我们就可以考虑 Kruskal 重构树算法了。

### 算法简述

Kruskal 重构树算法是基于 Kruskal 算法得到的。

具体来说，我们在每次合并集合的时候，新建一个结点表示当前合并的这条边。它的点权就是原来边的边权，左右儿子就是当前正在合并的两集合原来的根。

容易发现，这样构造出来的二叉树上，所有叶子结点均是原图上的点，其余结点代表最小生成树上的边。

那我们来分析一下这棵树 $T$ 有什么性质。

首先，它把**边的信息**转化为了**点的信息**。容易发现除去叶子结点，每个点的点权均大于等于其子树内所有点的点权（考虑构造过程），也就是说，从叶子结点到根的路径上的点权是**单调不降**的。

另外，容易发现原图 $G$ 中 $A,B$ 间瓶颈路上的最小值正是 $T$ 上 $\operatorname{lca}(A,B)$ 的点权。仍然考虑树的构造过程，这是显然的。

/// details | 参考实现
    open: False

```cpp
struct Node{
	i64 u,v,w;
}a[N<<1];
bool cmp(Node a,Node b){
	return a.w<b.w;
}
i64 fa[N],rt[N];
i64 val[N<<1],ans[N<<1],son[N<<1][2];
void kruskal(){
	sort(a+1,a+m+1,cmp);
	forup(i,1,n){
		fa[i]=rt[i]=i;
		val[i]=inf;
		ans[i]=dis[i];
		son[i][0]=son[i][1]=0;
	}
	int cntn=n;
	forup(i,1,m){
		i64 u=a[i].u,v=a[i].v,w=a[i].w;
		i64 fu=getfa(u),fv=getfa(v);
		if(fu==fv) continue;
		++cntn;
		son[cntn][0]=rt[fu];son[cntn][1]=rt[fv];
		val[cntn]=w;ans[cntn]=min(ans[son[cntn][0]],ans[son[cntn][1]]);
		fa[fu]=fv;rt[fv]=cntn;
	}
}
```

///

#### 例题

//// details | P4768 [NOI2018] 归程

[传送门](https://www.luogu.com.cn/problem/P4768)

首先题意可以转化为每次求点 $v$ 只经过点权大于 $p$ 的边能到达的所有结点中，与结点 $1$ 间最短路最小是多少。

我们之前说过，Kruskal 是可以跑最大生成树的，且最大生成树也具有类似性质。

那么设这样构造的 Kruskal 重构树为 $T$。根据上面的结论，容易发现 $v$ 必然存在一个祖先 $f$（可能是它自己），$a_f>p,a_{father(f)}\le p$，那么 $f$ 子树内所有叶子结点就是从 $v$ 出发可达的结点，因为它们和 $v$ 的 $\operatorname{lca}$ 的点权显然均小于等于 $f$。

那么我们可以先跑一遍以 $1$ 为起点的单源最短路，然后对每个点维护子树内叶子结点最短路最小值，最后在线倍增找到这个 $f$ 即可。复杂度 $O(m\log m+q\log m)$。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(i64 i=(s);i<=(e);i++)
#define fordown(i,s,e) for(i64 i=(s);i>=(e);i--)
using namespace std;
using i64=long long;
#define gc getchar()
inline i64 read(){
    i64 x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const i64 N=2e5+5,inf=1e18;
i64 t,n,m,q,tp,s,lans;
struct Node{
	i64 u,v,w;
}a[N<<1];
struct edge{
	i64 v,w;
};
vector<edge> e[N];
i64 fa[N],rt[N];
i64 getfa(i64 x){return x==fa[x]?x:fa[x]=getfa(fa[x]);}
struct node{
	i64 u,dis;
	bool operator <(const node &r)const{return dis>r.dis;}
};
priority_queue<node> qq;
i64 vis[N],dis[N];
i64 val[N<<1],ans[N<<1],son[N<<1][2];
i64 f[N<<1][21];
void dfs(int x){
	if(x==0) return;
	forup(i,1,20){
		f[x][i]=f[f[x][i-1]][i-1];
	}
	f[son[x][0]][0]=f[son[x][1]][0]=x;
	dfs(son[x][0]);dfs(son[x][1]);
}
bool cmp(Node a,Node b){
	return a.w>b.w;
}
void kruskal(){
	sort(a+1,a+m+1,cmp);
	forup(i,1,n){
		fa[i]=rt[i]=i;
		val[i]=inf;
		ans[i]=dis[i];
		son[i][0]=son[i][1]=0;
	}
	int cntn=n;
	forup(i,1,m){
		i64 u=a[i].u,v=a[i].v,w=a[i].w;
		i64 fu=getfa(u),fv=getfa(v);
		if(fu==fv) continue;
		++cntn;
		son[cntn][0]=rt[fu];son[cntn][1]=rt[fv];
		val[cntn]=w;ans[cntn]=min(ans[son[cntn][0]],ans[son[cntn][1]]);
		fa[fu]=fv;rt[fv]=cntn;
	}
	forup(i,0,cntn){
		forup(j,0,20){
			f[i][j]=0;
		}
	}
	dfs(cntn);
}
void dijkstra(){
	while(qq.size()) qq.pop();
	forup(i,1,n){
		dis[i]=inf;vis[i]=0;
	}
	dis[1]=0;qq.push(node{1,0});
	while(qq.size()){
		i64 u=qq.top().u;qq.pop();
		if(vis[u]) continue;
		vis[u]=1;
		for(auto i:e[u]){
			i64 v=i.v,w=i.w;
			if(dis[u]+w<dis[v]){
				dis[v]=dis[u]+w;
				qq.push(node{v,dis[v]});
			}
		}
	}
}
signed main(){
	t=read();
	while(t--){
		n=read();m=read();
		forup(i,1,n){
			e[i].clear();
		}
		forup(i,1,m){
			i64 u=read(),v=read(),len=read(),w=read();
			a[i].u=u;a[i].v=v;a[i].w=w;
			e[u].push_back(edge{v,len});
			e[v].push_back(edge{u,len});
		}
		dijkstra();
		kruskal();
		q=read();tp=read();s=read();lans=0;
		forup(Case,1,q){
			int x=read(),p=read();
			x=(x+tp*lans-1)%n+1;
			p=(p+tp*lans)%(s+1);
			int l=x;
			fordown(i,20,0){
				if(f[l][i]&&val[f[l][i]]>p) l=f[l][i];
			}
			lans=ans[l];
			printf("%lld\n",ans[l]);
		}
	}
}
```

~~容易发现上面的参考实现是从这里复制过去的。~~

///

////


### 基于点的 Kruskal 重构树

还有这么一类问题，就是只能经过点权小于/大于某值的点，然后要查询连通块信息。这个当然可以把 $(u,v)$ 的边权赋成 $\max(w_u,w_v)$ 然后直接跑 Kruskal 重构树。但其实还有一个更简洁的方法。

具体来说，先将点权从小到大排序，然后每加入一个点就将它所有**指向的点已加入重构树**的出边指向的连通块作为它的儿子。容易发现这样构造出来结构仍类似于 Kruskal 重构树，并且只有 $n$ 个点。暂未找到例题。

## 其余典型问题

### 黑白生成树

接下来讨论一类最小生成树的常见问题：黑白最小生成树。

这类问题的设问大概是每条边除了边权以外还有一个黑白颜色，问恰好有 $k$ 条白色边的最小生成树。这个的常见做法是 wqs 二分。

然后 wqs 二分可以参考我的[这篇博客]()

### 最小生成树数量

这个比较考验对 kruskal 的理解。

就是 kruskal 排序后显然会是一段一段的边权相同。然后容易发现加了一段之后图的连通性是固定的，而每一段具体加边方式是不固定的，只需要加出来是一个生成森林即可。相当于每次取出一整段，然后随便连一个生成森林，再把每个连通块缩成一个点，进入下一段。而缩点的方式是唯一的，即每一段独立。可以对每一段用矩阵树定理求出生成森林数量，然后用乘法原理乘出来。