---
comments: true
---

# 网络流合集

## 最大流与二分图匹配

### P3163 [CQOI2014] 危桥

[传送门](https://www.luogu.com.cn/problem/P3163)

妙题。

第一眼：乂？这不是建个超级源点超级汇点直接做就行了吗（就把一次往返视为只经过一次）。

然后发现不对，因为会有 $a_1\to b_2$ 和 $a_2\to b_1$ 的流，还有无向边两个方向都走一遍的情况。

怎么办呢？考虑流的组成部分，设 $f(u,v)$ 表示从 $u$ 到 $v$ 的流量和是否为 $a_n+b_n$，称流量和为 $a_n+b_n$ 为“合法”。

则求出来的流是 $f(a_1,a_2)+f(a_1,b_2)+f(b_1,b_2)+f(b_1,a_2)$。

这时候怎么办呢？考虑有没有什么办法在不改变 $f(a_1,a_2)$ 和 $f(b_1,b_2)$ 的情况下改变剩下两项。一个想法是交换 $b_1,b_2$，即把 $b_2$ 作为源点，$b_1$ 作为汇点，此时流变为 $f(a_1,a_2)+f(a_1,b_1)+f(b_1,b_2)+f(b_2,a_2)$。由于是无向边，那么若 $f(a_1,a_2)$ 与 $f(b_1,b_2)$ 没有同时经过某一条边的两个方向 $f(b_2,b_1)$ 完全可以直接把原先的流反过来，故已经排除了反向边的情况。

接下来考虑串流的情况。若这个也合法，那么有两种情况：

1. $f(a_1,b_2)=f(a_2,b_1)=f(a_1,b_1)=f(b_2,a_2)=0$

显然合法。

2. $f(a_1,b_2)+f(b_1,a_2)=f(a_1,b_1)+f(b_2,a_2)$

由于可以构造出 $f(a_1,a_2)$ 不变，$f(b_1,b_2)$ 反向的情况，那么此时上面四个都相等。

那么有 $f(a_1,b_2)=f(a_1,b_1)$，且不影响 $f(a_1,a_2)$ 与 $f(b_1,b_2)$ 的流。此时可以将 $f(a_1,b_2)$ 反向加在 $f(b_1,b_2)$ 的流里面。

那么就以 $a_1,b_1$ 为源点，$a_1,b_2$ 为源点分别跑一遍即可。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=70,inf=0x3f3f3f3f;
int n,a1,a2,ca,b1,b2,cb;
struct FLOW{
	struct edge{
		int v,rst,nxt;
	}e[N*N*2];
	int head[N],cur[N],dpt[N],cnte,s,t;
	void adde(int u,int v,int w){
		e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,w,head[v]};head[v]=cnte;
	}
	void clear(){
		cnte=1;mem(head,0);
		s=n+1;t=n+2;
	}
	bool bfs(){
		forup(i,1,t){
			cur[i]=head[i];
			dpt[i]=-1;
		}
		queue<int> q;
		q.push(s);dpt[s]=0;
		while(q.size()){
			int u=q.front();q.pop();
			for(int i=head[u];i;i=e[i].nxt){
				if(!e[i].rst) continue;
				int v=e[i].v;
				if(dpt[v]==-1){
					dpt[v]=dpt[u]+1;
					q.push(v);
				}
			}
		}
		return dpt[t]!=-1;
	}
	int dfs(int x,int flow){
		if(x==t||!flow) return flow;
		int res=0;
		for(int i=cur[x];i;i=e[i].nxt){
			cur[x]=i;int v=e[i].v;
			if(dpt[v]!=dpt[x]+1) continue;
			int gt=dfs(v,min(flow-res,e[i].rst));
			if(gt){
				res+=gt;
				e[i].rst-=gt;
				e[i^1].rst+=gt;
				if(res==flow) break;
			}
		}
		return res;
	}
	int dinic(){
		int res=0;
		while(bfs()){
			res+=dfs(s,inf);
		}
		return res;
	}
}flow;
char grp[N][N];
void solve(){
	a1=read()+1;a2=read()+1;ca=read();b1=read()+1;b2=read()+1;cb=read();
	flow.clear();
	forup(i,1,n){
		scanf(" %s",grp[i]+1);
	}
	forup(i,1,n){
		forup(j,i+1,n){
			if(grp[i][j]=='N'){
				flow.adde(i,j,inf);
			}else if(grp[i][j]=='O'){
				flow.adde(i,j,1);
			}
		}
	}
	flow.adde(flow.s,a1,ca);flow.adde(flow.s,b1,cb);
	flow.adde(a2,flow.t,ca);flow.adde(b2,flow.t,cb);
	int ans=flow.dinic();
	if(ans!=ca+cb){
		puts("No");
		return;
	}
	flow.clear();
	forup(i,1,n){
		forup(j,i+1,n){
			if(grp[i][j]=='N'){
				flow.adde(i,j,inf);
			}else if(grp[i][j]=='O'){
				flow.adde(i,j,1);
			}
		}
	}
	flow.adde(flow.s,a1,ca);flow.adde(flow.s,b2,cb);
	flow.adde(a2,flow.t,ca);flow.adde(b1,flow.t,cb);
	ans=flow.dinic();
	if(ans!=ca+cb){
		puts("No");
		return;
	}
	puts("Yes");
}
signed main(){
	while(scanf(" %d",&n)!=EOF){
		solve();
	}
}
```

///


### CF793G Oleg and chess

[传送门](https://www.luogu.com.cn/problem/CF793G)

简单题，但是一直想歪了，一直在想怎么优化建模。

容易发现如果规模比较小就是简单的二分图匹配。但是这道题 $n$ 很大（算是比较大吧），怎么办呢？

容易发现大部分连边都是某矩阵内横向所有点连向纵向所有点（即区间连区间），容易想到线段树优化建图，每次新建一个点作为中间点即可。找矩形可以用 ODT 之类的随便扫描线维护一下。

点数是 $O(n)$ 的，边数是 $O(n\log n)$ 的，dinic 随便过。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
#define y1 y114514
using namespace std;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=1<<14,inf=0x3f3f3f3f;
int n,m,cntn;
struct FLOW{
	struct edge{
		int v,rst,nxt;
	}e[N*80];
	int head[N*5],cur[N*5],cnte,s,t,dpt[N*5];
	void adde(int u,int v,int w){
		e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,0,head[v]};head[v]=cnte;
	}
	bool bfs(){
		forup(i,1,t){
			cur[i]=head[i];
			dpt[i]=-1;
		}
		dpt[s]=0;
		queue<int> q;q.push(s);
		while(q.size()){
			int u=q.front();q.pop();
			for(int i=head[u];i;i=e[i].nxt){
				if(!e[i].rst) continue;
				int v=e[i].v;
				if(dpt[v]==-1){
					dpt[v]=dpt[u]+1;
					q.push(v);
				}
			}
		}
		return dpt[t]!=-1;
	}
	int dfs(int x,int flow){
		if(x==t||!flow) return flow;
		int res=0;
		for(int i=cur[x];i;i=e[i].nxt){
			cur[x]=i;int v=e[i].v;
			if(!e[i].rst||dpt[v]!=dpt[x]+1) continue;
			int gt=dfs(v,min(e[i].rst,flow-res));
			if(gt){
				e[i].rst-=gt;
				e[i^1].rst+=gt;
				res+=gt;
				if(res==flow) break;
			}
		}
		return res;
	}
	int dinic(){
		int ans=0;
		while(bfs()){
			ans+=dfs(s,inf);
		}
		return ans;
	}
}flow;
struct range{
	int l,r,op;
};
vector<range> add[N],era[N];
struct SegTree{
	int node[N<<1];
	void Build(int flag){
		forup(i,1,n) node[i+N]=i+flag*n;
		fordown(i,N-1,1){
			if(!node[i<<1]&&!node[i<<1|1]) continue;
			node[i]=++cntn;
			if(flag){
				if(node[i<<1]) flow.adde(node[i],node[i<<1],inf);
				if(node[i<<1|1]) flow.adde(node[i],node[i<<1|1],inf);
			}else{
				if(node[i<<1]) flow.adde(node[i<<1],node[i],inf);
				if(node[i<<1|1]) flow.adde(node[i<<1|1],node[i],inf);
			}
		}
	}
	void Link(int l,int r,int X,int flag){
		if(flag){
			for(l+=N-1,r+=N+1;l^r^1;l>>=1,r>>=1){
				if(!(l&1)) flow.adde(X,node[l^1],inf);
				if(  r&1 ) flow.adde(X,node[r^1],inf);
			}
		}else{
			for(l+=N-1,r+=N+1;l^r^1;l>>=1,r>>=1){
				if(!(l&1)) flow.adde(node[l^1],X,inf);
				if(  r&1 ) flow.adde(node[r^1],X,inf);
			}
		}
	}
}t1,t2;
map<pii,int> odt;
using mit=map<pii,int>::iterator;
mit split(int x){
	mit it=odt.lower_bound(mkp(x,0));
	if(it->fi.fi==x) return it;
	--it;
	int l=it->fi.fi,r=it->fi.se,c=it->se;
	odt.erase(it);
	odt.insert(mkp(mkp(l,x-1),c));
	if(x<=r){
		return odt.insert(mkp(mkp(x,r),c)).fi;
	}else{
		return odt.end();
	}
}
signed main(){
	n=read();m=read();
	flow.cnte=1;
	cntn=n*2;
	t1.Build(1);t2.Build(0);
	forup(i,1,m){
		int x1=read(),y1=read(),x2=read(),y2=read();
		add[x1].push_back(range{y1,y2,0});
		era[x2+1].push_back(range{y1,y2,1});
	}
	odt.insert(mkp(mkp(1,n),1));
	forup(i,1,n){
		for(auto j:era[i]){
			mit ed=split(j.r+1),st=split(j.l);
			odt.erase(st,ed);
			odt.insert(mkp(mkp(j.l,j.r),i));
		}
		for(auto j:add[i]){
			mit ed=split(j.r+1),st=split(j.l);
			for(mit it=st;it!=ed;odt.erase(prev(++it))){
				if(it->se<=i-1){
					int nw=++cntn;
					t1.Link(it->fi.fi,it->fi.se,nw,1);
					t2.Link(it->se,i-1,nw,0);
				}
			}
			odt.insert(mkp(mkp(j.l,j.r),n+1));
		}
	}
	for(auto j:odt){
		if(j.se<=n){
			int nw=++cntn;
			t1.Link(j.fi.fi,j.fi.se,nw,1);
			t2.Link(j.se,n,nw,0);
		}
	}
	flow.s=++cntn;flow.t=++cntn;
	forup(i,1,n){
		flow.adde(flow.s,i,1);flow.adde(i+n,flow.t,1);
	}
	printf("%d\n",flow.dinic());
}
```

///


### CF590E Birthday

[传送门](https://www.luogu.com.cn/problem/CF590E)

第一眼感觉巨大困难，过了一个小时发现会了，写完发现 dinic 时空被卡了，最后用匈牙利算法解决。

首先由于子串关系可以形成一个偏序集，那么这道题就是要求最长反链。

容易想到用 AC 自动机求子串关系，但是在 AC 自动机上暴力跳显然复杂度是错的。容易想到每个点只连 $fail$ 树上最近的**是某字符串结尾**的祖先（这个可以在建自动机时维护），然后对这样求出来的 DAG 求传递闭包就能得到我们想连的图了。

根据最长反链等于最小链覆盖，可以二分图匹配求出最小链覆盖。然后考虑在每条链上选一个点作为答案。但是不能随便选，因为可能形成偏序关系，怎么办呢？

其实很简单，首先找到每条链的末尾（即左部点中没连匹配边的），然后每次随便找一个不合法的，接着往前找该链第一个合法的即可（这个每次跳对应右部点的匹配点就是链上的上一个点）。根据最长反链等于最小链覆盖这样必定是能求出解的，并且由于每个点只会被经过一次所以复杂度 $O(n)$。

友情提示：dinic 时空会被卡，请使用匈牙利算法。

总复杂度 $O(\sum|s_i|+\frac{n^3}{w})$。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=755,M=1e7+5,inf=0x3f3f3f3f;
int n;
char str[M];
vector<int> pth[N];
int tr[M][2],fail[M],cntn,lst[M];
void Insert(int n,int num){
	int p=0;
	forup(i,1,n){
		int c=str[i]-'a';
		if(!tr[p][c]) tr[p][c]=++cntn;
		p=tr[p][c];
		pth[num].push_back(p);
	}
	lst[p]=num;
}
void Build(){
	queue<int> q;
	forup(i,0,1){
		if(tr[0][i]) q.push(tr[0][i]);
	}
	while(q.size()){
		int u=q.front();q.pop();
		if(!lst[u]) lst[u]=lst[fail[u]];
		forup(c,0,1){
			if(tr[u][c]){
				fail[tr[u][c]]=tr[fail[u]][c];
				q.push(tr[u][c]);
			}else{
				tr[u][c]=tr[fail[u]][c];
			}
		}
	}
}
bitset<N> grp[N],nw,vis;
int pre[N],pr[N],pl[N];
bool bfs(int s){
	queue<int> q;
	q.push(s);
	pre[s]=0;
	vis.set();
	while(q.size()){
		int u=q.front();q.pop();
		nw=vis&grp[u];
		for(int i=nw._Find_first();i<=n;i=nw._Find_next(i)){
			vis.reset(i);
			pre[i]=u;
			if(!pr[i]){
				while(i){
					pr[i]=pre[i];
					swap(i,pl[pre[i]]);
				}
				return true;
			}
			q.push(pr[i]);
		}
	}
	return false;
}
int work(){
	int ans=0;
	forup(i,1,n){
		ans+=bfs(i);
	}
	return ans;
}
bitset<N> v;
vector<int> ans;
signed main(){
	n=read();
	forup(i,1,n){
		scanf(" %s",str+1);
		Insert(strlen(str+1),i);
	}
	Build();
	forup(i,1,n){
		for(auto j:pth[i]){
			if(lst[j]==i){
				if(lst[fail[j]]){
					grp[i][lst[fail[j]]]=true;
				}
			}else if(lst[j]){
				grp[i][lst[j]]=true;
			}
		}
	}
	forup(k,1,n){
		forup(i,1,n){
			if(i==k) continue;
			forup(j,1,n){
				if(i==j||k==j||grp[i][j]) continue;
				grp[i][j]=grp[i][k]&grp[k][j];
			}
		}
	}
	printf("%d\n",n-work());
	forup(i,1,n){
		if(!pl[i]){
			ans.push_back(i);
		}
	}
	bool flag=true;
	while(flag){
		flag=false;
		for(auto x:ans){
			v=v|grp[x];
		}
		for(auto &x:ans){
			if(v[x]){
				flag=true;
				while(v[x]) x=pr[x];
			}
		}
	}
	for(auto i:ans){
		printf("%d ",i);
	}
}
```

///

### P2825 [HEOI2016/TJOI2016] 游戏

[传送门](https://www.luogu.com.cn/problem/P2825)

容易发现如果没有 `#` 就是一个很板的二分图匹配模型。

那么有了 `#` 其实仍然是二分图匹配模型，只是每个结点不代表一整行/列，而是代表**能互相到达的横/纵向最长宽度为 $1$ 的连通块**。

边数是 $O(n^2)$ 级别的，随便过。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=55,inf=0x3f3f3f3f;
int n,m;
int a[N][N],num[N][N],cntn;
char str[N];
namespace flow{
	struct edge{
		int v,rst,nxt;
	}e[N*N*2];
	int head[N*N*2],cur[N*N*2],dpt[N*N*2],cnte,s,t;
	void adde(int u,int v,int w){
		e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,0,head[v]};head[v]=cnte;
	}
	bool bfs(){
		forup(i,1,cntn){
			dpt[i]=-1;
			cur[i]=head[i];
		}
		queue<int> q;
		q.push(s);dpt[s]=0;
		while(q.size()){
			int u=q.front();q.pop();
			for(int i=head[u];i;i=e[i].nxt){
				int rst=e[i].rst,v=e[i].v;
				if(!rst||~dpt[v]) continue;
				dpt[v]=dpt[u]+1;
				q.push(v);
			}
		}
		return ~dpt[t];
	}
	int dfs(int x,int flow){
		if(x==t||!flow) return flow;
		int res=0;
		for(int i=cur[x];i;i=e[i].nxt){
			cur[x]=i;int v=e[i].v,rst=e[i].rst;
			if(dpt[v]!=dpt[x]+1||!rst) continue;
			int gt=dfs(v,min(rst,flow-res));
			if(gt){
				res+=gt;
				e[i].rst-=gt;
				e[i^1].rst+=gt;
				if(res==flow) break;
			}
		}
		return res;
	}
	int dinic(){
		int ans=0;
		while(bfs()){
			ans+=dfs(s,inf);
		}
		return ans;
	}
}
signed main(){
	n=read();m=read();
	forup(i,1,n){
		scanf(" %s",str+1);
		forup(j,1,m){
			a[i][j]=(str[j]=='*'?0:str[j]=='x'?1:2);
		}
	}
	flow::s=1;flow::t=2;cntn=2;
	flow::cnte=1;
	forup(i,1,n){
		++cntn;bool flag=false;
		flow::adde(flow::s,cntn,1);
		forup(j,1,m){
			if(a[i][j]==2&&flag){
				++cntn;flag=false;
				flow::adde(flow::s,cntn,1);
			}
			if(a[i][j]!=2) flag=true;
			num[i][j]=cntn;
		}
	}
	forup(j,1,m){
		++cntn;bool flag=false;
		flow::adde(cntn,flow::t,1);
		forup(i,1,n){
			if(a[i][j]==2&&flag){
				++cntn;flag=false;
				flow::adde(cntn,flow::t,1);
			}
			if(a[i][j]!=2) flag=true;
			if(a[i][j]==0){
				flow::adde(num[i][j],cntn,inf);
			}
		}
	}
	printf("%d\n",flow::dinic());
}
```

///


### AGC029F Construction of a tree

[传送门](https://www.luogu.com.cn/problem/AT_agc029_f)

唉，Hall 定理。

妙题。

考虑什么情况显然无解，若选定 $k$ 个给定的点集 $V_{p_1},V_{p_2},\dots,V_{p_k}$，它们并起来小于等于 $k$，那么对不起，这些点集无论怎么选边必定会形成环（这对于“形成环”的结论是充分条件，但不是必要条件），那么就无法建出树了。（这个结论是显然的吧？如果能建出不是环的图至少包含 $k+1$ 个点）。那么若 $S=\begin{Bmatrix}p_1,p_2,p_3\dots p_k\end{Bmatrix}$，令 $f(S)=V_{p_1}\cup V_{p_2}\cup V_{p_3}\cup \dots \cup V_{p_k}$。容易发现若 $\exists S\subseteq [1,n-1],|f(S)|\le |S|$，则无解。

考虑逆否命题是若有解则有 $\forall S\subseteq [1,n-1],|f(S)|> |S|$。根据上文可知这是一个有解的必要不充分条件。而有经验的同学就能看出这长得非常像 Hall 定理，那么考虑跑一遍二分图匹配。

具体来说，左部点是点，右部点是题目给定的点集。然后每个点集内所有点向它连边。这样匹配出来（如果有解）最大匹配必定是 $n-1$。那么左部点必然恰好有一个点 $r$ 没被匹配。

考虑从点 $r$ 开始 dfs，设当前点是 $u$，每次找一条 $u$ 开始**不是匹配边**的边走到一个没用过的右部点（点集）。那么它的匹配点 $v$ 显然能和 $u$ 连一条边 $(u,v)$。并且由于每个左部点只会有一条匹配边（除去 $r$）所以显然不会连出环。这样构造出来的显然是正确的解。考虑有没有可能有解而无法被这样构造出来。

容易发现算法终止当且仅当对于每一个经过的左部点，和它相邻的所有右部点都被经过了（无法拓展新点）。那么考虑有没有可能在有解的情况下，在没求出解时终止算法。

假设存在一个有解的情况算法终止在一半。当算法终止时，对于剩下的右部点集合 $T$，容易发现 $f(T)$ 必然是没被经过的左部点的子集（更准确的说，由于最大匹配是 $n-1$，所以其实必定就是剩下的左部点集合）。又由于无论何时被经过的左部点数量都恰好是右部点数量 $+1$（考虑每次左右各拓展一个，但最初有一个 $r$）。则此时 $|f(T)|\le |T|$，根据上文可知无解，与假设矛盾。值得注意的是，由于命题“若有解则有 $\forall S\subseteq [1,n-1],|f(S)|> |S|$”是一个必要不充分条件，所以当无解时最大匹配也有可能是 $n-1$，但是这个构造方案是充分必要的，具体实现需要注意这一点。复杂度 $O(N+M+M\sqrt{N})$。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=2e5+5,inf=0x3f3f3f3f;
int n;
struct edge{
	int v,rst,nxt;
}e[N<<2];
int head[N],cur[N],dpt[N],cnte,s,t;
void adde(int u,int v,int w){
//	msg("%d %d||\n",u,v);
	e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
	e[++cnte]=edge{u,0,head[v]};head[v]=cnte;
}
bool bfs(){
	forup(i,1,t){
		cur[i]=head[i];
		dpt[i]=-1;
	}
	queue<int> q;
	dpt[s]=0;q.push(s);
	while(q.size()){
		int u=q.front();q.pop();
		for(int i=head[u];i;i=e[i].nxt){
			if(!e[i].rst) continue;
			int v=e[i].v;
			if(dpt[v]==-1){
				dpt[v]=dpt[u]+1;
				q.push(v);
			}
		}
	}
	return dpt[t]!=-1;
}
int dfs(int x,int flow){
	if(x==t||!flow) return flow;
	int res=0;
	for(int i=cur[x];i;i=e[i].nxt){
		cur[x]=i;int v=e[i].v;
		if(dpt[v]!=dpt[x]+1) continue;
		int gt=dfs(v,min(flow-res,e[i].rst));
		if(gt){
			res+=gt;
			e[i].rst-=gt;
			e[i^1].rst+=gt;
			if(res==flow) break;
		}
	}
	return res;
}
int dinic(){
	int ans=0;
	while(bfs()){
		ans+=dfs(s,inf);
	}
	return ans;
}
int vis[N];
pii ans[N];
void dfs1(int u){
	for(int i=head[u];i;i=e[i].nxt){
		int p=e[i].v;
//		msg("%d %d %d|1\n",u,p,e[i].rst);
		if(p<s&&!vis[p-n]&&e[i].rst){
			vis[p-n]=1;
			for(int j=head[p];j;j=e[j].nxt){
				int v=e[j].v;
				if(v<s&&e[j].rst){
//					msg("%d %d %d|2\n",p-n,u,v);
					ans[p-n]=mkp(u,v);
					dfs1(v);
				}
			}
		}
	}
}
signed main(){
	n=read();
	cnte=1;
	s=n*2+1;t=n*2+2;
	forup(i,1,n){
		adde(s,i,1);adde(i+n,t,1);
	}
	forup(i,1,n-1){
		int k=read();
		forup(j,1,k){
			int v=read();
			adde(v,n+i,1);
		}
	}
	int res=dinic();
	if(res!=n-1){
		puts("-1");
		return 0;
	}
	int r=0;
	for(int i=head[s];i;i=e[i].nxt){
		if(e[i].rst){
			r=e[i].v;
			break;
		}
	}
	dfs1(r);
	forup(i,1,n-1){
		if(!vis[i]){
			puts("-1");
			return 0;
		}
	}
	forup(i,1,n-1){
		printf("%d %d\n",ans[i].fi,ans[i].se);
	}
}
```

///

## 费用流

### P4249 [WC2007] 剪刀石头布

[传送门](https://www.luogu.com.cn/problem/P4249)

最大化竞赛图三元环。

考虑如果要判三元环，那么和三行都有关，不好搞。但是“从 $n$ 个点中选三个”这个问题是简单的。那么考虑用总的减去不是三元环的。

容易发现，竞赛图上三个点不组成三元环，当且仅当其中一个点指向剩下的两个。换句话说，**存在一个点在导出子图中出度为二**。那么容易发现，令 $d_u$ 为 $u$ 的出度，那么要减去的就是 $\sum_{i=1}^n\frac{d_i(d_i-1)}{2}$。

对于一条边 $(u,v)$，若它是 $u\to v$ 的有向边，那么会向 $u$ 提供一个出度。若是无向边，那么有可能向 $u$ 或者 $v$ 提供出度。容易想到可以用 flow 做。

那么具体怎么做呢？注意到 $\frac{d_i(d_i-1)}{2}$ 这个式子长得像等差数列求和（其实手模一遍会发现者本质上就是等差数列求和）。那么容易想到费用流，从每个“点”向汇点连 $n$ 条流量为 $1$，费用分别为 $0,1,\dots n-1$ 的边，然后就解决了。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=105,inf=0x3f3f3f3f;
int n,ans[N][N];
struct edge{
	int v,w,rst,nxt;
}e[N*N*7];
int head[N*N],cnte,pre[N*N],incf[N*N],s,t,dis[N*N],cntn;
bool vis[N*N];
void adde(int u,int v,int w,int c){
	e[++cnte]=edge{v,w,c,head[u]};head[u]=cnte;
	e[++cnte]=edge{u,-w,0,head[v]};head[v]=cnte;
}
bool spfa(){
	mem(dis,0x3f);
	queue<int> q;
	q.push(s);dis[s]=0;incf[s]=inf;incf[t]=0;
	while(q.size()){
		int u=q.front();q.pop();
//		msg("%d|",u);
		vis[u]=0;
		for(int i=head[u];i;i=e[i].nxt){
			int v=e[i].v,w=e[i].w,rst=e[i].rst;
			if(!rst||dis[v]<=dis[u]+w) continue;
//			msg("%d ",v);
			dis[v]=dis[u]+w;incf[v]=min(rst,incf[u]);pre[v]=i;
			if(!vis[v]){
				q.push(v);
				vis[v]=1;
			}
		}
//		msg("|%d|\n",q.size());
	}
	return incf[t]!=0;
}
int mxf,mnc;
void update(){
	mxf+=incf[t];
	for(int u=t;u!=s;u=e[pre[u]^1].v){
//		msg("%d ",u);
		e[pre[u]].rst-=incf[t],e[pre[u]^1].rst+=incf[t];
		mnc+=incf[t]*e[pre[u]].w;
	}
//	msg("%d|\n",mxf);
}
signed main(){
	n=read();cntn=n;
	cnte=1;
	s=++cntn;
	forup(i,1,n){
		forup(j,1,n){
			int a=read();
			ans[i][j]=a;
			if(j>i){
				int nw=++cntn;
				if(a!=1) adde(nw,j,0,1);
				if(a!=0) adde(nw,i,0,1);
				adde(s,nw,0,1);
				if(a==2) ans[i][j]=nw;
			}
		}
	}
	t=++cntn;
	forup(i,1,n){
		forup(j,0,n-1){
			adde(i,t,j,1);
		}
	}
//	msg("1");
	while(spfa())update();
	printf("%d\n",n*(n-1)*(n-2)/6-mnc);
	forup(i,1,n){
		forup(j,i+1,n){
			if(ans[i][j]>1){
				int nw=ans[i][j];
				for(int k=head[nw];k;k=e[k].nxt){
					if(e[k].rst&&e[k].v<=n){
						int v=e[k].v;
						if(v==i){
							ans[i][j]=0;ans[j][i]=1;
						}else{
							ans[i][j]=1;ans[j][i]=0;
						}
						break;
					}
				}
			}
		}
	}
	forup(i,1,n){
		forup(j,1,n){
			printf("%d ",ans[i][j]);
		}
		puts("");
	}
}
```

///

### P3967 [TJOI2014] 匹配

[传送门](https://www.luogu.com.cn/problem/P3967)

简单题啊。

首先求最大权匹配可以跑最大费用最大流。

那么怎么看一条边是不是在所有匹配中都出现了呢？删掉它看答案变不变不就行了。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=85,inf=0x3f3f3f3f;
int n,h[N][N];
vector<pii> vec,ans;
struct FLOW{
	struct edge{
		int v,c,rst,nxt;
	}e[N*N*3];
	int head[N*2],dis[N*2],pre[N*2],incf[N*2],vis[N*2],cnte,s,t;
	void adde(int u,int v,int w,int c){
		e[++cnte]=edge{v,-c,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,c,0,head[v]};head[v]=cnte;
	}
	void init(){
		cnte=1;
		mxf=mnc=0;
		mem(head,0);
		s=n*2+1;t=n*2+2;
		forup(i,1,n){
			adde(s,i,1,0);adde(i+n,t,1,0);
		}
	}
	bool spfa(){
//		msg("1");
		mem(dis,0x3f);
		queue<int> q;q.push(s);dis[s]=0;
		incf[s]=inf;incf[t]=0;
		while(q.size()){
			int u=q.front();q.pop();
//			msg("%d %d|\n",u,dis[u]);
			vis[u]=0;
			for(int i=head[u];i;i=e[i].nxt){
				int v=e[i].v,c=e[i].c,rst=e[i].rst;
				if(!rst||dis[v]<=dis[u]+c) continue;
				dis[v]=dis[u]+c;incf[v]=min(rst,incf[u]);pre[v]=i;
				if(!vis[v]) q.push(v),vis[v]=1;
			}
		}
		return incf[t]!=0;
	}
	int mxf,mnc;
	void update(){
//		msg("2");
		mxf+=incf[t];
		for(int p=t;p!=s;p=e[pre[p]^1].v){
			e[pre[p]].rst-=incf[t];
			e[pre[p]^1].rst+=incf[t];
			mnc+=e[pre[p]].c*incf[t];
		}
//		msg("3");
	}
	void calc(){
		while(spfa()) update();
	}
	void find(){
		for(int i=head[s];i;i=e[i].nxt){
			if(!e[i].rst){
				int v=e[i].v;
				for(int j=head[v];j;j=e[j].nxt){
					if(!e[j].rst){
						msg("%d %d|\n",v,e[j].v);
						vec.push_back(mkp(v,e[j].v));
					}
				}
			}
		}
	}
}flow;
int oflow,ocost;
signed main(){
	n=read();
	flow.init();
	forup(i,1,n){
		forup(j,1,n){
			h[i][j]=read();
			flow.adde(i,j+n,1,h[i][j]);
		}
	}
	flow.calc();
	flow.find();
	oflow=flow.mxf;ocost=flow.mnc;
	for(auto t:vec){
		int u=t.fi,v=t.se;
		flow.init();
		forup(i,1,n){
			forup(j,1,n){
				if(i==u&&j+n==v) continue;
				flow.adde(i,j+n,1,h[i][j]);
			}
		}
		flow.calc();
		if(flow.mxf!=oflow||flow.mnc!=ocost){
			ans.push_back(t);
		}
	}
	printf("%d\n",-ocost);
	sort(ans.begin(),ans.end());
	for(auto i:ans){
		printf("%d %d\n",i.fi,i.se-n);
	}
}
```

///

## 最小割

### P1791 [国家集训队] 人员雇佣

[传送门](https://www.luogu.com.cn/problem/P1791)

首先如果不看各种 $E_{i,j}$ 的问题，即每个人贡献独立，那么就是一个经典最小割模型。

具体来说，核心思路为用收益综合减去最小代价。我们对每个人开一个点，然后直接从源点连向他们，从他们连向汇点。我们希望若他处于 $S$ 集合，则表示雇佣他，若处于 $T$ 集合则表示不雇佣他。

如果不雇佣他，那么就会损失他的贡献，此时会割掉远点到他的边，这条边容量就应该是他的贡献。而如果雇佣他，就会损失他的工资，此时会割掉他到汇点的边，所以这条边的容量就应该是他的工资。

形式化的，对于所有点 $u$ 连边 $(s,u,\sum E_{u,i}),(u,t,A_u)$。

接着考虑对于两个人 $i,j$，如果只雇佣其中一个人会发生什么。容易发现此时两人不在同一集合。即两人之间的边会被割掉。注意我们之前的贡献是假设**其余所有人都被雇佣**，那么就会在原贡献中减去一个 $E_{i,j}$，又因为题设中有说“若雇佣了 $i$ 没雇佣 $j$ 那么利润会减少 $E_{i,j}$”，故连一条 $(i,j,2E_{i,j})$ 的边即可。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(i64 i=(s);i<=(e);i++)
#define fordown(i,s,e) for(i64 i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
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
const i64 N=1005,inf=1e18;
i64 n,a[N],e[N][N],ans;
struct FLOW{
	struct edge{
		i64 v,rst,nxt;
	}e[N*N*3];
	i64 head[N],cur[N],cnte,s,t,dpt[N];
	void adde(i64 u,i64 v,i64 w){
		e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,0,head[v]};head[v]=cnte;
	}
	bool bfs(){
		forup(i,1,t){
			cur[i]=head[i];
			dpt[i]=-1;
		}
		dpt[s]=0;
		queue<i64> q;q.push(s);
		while(q.size()){
			i64 u=q.front();q.pop();
			for(i64 i=head[u];i;i=e[i].nxt){
				if(!e[i].rst) continue;
				i64 v=e[i].v;
				if(dpt[v]==-1){
					dpt[v]=dpt[u]+1;
					q.push(v);
				}
			}
		}
		return dpt[t]!=-1;
	}
	i64 dfs(i64 x,i64 flow){
		if(x==t||!flow) return flow;
		i64 res=0;
		for(i64 i=cur[x];i;i=e[i].nxt){
			cur[x]=i;i64 v=e[i].v;
			if(!e[i].rst||dpt[v]!=dpt[x]+1) continue;
			i64 gt=dfs(v,min(e[i].rst,flow-res));
			if(gt){
				e[i].rst-=gt;
				e[i^1].rst+=gt;
				res+=gt;
				if(res==flow) break;
			}
		}
		return res;
	}
	i64 dinic(){
		i64 ans=0;
		while(bfs()){
			ans+=dfs(s,inf);
		}
		return ans;
	}
}flow;
signed main(){
	n=read();
	forup(i,1,n) a[i]=read();
	flow.cnte=1;flow.s=n+1;flow.t=n+2;
	forup(i,1,n){
		i64 sum=0;
		forup(j,1,n){
			e[i][j]=read();
			if(i!=j) flow.adde(i,j,2*e[i][j]);
			sum+=e[i][j];
			ans+=e[i][j];
		}
		flow.adde(flow.s,i,sum);flow.adde(i,flow.t,a[i]);
	}
	ans-=flow.dinic();
	printf("%lld\n",ans);
}
```

///

### P1361 小M的作物

[传送门](https://www.luogu.com.cn/problem/P1361)

套路题。

因为是把一些东西分成两半的形式，容易想到最小割。

那么先不考虑组合，就是经典最小割建模，这篇博客前面有，此处不具体说明。本题中我设在 $S$ 集合为在 $A$ 田地，在 $T$ 集合为在 $B$ 田地。

然后考虑一个组合何时不产生贡献。容易发现，当该组合**存在一个成员在 $A$ 田地时，会损失 $c_{2,i}$ 的代价**，对于 $c_{1,i}$ 同理。

那么考虑怎么构造一条边使得**当且仅当对应组合中所有点都在 $T$ 集合时，它不会被割掉**。容易想到新建一个点向汇点连容量为 $c_{2,i}$ 的边，然后组合内所有点向它连流量为正无穷的边。对于 $c_{1,i}$ 同理。

看起来最多需要连 $O(nm)$ 条边（就是每个组合都包含所有点），但是跑的贼快。我觉得可能是数据范围有锅。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=1005,inf=0x3f3f3f3f;
int n,m,a[N],b[N],cntn,ans;
namespace flow{
	struct edge{
		int v,rst,nxt;
	}e[N*N*4];
	int head[N*3],cur[N*3],dpt[N*3],cnte,s,t;
	void init(){
		cnte=1;
		cntn=n;s=++cntn;t=++cntn;
	}
	void adde(int u,int v,int w){
		e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,0,head[v]};head[v]=cnte;
	}
	bool bfs(){
		forup(i,1,cntn){
			dpt[i]=-1;cur[i]=head[i];
		}
		queue<int> q;
		q.push(s);dpt[s]=0;
		while(q.size()){
			int u=q.front();q.pop();
			for(int i=head[u];i;i=e[i].nxt){
				int rst=e[i].rst,v=e[i].v;
				if(!rst||~dpt[v]) continue;
				dpt[v]=dpt[u]+1;
				q.push(v);
			}
		}
		return ~dpt[t];
	}
	int dfs(int x,int flow){
		if(x==t||!flow) return flow;
		int res=0;
		for(int i=cur[x];i;i=e[i].nxt){
			cur[x]=i;
			int v=e[i].v,rst=e[i].rst;
			if(!rst||dpt[v]!=dpt[x]+1) continue;
			int gt=dfs(v,min(rst,flow-res));
			if(gt){
				res+=gt;
				e[i].rst-=gt;
				e[i^1].rst+=gt;
				if(res==flow) break;
			}
		}
		return res;
	}
	int dinic(){
		int ans=0;
		while(bfs()){
			ans+=dfs(s,inf);
		}
		return ans;
	}
}
signed main(){
	n=read();
	forup(i,1,n) a[i]=read();
	forup(i,1,n) b[i]=read();
	flow::init();
	forup(i,1,n){
		flow::adde(flow::s,i,a[i]);flow::adde(i,flow::t,b[i]);
		ans+=a[i];ans+=b[i];
	}
	m=read();
	forup(i,1,m){
		int k=read(),c1=read(),c2=read(),n1=++cntn,n2=++cntn;
		flow::adde(n2,flow::t,c2);flow::adde(flow::s,n1,c1);
		ans+=c1+c2;
		forup(j,1,k){
			int u=read();
			flow::adde(u,n2,inf);flow::adde(n1,u,inf);
		}
	}
	printf("%d\n",ans-flow::dinic());
}
```

///

### P2805 [NOI2009] 植物大战僵尸

[传送门](https://www.luogu.com.cn/problem/P2805)

容易发现，这道题吃每个植物有个先后顺序，那么要求的就是反图的最大权闭合子图。

但有个问题，就是有环的情况。显然如果有环那么**原图**上的后继都要不了了，在原图上跑个拓扑排序即可。

边数最多是 $O(n^2)$ 的，主要看数据怎么给。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(int i=(s);i<=(e);i++)
#define fordown(i,s,e) for(int i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=605,inf=0x3f3f3f3f;
int n,m,val[N],deg[N],ans;
vector<int> e[N];
namespace flow{
	struct edge{
		int v,rst,nxt;
	}e[N*N*2];
	int head[N],cur[N],dpt[N],cnte,s,t;
	void adde(int u,int v,int w){
		e[++cnte]=edge{v,w,head[u]};head[u]=cnte;
		e[++cnte]=edge{u,0,head[v]};head[v]=cnte;
	}
	bool bfs(){
		forup(i,0,t){
			dpt[i]=-1;
			cur[i]=head[i];
		}
		queue<int> q;
		q.push(s);dpt[s]=0;
		while(q.size()){
			int u=q.front();q.pop();
			for(int i=head[u];i;i=e[i].nxt){
				int rst=e[i].rst,v=e[i].v;
				if(!rst||~dpt[v]) continue;
				dpt[v]=dpt[u]+1;
				q.push(v);
			}
		}
		return ~dpt[t];
	}
	int dfs(int x,int flow){
		if(x==t||!flow) return flow;
		int res=0;
		for(int i=cur[x];i;i=e[i].nxt){
			cur[x]=i;int v=e[i].v,rst=e[i].rst;
			if(dpt[v]!=dpt[x]+1||!rst) continue;
			int gt=dfs(v,min(rst,flow-res));
			if(gt){
				res+=gt;
				e[i].rst-=gt;
				e[i^1].rst+=gt;
				if(res==flow) break;
			}
		}
		return res;
	}
	int dinic(){
		int ans=0;
		while(bfs()){
			ans+=dfs(s,inf);
		}
		return ans;
	}
}
signed main(){
	n=read();m=read();
	flow::cnte=1;
	forup(i,0,n*m-1){
		val[i]=read();
		int k=read();
		forup(j,1,k){
			int x=read(),y=read();
			e[i].push_back(x*m+y);
			++deg[x*m+y];
		}
	}
	forup(i,0,n-1){
		forup(j,1,m-1){
			e[i*m+j].push_back(i*m+j-1);
			++deg[i*m+j-1];
		}
	}
	queue<int> q;
	forup(i,0,n*m-1){
		if(!deg[i]) q.push(i);
	}
	flow::s=n*m;flow::t=n*m+1;
	while(q.size()){
		int u=q.front();q.pop();
		if(val[u]>0){
			ans+=val[u];
			flow::adde(flow::s,u,val[u]);
		}else{
			flow::adde(u,flow::t,-val[u]);
		}
		for(auto i:e[u]){
			--deg[i];
			flow::adde(i,u,inf);
			if(!deg[i]){
				q.push(i);
			}
		}
	}
	printf("%d\n",ans-flow::dinic());
}
```

///