---
comments: true
---

# 反演与筛法相关题目

把这俩放一起的原因是很多筛题都需要先反演。而且两类算法本质上都是求某个函数的值。

## P2257 YY 的 GCD

[传送门](https://www.luogu.com.cn/problem/P2257)

> 题意

- 给定 $n,m\le 10^7$，求 $1\le x\le n,1\le y\le m$，且 $\gcd(x,y)$ 是质数的 $(x,y)$ 有多少对。有多测 $t\le 10^4$。

> 题解

容易想到莫反，考虑化式子：

$$
\begin{aligned}
&\sum_{p\in prime}^{p\le \min(n,m)}\sum_{i=1}^{\left\lfloor\frac{n}{p}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{p}\right\rfloor}[\gcd(i,j)=1]\\\\
=&\sum_{p\in prime}^{p\le \min(n,m)}\sum_{i=1}^{\left\lfloor\frac{n}{p}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{p}\right\rfloor}\sum_{d\mid \gcd(i,j)}\mu(d)\\\\
=&\sum_{p\in prime}^{p\le \min(n,m)}\sum_{d=1}^{d\le \min(\left\lfloor\frac{n}{p}\right\rfloor,\left\lfloor\frac{m}{p}\right\rfloor)}\mu(d)\sum_{d\mid i}^{i\le \left\lfloor\frac{n}{p}\right\rfloor}\sum_{d\mid j}^{j\le \left\lfloor\frac{m}{p}\right\rfloor}1\\\\
=&\sum_{p\in prime}^{p\le \min(n,m)}\sum_{d=1}^{d\le \min(\left\lfloor\frac{n}{p}\right\rfloor,\left\lfloor\frac{m}{p}\right\rfloor)}\mu(d)\left\lfloor\frac{n}{p\cdot d}\right\rfloor\left\lfloor\frac{m}{p\cdot d}\right\rfloor\\\\
\end{aligned}
$$

这个看起来已经很简了，但还是会 TLE，考虑继续优化。如果先枚举 $T=p\cdot d$ 的话，容易发现：


$$
\begin{aligned}
&\sum_{p\in prime}^{p\le \min(n,m)}\sum_{d=1}^{d\le \min(\left\lfloor\frac{n}{p}\right\rfloor,\left\lfloor\frac{m}{p}\right\rfloor)}\mu(d)\left\lfloor\frac{n}{p\cdot d}\right\rfloor\left\lfloor\frac{m}{p\cdot d}\right\rfloor\\\\
=&\sum_{T=1}^{\min(n,m)}\left\lfloor\frac{n}{T}\right\rfloor\left\lfloor\frac{m}{T}\right\rfloor\sum_{p\in prime}^{p\mid T}\mu(\frac{T}{p})\\\\
\end{aligned}
$$

而 $\sum_{p\in prime}^{p\mid T}\mu(\frac{T}{p})$ 是可以在线性筛的过程中求出来的，设 $g(T)=\sum_{p\in prime}^{p\mid T}\mu(\frac{T}{p})$。具体考虑线性筛每次枚举的 $i$ 和质数 $j$，若 $i\bmod j=0$，则 $g(ij)=\mu(i)-g(i)$（$i$ 里面已有的会乘上 $\mu(j)$，再加上取 $p=j$），否则，$g(ij)=\mu(i)$（如果 $p\ne j$，那么 $j^2\mid \frac{T}{p}$，则 $\mu(\frac{T}{p})=0$，$p$ 只能取 $j$），然后算一下前缀和就做完了。

复杂度 $O(n+t\sqrt{n})$。

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
const i64 N=1e7+5;
i64 n,m,mu[N],vv[N],sum[N];
vector<i64> pri;
void init(i64 n){
	mu[1]=vv[1]=1;
	forup(i,2,n){
		if(!vv[i]){
			mu[i]=-1;
			sum[i]=1;
			pri.push_back(i);
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			vv[i*j]=1;
			if(!(i%j)){
				mu[i*j]=0;
				sum[i*j]=mu[i];
				break;
			}else{
				mu[i*j]=-mu[i];
				sum[i*j]=-sum[i]+mu[i];
			}
		}
	}
	forup(i,1,n){
		sum[i]+=sum[i-1];
	}
}
signed main(){
	int t=read();
	init(1e7);
	while(t--){
		n=read();m=read();
		i64 res=0;
		for(i64 l=1,r=1;l<=min(n,m);l=r+1){
			r=min(n/(n/l),m/(m/l));	
			res+=1ll*(sum[r]-sum[l-1])*(n/l)*(m/l);
		}
		printf("%lld\n",res);
	}
}
```

///

## P2522 [HAOI2011] Problem b

[传送门](https://www.luogu.com.cn/problem/P2522)

一眼莫反板子，首先简单转化成 $\gcd=1$ 的问题，然后化一下式子：

$$
\begin{aligned}
&\sum_{i=a}^b\sum_{j=c}^d[\gcd(i,j)=1]\\\\
=&\sum_{i=a}^b\sum_{j=c}^d\sum_{t\mid \gcd(i,j)}\mu(t)\\\\
=&\sum_{t}\mu(t)\sum_{t\mid i}^{a\le i\le b}\sum_{t\mid j}^{c\le j\le d}1\\\\
=&\sum_{t}\mu(t)(\left\lfloor\frac{b}{t}\right\rfloor-\left\lfloor\frac{a-1}{t}\right\rfloor)(\left\lfloor\frac{d}{t}\right\rfloor-\left\lfloor\frac{c-1}{t}\right\rfloor)\\\\
\end{aligned}
$$

然后数论分块即可。

转化为 $\gcd=1$ 问题时，注意 $a,c$ 要上取整，$c,d$ 要下取整，以及数论分块边界问题。

复杂度 $O(a+n\sqrt{a})$。

/// details | 参考代码片段
	open: False
	type: success

```cpp
const int N=5e4+5;
int a,b,c,d,k;
i64 sum[N];//莫比乌斯函数的前缀和。
int calcr(int l){
	int r=min(b/(b/l),d/(d/l));
	if(l<=a) r=min(r,a/(a/l));
	if(l<=c) r=min(r,c/(c/l));
	return r;
}
void solve(){
	a=read(),b=read(),c=read(),d=read(),k=read();
	a=(a-1)/k;b=b/k;c=(c-1)/k;d=d/k;
	i64 res=0;
	for(int l=1,r=0;l<=min(b,d);l=r+1){
		r=calcr(l);
		res+=1ll*(sum[r]-sum[l-1])*(b/l-a/l)*(d/l-c/l);
	}
	printf("%lld\n",res);
}
```

///

## 模拟赛神秘题目

> 题意

- 给定 $n$，问有多少 $(a,b,c)$ 正整数三元组，满足 $a^2+b^2=c^2$，且 $1\le a\le n$。
- $1\le n\le 10^{10}$。

> 题解

赛时一点思路没有，感觉很神秘啊。

考虑 $a^2+b^2=c^2\Leftrightarrow a^2=(b+c)(b-c)$，那么对于一个 $a$，每个满足 $d\mid a^2,d\equiv \frac{a}{d}\pmod 2,d\ne a$ 的正整数 $d$ 都能对应一个 $b-c$（或者 $b+c$，显然同一个情况会在 $b-c,b+c$ 各算一次，除以二即可）。其中 $d\equiv \frac{a}{d}\pmod 2$ 是保证 $b,c$ 是整数，其余两个限制都显然。

设 $\tau(x)$（`\tau`） 表示 $x$ 的约数个数。容易想到 $a$ 的贡献就是 $\frac{\tau(a^2)-1}{2}$，其中 $-1$ 是去掉 $d=a$ 的情况（这种情况 $c=0$ 了）。

但这个没有考虑 $d,\frac{a}{d}$ 奇偶性不同的情况。如果 $a$ 是奇数，那么这两个东西的奇偶性必然相同，可以直接用上面那个式子。但当 $a$ 是偶数时，必须两边各分至少一个 $2$，那么不妨直接将两边均除以 $2$ 来计算，这样就能避免奇偶性的限制了，于是贡献为 $\frac{\tau(\frac{a^2}{4})-1}{2}$。

综上，答案即为：

$$
\begin{aligned}
&\left(\sum_{a=1,2\mid a}^{n}\frac{\tau(\frac{a^2}{4})-1}{2}\right)+\left(\sum_{a=1,2\nmid a}^n\frac{\tau(a)-1}{2}\right)\\\\
=&\dfrac{\left(\sum_{i=1}^{\frac{n}{2}}\tau(i^2)\right)-\left\lfloor\frac{n}{2}\right\rfloor+\left(\sum_{i=1,2\nmid i}^{n}\tau(i^2)\right)-\left\lceil\frac{n}{2}\right\rceil}{2}
\end{aligned}
$$

那么难点就规约成了计算 $\sum_{i=1}^{n}\tau(i^2)$ 和 $\sum_{i=1,i\nmid 2}^{\frac{n}{2}}\tau(i^2)$。

- $\sum_{i=1}^{n}\tau(i^2)$

考虑一个经典（？）结论，$\tau(n^2)=\sum_{x\mid n}\sum_{y\mid n}[\gcd(x,y)=1]$，大概思路就是每一对 $(x,y)$ 二元组对应 $\frac{n\times x}{y}$，当 $x,y$ 不互质时就会和某对互质的 $(x',y')$ 算重。

于是：

$$
\begin{aligned}
\sum_{i=1}^{n}\tau(i^2)
&=\sum_{i=1}^n\sum_{x\mid i}\sum_{y\mid i}[\gcd(x,y)=1]\\\\
&=\sum_{x=1}^n\sum_{y=1}^n[\gcd(x,y)=1]\sum_{\mathrm{lcm}(x,y)\mid i}^{i\le n}1\\\\
&=\sum_{x=1}^n\sum_{y=1}^n[\gcd(x,y)=1]\left\lfloor\frac{n}{\mathrm{lcm}(x,y)}\right\rfloor\\\\
&=\sum_{x=1}^n\sum_{y=1}^n[\gcd(x,y)=1]\left\lfloor\frac{n}{xy}\right\rfloor\\\\
&=\sum_{x=1}^n\sum_{y=1}^n\left\lfloor\frac{n}{xy}\right\rfloor\sum_{d\mid \gcd(x,y)}\mu(d)\\\\
&=\sum_{d=1}^n\mu(d)\sum_{d\mid x}\sum_{d\mid y}\left\lfloor\frac{n}{xy}\right\rfloor\\\\
&=\sum_{d=1}^n\mu(d)\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\left\lfloor\frac{n}{(i\times d)(j\times d)}\right\rfloor\\\\
&=\sum_{d=1}^{\left\lfloor\sqrt{n}\right\rfloor}\mu(d)\sum_{i=1}^{\left\lfloor\frac{n}{d^2}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{d^2}\right\rfloor}\left\lfloor\frac{n}{ij\times d^2}\right\rfloor\\\\
&=\sum_{d=1}^{\left\lfloor\sqrt{n}\right\rfloor}\mu(d)\sum_{i=1}^{\left\lfloor\frac{n}{d^2}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{d^2}\right\rfloor}\left\lfloor\frac{\left\lfloor\frac{n}{i\times d^2}\right\rfloor}{j}\right\rfloor
\end{aligned}
$$

其中 $n\to \left\lfloor\sqrt{n}\right\rfloor$ 的这一步是因为当 $d>\sqrt{n}$ 时 $\left\lfloor\frac{n}{ij\times d^2}\right\rfloor$ 就变成 $0$ 了，$i,j$ 上界的变化也同理。

容易发现我们可以维护 $H(n)=\sum_{i=1}^n \left\lfloor\frac{n}{i}\right\rfloor=\sum_{i=1}^n\tau(i)$（线性筛求），大于 $n^{\frac{2}{3}}$ 时暴力数论分块，小于时线性筛预处理。

这样先暴力枚举 $d$，再数论分块 $i$，再引用 $H(\left\lfloor\frac{n}{i\times d^2}\right\rfloor)$，复杂度是 $O(n^{\frac{2}{3}})$。证明不会，要用一堆神秘撬棍符号，总之能过 $10^{10}$。

- $\sum_{i=1,i\nmid 2}^{\frac{n}{2}}\tau(i^2)$

其实类似吧，但是容易发现每个 $\sum$ 符号下面都要加 $2\nmid i$，然后把 $\left\lfloor\frac{n}{xy}\right\rfloor$ 换成 $\left\lceil\frac{\left\lfloor\frac{n}{xy}\right\rfloor}{2}\right\rceil$（因为只算奇数），于是有：

$$
\begin{aligned}
&\sum_{d=1,2\nmid d}^{\left\lfloor\sqrt{n}\right\rfloor}\mu(d)\sum_{i=1,2\nmid i}^{\left\lfloor\frac{n}{d^2}\right\rfloor}\sum_{j=1,2\nmid j}^{\left\lfloor\frac{n}{d^2}\right\rfloor}\left\lceil\frac{\left\lfloor\frac{\left\lfloor\frac{n}{i\times d^2}\right\rfloor}{j}\right\rfloor}{2}\right\rceil\\\\
\end{aligned}
$$

然后就维护 $H'(n)=\sum_{i=1,2\nmid i}^n\left\lceil\frac{\left\lfloor\frac{n}{i}\right\rfloor}{2}\right\rceil=\sum_{i=1,2\nmid i}^n\tau(i)$，因为 $\left\lceil\frac{\left\lfloor\frac{n}{i}\right\rfloor}{2}\right\rceil$ 就是 $i$ 的奇倍数个数，那么就是所有奇数的约数和。接着用同样方法维护即可。

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
const i64 N=1e7+5;
i64 n,mu[N],tau[N],cntl[N],sumtau[N],sumtauodd[N];
bool vv[N];
vector<i64> pri;
void init(i64 n){
	vv[1]=1;
	mu[1]=tau[1]=1;
	forup(i,2,n){
		if(!vv[i]){
			mu[i]=-1;
			tau[i]=2;
			cntl[i]=1;
			pri.push_back(i);
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			vv[i*j]=1;
			if(i%j){
				mu[i*j]=-mu[i];
				tau[i*j]=2*tau[i];
				cntl[i*j]=1;
			}else{
				mu[i*j]=0;
				tau[i*j]=(tau[i]/(cntl[i]+1))*(cntl[i]+2);
				cntl[i*j]=cntl[i]+1;
				break;
			}
		}
	}
	forup(i,1,n){
		sumtau[i]=sumtau[i-1]+tau[i];
		sumtauodd[i]=sumtauodd[i-1]+(i&1)*tau[i];
	}
}
i64 getsumtau(i64 x){
	if(x<=1e7) return sumtau[x];
	i64 res=0;
	for(i64 l=1,r=0;l<=x;l=r+1){
		r=x/(x/l);
		res+=(r-l+1)*(x/l);
	}
	return res;
}
i64 work1(i64 n){
	i64 res=0;
	for(i64 t=1;t*t<=n;++t){
		i64 w=n/(t*t),ss=0;
		for(i64 l=1,r=0;l<=w;l=r+1){
			r=w/(w/l);
			ss+=(r-l+1)*getsumtau(w/l);
		}
		res+=mu[t]*ss;
	}
	return res;
}
i64 getsumtauodd(i64 x){
	if(x<=1e7) return sumtauodd[x];
	i64 res=0;
	for(i64 l=1,r=0;l<=x;l=r+1){
		r=x/(x/l);
		res+=((r+1)/2-l/2)*((x/l+1)/2);
	}
	return res;
}
i64 work2(i64 n){
	i64 res=0;
	for(i64 t=1;t*t<=n;++t){
		if(!(t&1)) continue;
		i64 w=n/(t*t),ss=0;
		for(i64 l=1,r=0;l<=w;l=r+1){
			r=w/(w/l);
			ss+=((r+1)/2-l/2)*getsumtauodd(w/l);
		}
		res+=mu[t]*ss;
	}
	return res;
}
signed main(){
	init(1e7);
	n=read();
	i64 res=0;
	res+=(work1(n/2)-(n/2))/2;
	res+=(work2(n)-(n+1)/2)/2;
	printf("%lld\n",res);
}
```

///

## P3349 [ZJOI2016] 小星星

[传送门](https://www.luogu.com.cn/problem/P3349)

神秘子集反演。

> 题意

- 给定一棵树 $T$ 和一张无向图简单连通 $G$，点数均为 $n$。
- 问有多少个排列 $p_i$，满足 $\forall (u,v)\in E_T,(p_u,p_v)\in E_G$。
- $1\le n\le 17$，保证至少有一解。

> 题解

首先有个显然的 DP，设 $dp_{i,j,S}$ 表示考虑 $i$ 子树内 $p_i=j$ 且 $i$ 的子树恰好占满 $S$ 的方案数，转移就是一个枚举子集，复杂度 $O(n^23^n)$，显然过不了。

考虑“排列”是个非常强的限制，因为它要求 $p_i$ 不能重复。如何把这玩意去掉呢？容易想到子集反演。

具体来说，设 $f(S)$ 表示用过的 $p_i$ 恰好是 $S$ 集合的方案数（可重复使用），显然 $f(V)$ 就是答案（$V$ 是点集的全集），容易发现可以用 $g(S)=\sum_{T\subseteq S}f(T)$ 表示用过的 $p_i$ 是 $S$ 的子集的方案数。根据子集反演，有 $f(S)=\sum_{T\subseteq S}(-1)^{|S|-|T|}g(T)$。

而 $g(S)$ 可以在枚举 $S$ 后 $O(n^2)$ 求出（还是考虑 DP），于是复杂度 $O(n^22^n)$ ，就能过了。

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
const i64 N=20,inf=1e18;
i64 n,m,al;
vector<i64> e[N],e2[N];
i64 dp[N][N];
i64 ppcnt(int x){
	i64 res=0;
	while(x){
		x-=x&-x;
		++res;
	}
	return res;
}
void dfs(i64 x,i64 fa,i64 msk){
	forup(y,1,n){
		if(msk&(1<<(y-1))) dp[x][y]=1;
	}
	for(auto i:e[x]){
		if(i==fa) continue;
		dfs(i,x,msk);
		forup(y,1,n){
			if(!(msk&(1<<(y-1)))) continue;
			i64 ct=0;
			for(auto j:e2[y]){
				if(!(msk&(1<<(j-1)))) continue;
				ct+=dp[i][j];
			}
			dp[x][y]*=ct;
		}
	}
}
i64 calcg(i64 msk){
	mem(dp,0);
	dfs(1,0,msk);
	i64 res=0;
	forup(i,1,n){
		if(msk&(1<<(i-1))) res+=dp[1][i];
	}
	return res;
}
signed main(){
	n=read();m=read();
	al=(1<<n)-1;
	forup(i,1,m){
		i64 u=read(),v=read();
		e2[u].push_back(v);
		e2[v].push_back(u);
	}
	forup(i,1,n-1){
		i64 u=read(),v=read();
		e[u].push_back(v);
		e[v].push_back(u);
	}
	i64 res=0;
	forup(i,0,al){
		res+=((n-ppcnt(i))&1?-1:1)*calcg(i);
	}
	printf("%lld\n",res);
}
```

///

## P6478 [NOI Online #2 提高组] 游戏

[传送门](https://www.luogu.com.cn/problem/P6478)

> 题意

- 给定一棵 $n$ 个点（$n\equiv 0\pmod 2$）的树，上面有一半白点一半黑点。
- 现在你要把白点和黑点两两匹配，称一个匹配 $(w_i,b_i)$ 的权值为 $\sum_{i=1}^{\frac{n}{2}}[w_i\text{ is an ancestor of }b_i \lor b_i\text{ is an ancestor of }w_i ]$。
- 对于 $k\in[0,\frac{n}{2}]$，求出权值为 $k$ 的匹配个数。称两个匹配不同当且仅当同一个白点与不同的黑点匹配了。
- $1\le n\le 5000$

> 题解

非常经典的二项式反演。

容易发现权值**恰好**为 $k$ 是不好做的，因为还需要要求剩下的不能产生贡献。考虑把剩下的限制去掉，容易想到二项式反演。

设 $g(k)$ 表示权值恰好为 $k$ 的匹配数，$f(k)$ 表示权值至少为 $k$ 的匹配数，显然有 $f(k)=\sum_{i\ge k}\binom{i}{k}g(k)$，于是根据二项式反演，有 $g(k)=\sum_{i\ge k}(-1)^{i-k}\binom{i}{k}f(k)$。

那么如何求 $f(k)$ 呢？有一个很显然的树形背包，设 $dp_{i,j}$ 表示考虑 $i$ 及其子树，钦定 $j$ 对产生贡献的方案数。转移首先是将不同儿子的贡献乘起来再加，然后考虑 $i$ 和子树内颜色不同的点产生一次贡献。显然子树内没用过的点就是总数减去已经匹配过的对数，这个可以在 $dfn$ 上前缀和。复杂度分析同树形背包，为 $O(n^2)$。

然后 $f(i)=dp_{1,i}\times (n-i)!$，于是做完了，复杂度 $O(n^2)$。

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
const int N=5005,inf=0x3f3f3f3f,mod=998244353;
int n,sum[2][N],a[N];
char str[N];
vector<int> e[N];
int dfn[N],Tm,sz[N];
void dfs1(int x,int fa){
	dfn[x]=++Tm;
	a[x]=str[x]-'0';
	sum[0][dfn[x]]=sum[0][dfn[x]-1];
	sum[1][dfn[x]]=sum[1][dfn[x]-1];
	sum[a[x]][dfn[x]]++;
	for(auto i:e[x]){
		if(i==fa) continue;
		dfs1(i,x);
	}
}
int dp[N][N];
void dfs2(int x,int fa){
	dp[x][0]=1;
	sz[x]=0;
	for(auto i:e[x]){
		if(i==fa) continue;
		dfs2(i,x);
		fordown(j,sz[x],0){
			fordown(k,sz[i],1){
				(dp[x][j+k]+=1ll*dp[x][j]*dp[i][k]%mod)%=mod;
			}
		}
		sz[x]+=sz[i];
	}
	++sz[x];
	int ss=sum[a[x]^1][dfn[x]+sz[x]-1]-sum[a[x]^1][dfn[x]];
	fordown(i,sz[x],1){
		if(ss>=i) (dp[x][i]+=1ll*dp[x][i-1]*(ss-i+1)%mod)%=mod;
	}
}
int fact[N],binom[N][N];
int f[N],g[N];
signed main(){
	n=read();
	fact[0]=1;
	forup(i,1,n) fact[i]=1ll*fact[i-1]*i%mod;
	forup(i,0,n){
		binom[i][0]=1;
		forup(j,1,n){
			binom[i][j]=(binom[i-1][j-1]+binom[i-1][j])%mod;
		}
	}
	scanf(" %s",str+1);
	forup(i,1,n-1){
		int u=read(),v=read();
		e[u].push_back(v);
		e[v].push_back(u);
	}
	dfs1(1,0);
	dfs2(1,0);
	int m=n/2;
	forup(i,0,m) f[i]=1ll*dp[1][i]*fact[m-i]%mod;
	forup(i,0,m){
		forup(j,i,m){
			g[i]=(g[i]+1ll*((j-i)&1?mod-1:1)*binom[j][i]%mod*f[j]%mod)%mod;
		}
		printf("%d\n",g[i]);
	}
}
```

///

## P4859 已经没有什么好害怕的了

> 题意

- 有一个二分完全图，$u$ 的权值为 $w_u$，点权互不相同，称一个匹配的权值为 $\sum_{(u,v)}[w_u>w_v]-[w_v>w_u]$。
- 求权值恰为 $k$ 的匹配数。
- $1\le k\le n\le 2000$（两边的点数均为 $n$）

> 题解

版。

首先若 $n-k$ 是奇数显然无解。

若 $n-k$ 是偶数，容易发现把两边放到一起离散化后就变成了上一道题链的情况只算白点在上黑点在下的权值而且只有单组询问。

复杂度 $O(n^2)$。

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
const int N=2005,inf=0x3f3f3f3f,mod=1e9+9;
int n,k,t;
struct Node{
	int v,co;
}s[N<<1];
int dp[N],cnt[2],fact[N];
int binom[N][N];
signed main(){
	n=read();k=read();
	if((n-k)&1){
		puts("0");
		return 0;
	}
	t=k+(n-k)/2;
	fact[0]=1;
	forup(i,0,n){
		binom[i][0]=1;
		if(i) fact[i]=1ll*fact[i-1]*i%mod;
		forup(j,1,i){
			binom[i][j]=(binom[i-1][j]+binom[i-1][j-1])%mod;
		}
	}
	forup(i,1,n){
		s[i].v=read();s[i].co=0;
	}
	forup(i,n+1,n*2){
		s[i].v=read();s[i].co=1;
	}
	sort(s+1,s+n*2+1,[&](Node a,Node b){return a.v<b.v;});
	dp[0]=1;
	forup(i,1,n*2){
		++cnt[s[i].co];
		if(!s[i].co){
			fordown(j,cnt[1],1){
				(dp[j]+=1ll*dp[j-1]*(cnt[1]-j+1)%mod)%=mod;
			}
		}
	}
	int res=0;
	forup(i,t,n){
		(res+=1ll*((i-t)&1?mod-1:1)*dp[i]%mod*fact[n-i]%mod*binom[i][t]%mod)%=mod;
	}
	printf("%d\n",res);
}
```

///

## P3327 [SDOI2015] 约数个数和

[传送门](https://www.luogu.com.cn/problem/P3327)

> 题意

- 求 $\sum_{i=1}^n\sum_{j=1}^m\tau(i\times j)$，其中 $\tau(i)$ 表示 $i$ 的约数个数。
- $1\le n,m,T\le 50000$，$T$ 是测试数据组数。

> 题解

莫反套路题。

考虑 $\tau(n\times m)=\sum_{i\mid n}\sum_{j\mid m}[\gcd(i,j)=1]$，一对 $(i,j)$ 代表 $\frac{n\times j}{i}$，如果不互质就会算重。

于是（钦定 $n<m$）：

$$
\begin{aligned}
&\sum_{i=1}^n\sum_{j=1}^m\tau(i\times j)\\\\
=&\sum_{i=1}^n\sum_{j=1}^m\sum_{x\mid i}\sum_{y\mid j}[\gcd(x,y)=1]\\\\
=&\sum_{i=1}^n\sum_{j=1}^m\sum_{x\mid i}\sum_{y\mid j}\sum_{d\mid \gcd(x,y)}\mu(d)\\\\
=&\sum_{d=1}^n\mu(d)\sum_{d\mid x}^{x\le n}\sum_{d\mid y}^{y\le m}\sum_{x\mid i}^{i\le n}\sum_{y\mid j}^{j\le m}1\\\\
=&\sum_{d=1}^n\mu(d)\sum_{x=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{y=1}^{\left\lfloor\frac{m}{d}\right\rfloor}\sum_{i=1}^{\left\lfloor\frac{n}{x\times d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{y\times d}\right\rfloor}1\\\\
=&\sum_{d=1}^n\mu(d)\sum_{x=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\left\lfloor\frac{n}{x\times d}\right\rfloor\sum_{y=1}^{\left\lfloor\frac{m}{d}\right\rfloor}\left\lfloor\frac{m}{y\times d}\right\rfloor\\\\
=&\sum_{d=1}^n\mu(d)\sum_{x=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\tau(x)\sum_{y=1}^{\left\lfloor\frac{m}{d}\right\rfloor}\tau(y)\\\\
\end{aligned}
$$

最后一步是因为每个数在 $\left\lfloor\frac{n}{d}\right\rfloor$ 以内的倍数个数和就是这个范围内每个数的因数个数和。

然后就可以线性筛预处理 $\tau(n)$ 的前缀和然后数论分块做了。复杂度 $O(n+T\sqrt{n})$。

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
const i64 N=50005,inf=0x3f3f3f3f;
i64 mu[N],summu[N],tau[N],cntl[N],sumtau[N];
vector<i64> pri;
void init(i64 n){
	tau[1]=1;mu[1]=1;
	forup(i,2,n){
		if(!tau[i]){
			tau[i]=2;cntl[i]=1;
			mu[i]=-1;
			pri.push_back(i);
		}
		for(auto j:pri){
			if(i*j>n) break;
			if(i%j){
				tau[i*j]=tau[i]*2;cntl[i*j]=1;
				mu[i*j]=-mu[i];
			}else{
				tau[i*j]=tau[i]*(cntl[i]+2)/(cntl[i]+1);cntl[i*j]=cntl[i]+1;
				mu[i*j]=0;
				break;
			}
		}
	}
	forup(i,1,n){
		sumtau[i]=sumtau[i-1]+tau[i];
		summu[i]=summu[i-1]+mu[i];
	}
}
void solve(){
	i64 n=read(),m=read();
	if(n>m) swap(n,m);
	i64 res=0;
	for(i64 l=1,r=0;l<=n;l=r+1){
		r=min(n/(n/l),m/(m/l));
		res+=(summu[r]-summu[l-1])*sumtau[n/l]*sumtau[m/l];
	}
	printf("%lld\n",res);
}
signed main(){
	i64 t=read();
	init(5e4);
	while(t--){
		solve();
	}
}
```

///

## P1829 [国家集训队] Crash的数字表格 / JZPTAB

[传送门](https://www.luogu.com.cn/problem/P1829)

> 题意

- 给定 $n,m$，求 $\sum_{i=1}^n\sum_{j=1}^m\mathrm{lcm}(i,j)$。
- 单组询问，$1\le n,m\le 10^7$。

> 题解

怎么所有人都觉得这道题很简单，我觉得很复杂啊。

考虑 $\mathrm{lcm}$ 不好做，但是有 $\mathrm{lcm}(i,j)=\frac{i\times j}{\gcd(i,j)}$，而 $\gcd$ 是好做一点的。

但是 $\gcd$ 在分母仍然不好搞，考虑令 $i'=\frac{i}{\gcd(i,j)},j'=\frac{j}{\gcd(i,j)}$，显然 $\gcd(i',j')=1$，并且有 $\frac{i\times j}{\gcd(i,j)}=\gcd(i,j)i'j'$，于是就把 $\gcd$ 提到分子了。

于是有（钦定 $n\le m$）：

$$
\begin{aligned}
&\sum_{i=1}^n\sum_{j=1}^m\mathrm{lca}(i,j)\\\\
=&\sum_{d=1}^n d\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{d}\right\rfloor}[\gcd(i,j)=1]ij\\\\
=&\sum_{d=1}^n d\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{d}\right\rfloor}ij\sum_{t\mid \gcd(i,j)}\mu(t)\\\\
=&\sum_{d=1}^n d\sum_{t=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\mu(t)\sum_{i=1}^{\left\lfloor\frac{n}{d\times t}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{d\times t}\right\rfloor}(i\cdot t)(j\cdot t)\\\\
=&\sum_{d=1}^n d\sum_{t=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\mu(t)t^2\left(\sum_{i=1}^{\left\lfloor\frac{n}{d\times t}\right\rfloor}i\right)\left(\sum_{j=1}^{\left\lfloor\frac{m}{d\times t}\right\rfloor}j\right)\\\\
\end{aligned}
$$

然后容易发现 $\sum_{t=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\mu(t)t^2\left(\sum_{i=1}^{\left\lfloor\frac{n}{d\times t}\right\rfloor}i\right)\left(\sum_{j=1}^{\left\lfloor\frac{m}{d\times t}\right\rfloor}j\right)$ 可以预处理 $\mu(t)t^2$ 的前缀和然后数论分块做，外层也能数论分块做。于是套两层数论分块，复杂度不会分析但是显然小于 $O(\sqrt{n}^2)=O(n)$。

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
const int N=1e7+5,inf=0x3f3f3f3f,mod=20101009;
int n,m,mu[N],vv[N],sum[N],summu[N];
vector<int> pri;
void init(int n){
	vv[1]=mu[1]=1;
	forup(i,2,n){
		if(!vv[i]){
			pri.push_back(i);
			mu[i]=-1;
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			vv[i*j]=1;
			if(i%j){
				mu[i*j]=-mu[i];
			}else{
				mu[i*j]=0;
				break;
			}
		}
	}
	forup(i,1,n){
		sum[i]=(sum[i-1]+i)%mod;
		summu[i]=(summu[i-1]+1ll*mu[i]*i*i%mod)%mod;
	}
}
int calcg(int d){
	int res=0,nn=n/d,mm=m/d;
	for(int l=1,r=0;l<=nn;l=r+1){
		r=min(nn/(nn/l),mm/(mm/l));
		(res+=1ll*(sum[r]-sum[l-1]+mod)*sum[nn/l]%mod*sum[mm/l]%mod)%=mod;
	}
	return res;
}
signed main(){
	n=read();m=read();
	if(n>m) swap(n,m);
	init(m);
	int ans=0;
	for(int l=1,r=0;l<=n;l=r+1){
		r=min(n/(n/l),m/(m/l));
		(ans+=1ll*(summu[r]-summu[l-1]+mod)*calcg(l)%mod)%=mod;
	}
	printf("%d\n",ans);
}
```

///

## P3768 简单的数学题

[传送门](https://www.luogu.com.cn/problem/P3768)

> 题意

- 给定 $n$,求 $\sum_{i=1}^n\sum_{j=1}^nij\gcd(i,j)$。
- 防止答案过大，答案模 $p$ 输出（其实对做法没有影响），$p$ 是质数。
- $1\le n\le 10^{10},5\times 10^8\le p\le1.1\times 10^9$

> 题解

还是比较套路的。

看到 $\gcd$ 想到往反演方向推：

$$
\begin{aligned}
&\sum_{i=1}^n\sum_{j=1}^nij\gcd(i,j)\\\\
=&\sum_{p=1}^np\sum_{p\mid i}\sum_{p\mid j}ij[\gcd(i/p,j/p)=1]\\\\
=&\sum_{p=1}^np\sum_{i=1}^{\left\lfloor\frac{n}{p}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{p}\right\rfloor}ip\times jp\times [\gcd(i,j)=1]\\\\
=&\sum_{p=1}^np^3\sum_{i=1}^{\left\lfloor\frac{n}{p}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{p}\right\rfloor}ij\times [\gcd(i,j)=1]\\\\
=&\sum_{p=1}^np^3\sum_{i=1}^{\left\lfloor\frac{n}{p}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{p}\right\rfloor}ij\sum_{d\mid \gcd(i,j)}\mu(d)\\\\
=&\sum_{p=1}^np^3\sum_{d=1}^{\left\lfloor\frac{n}{p}\right\rfloor}\mu(d)\sum_{i=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}id\times jd\\\\
=&\sum_{p=1}^np^3\sum_{d=1}^{\left\lfloor\frac{n}{p}\right\rfloor}d^2\mu(d)\sum_{i=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}i\sum_{j=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}j\\\\
=&\sum_{p=1}^np^3\sum_{d=1}^{\left\lfloor\frac{n}{p}\right\rfloor}d^2\mu(d)\left(\sum_{i=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}i\right)^2\\\\
=&\sum_{p=1}^np^3\sum_{d=1}^{\left\lfloor\frac{n}{p}\right\rfloor}d^2\mu(d)\left(\sum_{i=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}i^3\right)\\\\
=&\sum_{T=1}^n\left(\sum_{i=1}^{\left\lfloor\frac{n}{T}\right\rfloor}i^3\right)\sum_{p\mid T}p^3\left(\frac{T}{p}\right)^2\mu(\frac{T}{p})\\\\
=&\sum_{T=1}^n\left(\sum_{i=1}^{\left\lfloor\frac{n}{T}\right\rfloor}i^3\right)T^2\sum_{p\mid T}p\mu(\frac{T}{p})\\\\
=&\sum_{T=1}^n\left(\sum_{i=1}^{\left\lfloor\frac{n}{T}\right\rfloor}i^3\right)T^2(id\ast\mu)(T)\\\\
=&\sum_{T=1}^n\left(\sum_{i=1}^{\left\lfloor\frac{n}{T}\right\rfloor}i^3\right)T^2\varphi(T)\\\\
\end{aligned}
$$

其中 $\left(\sum_{i=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}i\right)^2=\sum_{i=1}^{\left\lfloor\frac{n}{dp}\right\rfloor}i^3$ 是经典小学奥数，我证不来。然后 $id\ast \mu=\varphi$ 是经典结论。

那么前面 $\sum_{i=1}^{\left\lfloor\frac{n}{T}\right\rfloor}i^3$ 的部分可以数论分块解决，问题在于我们需要求 $T^2\varphi(T)$ 的前缀和。因为 $n=10^{10}$ 肯定考虑筛法了。考虑杜教筛。

那么我们需要构造 $h=g\ast f$，其中 $f(i)=i^2\varphi(i)$，有经典结论 $id=\varphi\ast I$，即 $\sum_{d\mid n}\varphi(d)=n$，容易想到 $\sum_{d\mid n}d^2\varphi(d)(\frac{n}{d})^2=n^3$，即 $id^3=(id^2\times\varphi)\ast id^2$。而 $id^3$ 和 $id^2$ 的前缀和都能经典小学奥数求出，于是套个板子就做完了。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(i64 i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(i64 i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void();
#endif
using namespace std;
using i64=long long;
#define gc getchar()
i64 read(){
	i64 x=0,f=1;char c;
	while(!isdigit(c=gc)) if(c=='-') f=-1;
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
	return x*f;
}
#undef gc
const i64 N=1e7,inf=0x3f3f3f3f;
#define ull unsigned long long
#define ui128 __uint128_t
struct Barrett{
	ull d;ui128 m;
	void init(ull _d){
		d=_d;m=((ui128)(1)<<64)/d;
	}
};
Barrett mod;
ull operator %(const ull a,const Barrett mod){
	ull w=(mod.m*a)>>64;w=a-w*mod.d;
	if(w>=mod.d)w-=mod.d;return w;
}
i64 ksm(i64 a,i64 b){
	i64 c=1;
	while(b){
		if(b&1) c=1ll*a*c%mod;
		a=1ll*a*a%mod;
		b>>=1;
	}
	return c;
}
i64 n,p,phi[N+5],sumphi[N+5],inv6,inv4;
vector<i64> pri;
void init(i64 n){
	phi[1]=1;
	forup(i,2,n){
		if(!phi[i]){
			phi[i]=i-1;
			pri.push_back(i);
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			if(i%j){
				phi[i*j]=phi[i]*(j-1)%mod;
			}else{
				phi[i*j]=phi[i]*j%mod;
				break;
			}
		}
	}
	forup(i,1,n){
		sumphi[i]=(sumphi[i-1]+phi[i]*i%mod*i%mod)%mod;
	}
}
i64 sump2(i64 n){
	n=n%mod;
	return 1ll*n*(n+1)%mod*(2*n+1)%mod*inv6%mod;
}
i64 sump3(i64 n){
	n=n%mod;
	return n*n%mod*(n+1)%mod*(n+1)%mod*inv4%mod;
}
map<i64,i64> val;
i64 calc(i64 n){
	if(n<=N) return sumphi[n];
	if(val.count(n)) return val[n];
	i64 va=sump3(n);
	for(i64 l=2,r=0;l<=n;l=r+1){
		r=n/(n/l);
		va=(va+p-(sump2(r)-sump2(l-1)+p)%mod*calc(n/l)%mod)%mod;
	}
	val[n]=va;
	return va;
}
signed main(){
	p=read();n=read();
	mod.init(p);
	inv6=ksm(6,p-2);
	inv4=ksm(4,p-2);
	init(min(n,N));
	i64 res=0;
	for(i64 l=1,r=0;l<=n;l=r+1){
		r=n/(n/l);
		res=(res+p+(calc(r)+p-calc(l-1))%mod*sump3(n/l)%mod)%mod;
	}
	printf("%lld\n",res);
}
```

///

## P5325 【模板】Min_25 筛

> 题意

- 求 $\sum_{i=1}^nf(i)$，其中 $f(i)$ 是积性函数，满足 $f(p^k)=p^k(p^k-1)$。
- $1\le n\le 10^{10}$

> 题解

积性函数前缀和考虑 PN 筛（主要是我不会 Min_25 筛）。

那么先考虑构造积性函数 $g$ 满足 $g(p)=f(p)$。看到“积性”和“$p-1$”容易想到 $\varphi$。容易发现构造 $g(i)=i\varphi(i)$ 是符合条件的。然后考虑寻找 $h\ast g=f$。

容易发现 $g(p^k)=p^k\left(p^{k-1}(p-1)\right)=p^{2k-1}(p-1)$（根据欧拉函数通项公式。另外 $k=0$ 不满足这个，$k=0$ 时 $g(1)=1$）那么有 $f(p^k)=p^k(p^k-1)=h(p^k)+\sum_{i=1}^kp^{2i-1}(p-1)h(p^{k-i})$，则 $h(p^k)=p^k(p^k-1)-\sum_{i=1}^kp^{2i-1}(p-1)h(p^{k-i})$，手模一下，容易得到：

$$
\begin{aligned}
h(1)&=1\\\\
h(p)&=0\\\\
h(p^2)&=p^2(p^2-1)-p^{2\times 2-1}(p-1)h(1)\\\\
	  &=p^3-p^2\\\\
h(p^3)&=p^3(p^3-1)-p^{1\times 2-1}(p-1)h(p^2)-p^{3\times 2-1}(p-1)h(1)\\\\
	  &=p^3(p^3-1)-p(p-1)(p^3-p^2)-p^{5}(p-1)\\\\
	  &=2(p^4-p^3)\\\\
h(p^4)&=p^4(p^4-1)-p^{1\times 2-1}(p-1)h(p^3)-p^{2\times 2-1}(p-1)h(p^2)-p^{4\times 2-1}(p-1)h(1)\\\\
	  &=p^4(p^4-1)-p(p-1)2(p^4-p^3)-p^{3}(p-1)(p^3-p^2)-p^{7}(p-1)\\\\
	  &=3(p^5-p^4)\\\\
\end{aligned}
$$

大胆猜测 $h(p^i)=(i-1)(p^{i+1}-p^{i})$（除去 $i=0$ 时，$h(1)=1$），考虑证明：

考虑数学归纳法。

首先对于 $k=1$，$p^k$ 不是 Powerful Number，则 $h(p^k)=0$，符合条件。

考虑假设对于所有 $1\le i\le k-1$ 上述结论均成立，证明 $h(p^k)=(k-1)(p^{k+1}-p^k)$。对于 $k>1$，注意到（具体注意方法是发现后面每个 $\sum_{i=1}^kg(p^i)h(p^{k-i})$ 中，对于指数相同的 $h$，$g$ 的指数都变大 $2$，那么把第一项 $f(p^k)$ 去掉乘以 $p^2$ 后再加上新增的项）：

$$
\begin{aligned}
h(p^k)&=p^2(h(p^{k-1})-p^{k-1}(p^{k-1}-1))+p^k(p^k-1)-p(p-1)h(p^{k-1})\\\\
&=(p^2-p^2+p)h(k-1)-p^{k+1}(p^{k-1}-1)+p^k(p^k-1)\\\\
&=p\cdot h(k-1)-p^{2k}+p^{k+1}+p^{2k}-p^k\\\\
&=p(k-2)(p^k-p^{k-1})+p^{k+1}-p^{k}\\\\
&=(k-2)(p^{k+1}-p^k)+(p^{k+1}-p^k)\\\\
&=(k-1)(p^{k+1}-p^k)
\end{aligned}
$$

得证。

然后 $h$ 可以在找 Powerful Number 的时候顺便求出，瓶颈在于求 $g$ 的前缀和，考虑杜教筛（$g\ast id=id^2$）。找到所有 $\left\lfloor\frac{n}{i}\right\rfloor$ 处前缀和值的复杂度为 $O(n^{\frac{2}{3}})$。于是总复杂度 $O(n^{\frac{2}{3}})$。

注意 $n=1$ 的时候如果写得不好访问第一个质数会 RE，考虑特判或者多预处理几个质数。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(i64 i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(i64 i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void();
#endif
using namespace std;
using i64=long long;
#define gc getchar()
i64 read(){
	i64 x=0,f=1;char c;
	while(!isdigit(c=gc)) if(c=='-') f=-1;
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
	return x*f;
}
#undef gc
const i64 N=1e7,mod=1e9+7;
i64 n,sphi[N+5],phi[N+5],inv6,inv2;
i64 ksm(i64 a,i64 b){
	i64 c=1;
	while(b){
		if(b&1) c=a*c%mod;
		a=a*a%mod;
		b>>=1;
	}
	return c;
}
vector<i64> pri;
void init(i64 n){
	phi[1]=1;
	forup(i,2,n){
		if(!phi[i]){
			pri.push_back(i);
			phi[i]=i-1;
		}
		for(auto j:pri){
			if(i*j>n) break;
			if(i%j){
				phi[i*j]=phi[i]*(j-1)%mod;
			}else{
				phi[i*j]=phi[i]*j%mod;
				break;
			}
		}
	}
	forup(i,1,n){
		sphi[i]=(sphi[i-1]+i*phi[i]%mod)%mod;
	}
}
map<i64,i64> val;
i64 sump2(i64 n){
	n%=mod;
	return n*(n+1)%mod*((2*n+1)%mod)%mod*inv6%mod;
}
i64 calcsg(i64 n){
	if(n<=N) return sphi[n];
	if(val.count(n)) return val[n];
	i64 res=sump2(n);
	for(i64 l=2,r=0;l<=n;l=r+1){
		r=n/(n/l);
		res-=((r+l)%mod)*((r-l+1)%mod)%mod*inv2%mod*calcsg(n/l)%mod;
		if(res<0) res+=mod;
	}
	val[n]=res;
	return res;
}
i64 ans;
i64 calch(i64 pk,i64 p,i64 k){
	pk%=mod;p=(p-1)%mod;k=(k-1)%mod;
	return k*p%mod*pk%mod;
}
void dfs(i64 v,i64 h,i64 p){
	(ans+=h*calcsg(n/v)%mod)%=mod;
	for(i64 rr=n/v;pri[p]*pri[p]<=rr;++p){
		for(i64 c=2,nv=pri[p]*pri[p];nv<=rr;nv*=pri[p],++c){
			dfs(v*nv,h*calch(nv,pri[p],c)%mod,p+1);
		}
	}
}
signed main(){
	n=read();
	init(min(n+5,N));
	inv6=ksm(6,mod-2);inv2=ksm(2,mod-2);
	dfs(1,1,0);
	printf("%lld\n",ans);
}
```

///

## Loj#6053. 简单的函数

[传送门](https://loj.ac/p/6053)

> 题意

- 给定 $n$，求积性函数 $f$ 的前缀和 $\sum_{i=1}^nf(i)$。
- 其中 $f(1)=1,f(p^c)=p\oplus c$，$\oplus$ 表示按位异或。
-  $1\le n\le 10^{10}$

> 题解

因为我不会 Min_25 筛所以考虑 Powerful Number 筛。先来构造 $g\ast h=f$。

因为需要积性函数，又发现在奇质数的位置 $f(p)=p-1$，而偶数的位置有 $f(2)=3$，容易想到构造：

$$
g(n)=\begin{cases}
\varphi(n)&,n\bmod 2=1\\\\
3\varphi(n)&,n\bmod 2=0
\end{cases}
$$

容易发现这个也是积性函数。设 $a,b$ 互质，显然里面最多有一个是偶数。若两个都是奇数则显然正确。否则不妨钦定 $a$ 是偶数，那么 $g(ab)=3\varphi(ab)=3\varphi(a)\varphi(b)=g(a)g(b)$。于是积性。

这个怎么求前缀和呢？容易发现等于 $\varphi$ 的前缀和加上偶数位置的总和乘 $2$。不妨把偶数 $i$ 全部除以二。若 $\frac{i}{2}$ 不是偶数那么 $\varphi(i)=\varphi(\frac{i}{2})$，否则 $\varphi(i)=2\varphi(\frac{i}{2})$（考虑带 $\varphi$ 的通项公式）。而这就变成了一个子问题（只不过变成偶数位置不用乘二），可以递归求解。而容易发现这样需要求的前缀和的下标均是 $\left\lfloor\frac{n}{2^k}\right\rfloor$ 的形式。于是考虑杜教筛，这部分复杂度是 $O(n^{\frac{2}{3}}+\sqrt{n}\log^2 n)$，其中这个 $\sqrt{n}$ 是枚举 Powerful Number 的复杂度，$\log^2$ 是因为如果用 map 记忆化每次访问会带个 $\log$。

然后考虑 $h$。手模几组容易发现没有显著规律，那么只能暴力求 $h$ 了，复杂度是 $O(\sqrt{n}\log n)$。

综上，复杂度 $O(n^{\frac{2}{3}}+\sqrt{n}\log^2 n)$。

/// details | 参考代码
	open: False
	type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(i64 i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(i64 i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void();
#endif
using namespace std;
using i64=long long;
#define gc getchar()
i64 read(){
	i64 x=0,f=1;char c;
	while(!isdigit(c=gc)) if(c=='-') f=-1;
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
	return x*f;
}
#undef gc
const i64 N=1e7,inf=0x3f3f3f3f,mod=1e9+7,inv2=5e8+4;
i64 ksm(i64 a,i64 b){
	i64 c=1;
	while(b){
		if(b&1) c=1ll*a*c%mod;
		a=1ll*a*a%mod;
		b>>=1;
	}
	return c;
}
i64 n,phi[N+5],sphi[N+5];
vector<i64> pri;
void init(i64 n){
	phi[1]=1;
	forup(i,2,n){
		if(!phi[i]){
			pri.push_back(i);
			phi[i]=i-1;
		}
		for(auto j:pri){
			if(i*j>n) break;
			if(i%j){
				phi[i*j]=phi[i]*(j-1)%mod;
			}else{
				phi[i*j]=phi[i]*j%mod;
				break;
			}
		}
	}
	forup(i,1,n){
		sphi[i]=(sphi[i-1]+phi[i])%mod;
	}
}
map<i64,i64> val;
i64 calcsphi(i64 n){
	if(n<=N) return sphi[n];
	if(val.count(n)) return val[n];
	i64 res=(n%mod)*((n+1)%mod)%mod*inv2%mod;
	for(i64 l=2,r=0;l<=n;l=r+1){
		r=n/(n/l);
		res=(res+mod-(r-l+1)%mod*calcsphi(n/l)%mod)%mod;
	}
	val[n]=res;
	return res;
}
i64 calcG(i64 n){
	if(n==0) return 0;
	return (calcsphi(n)+calcG(n/2))%mod;
}
vector<i64> hp[10000];
i64 ans;
void dfs(i64 p,i64 h,i64 v){
	(ans+=h*(calcsphi(n/v)+2*calcG((n/v)/2)%mod))%=mod;
	for(i64 rr=n/v;pri[p]*pri[p]<=rr;++p){
		for(i64 e=2,vv=pri[p]*pri[p];vv<=rr;++e,vv*=pri[p]){
			dfs(p+1,h*hp[p][e]%mod,v*vv);
		}
	}
}
signed main(){
	n=read();
	init(min(n+5,N));
	for(i64 i=0;;++i){
		if(pri[i]>n/pri[i]) break;
		hp[i].push_back(1);hp[i].push_back(0);
		for(i64 j=pri[i]*pri[i],k=2;j<=n;j=j*pri[i],++k){
			i64 nh=pri[i]^k,nv=pri[i]-1;
			fordown(l,k-1,0){
				nh=(nh+mod-hp[i][l]*nv%mod*(1+2*(pri[i]==2))%mod)%mod;
				nv=nv*pri[i]%mod;
			}
			hp[i].push_back(nh);
		}
	}
	dfs(0,1,1);
	printf("%lld\n",ans);
}
```

///

## P3704 [SDOI2017] 数字表格

[传送门](https://www.luogu.com.cn/problem/P3704)

> 题意

- 给定 $n,m$，求 $\prod_{i=1}^n\prod_{j=1}^mf(\gcd(i,j))$。
- 其中 $f_n$ 表示斐波那契数列的第 $n$ 项。此处定义 $f(0)=0,f(1)=1$，后面每个数等于它前面两个数相加。
- $1\le n,m\le 10^6$，多组询问，询问数量 $1\le T\le 1000$

> 题解

想了一下午带通项公式乘起来无果瞄了一眼题解发现可以直接莫反，破防了。

考虑化式子（钦定 $n\le m$）：

$$
\begin{aligned}
&\prod_{i=1}^n\prod_{j=1}^mf(\gcd(i,j))\\\\
=&\prod_{d=1}^nf(d)^{\sum_{i=1}^n\sum_{j=1}^m[\gcd(i,j)=d]}\\\\
=&\prod_{d=1}^nf(d)^{\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{d}\right\rfloor}[\gcd(i,j)=1]}\\\\
=&\prod_{d=1}^nf(d)^{\sum_{t=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\mu(t)\left\lfloor\frac{n}{d\cdot t}\right\rfloor\left\lfloor\frac{m}{d\cdot t}\right\rfloor}
\end{aligned}
$$

看起来化不了了。根据套路，把 $d\cdot t$ 乘在一起枚举：

$$
\begin{aligned}
=&\sum_{p=1}^n\sum_{d\mid p}f(d)^{\mu(\frac{p}{d})\left\lfloor\frac{n}{p}\right\rfloor\left\lfloor\frac{m}{p}\right\rfloor}\\\\
=&\sum_{p=1}^n\left(\sum_{d\mid p}f(d)^{\mu(\frac{p}{d})}\right)^{\left\lfloor\frac{n}{p}\right\rfloor\left\lfloor\frac{m}{p}\right\rfloor}\\\\
\end{aligned}
$$

容易发现中间这坨可以暴力预处理，枚举每个数和它的倍数即可，预处理复杂度 $O(n\log n)$。

然后单次询问可以数论分块 $+$ 快速幂，复杂度 $O(n\log n+T\sqrt{n}\log n)$。

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
const int N=1e6+5,inf=0x3f3f3f3f,mod=1e9+7;
int ksm(int a,int b){
	int c=1;
	while(b){
		if(b&1) c=1ll*a*c%mod;
		a=1ll*a*a%mod;
		b>>=1;
	}
	return c;
}
int n,m;
int mu[N],f[N],vv[N],val[N],sum[N],sinv[N];
vector<int> pri;
void init(int n){
	mu[1]=vv[1]=f[1]=1;
	forup(i,2,n){
		f[i]=(f[i-1]+f[i-2])%mod;
		if(!vv[i]){
			pri.push_back(i);
			mu[i]=-1;
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			vv[i*j]=1;
			if(i%j){
				mu[i*j]=-mu[i];
				vv[i*j]=1;
			}else{
				mu[i*j]=0;
				vv[i*j]=1;
				break;
			}
		}
	}
	forup(i,1,n) val[i]=1;
	forup(i,1,n){
		int inv=ksm(f[i],mod-2);
		forup(j,1,n/i){
			if(mu[j]==1){
				val[i*j]=1ll*val[i*j]*f[i]%mod;
			}else if(mu[j]==-1){
				val[i*j]=1ll*val[i*j]*inv%mod;
			}
		}
	}
	sum[0]=1;
	forup(i,1,n){
		sum[i]=1ll*val[i]*sum[i-1]%mod;
	}
	sinv[n]=ksm(sum[n],mod-2);
	fordown(i,n-1,1){
		sinv[i]=1ll*sinv[i+1]*val[i+1]%mod;
	}
	sinv[0]=1;
}
void solve(){
	n=read();m=read();
	if(n>m) swap(n,m);
	int ans=1;
	for(int l=1,r=0;l<=n;l=r+1){
		r=min(n/(n/l),m/(m/l));
		ans=1ll*ans*ksm(1ll*sum[r]*sinv[l-1]%mod,1ll*(n/l)*(m/l)%(mod-1))%mod;
	}
	printf("%d\n",ans);
}
signed main(){
	init(1e6);
	int t=read();
	while(t--){
		solve();
	}
}
```

///


## P6271 [湖北省队互测2014] 一个人的数论

[传送门](https://www.luogu.com.cn/problem/P6271)

这道题让我重新认识莫反。

> 题意

- 给定 $n,d$，求小于 $n$ 且与 $n$ 互质的数的 $d$ 次方之和。
- 形式化地，求 $G(n)=\sum_{i=1}^ni^d[\gcd(n,i)=1]$
- 本题中 $n$ 以质因数分解 $\prod_{i=1}^w p_i^{a_i}$ 给出
- $1\le p_i,a_i\le 10^9,1\le w\le 1000,1\le d\le 100$，保证 $p$ 是质数。

> 题解

之前 zmy 看到这道题的时候跟 XD 说，XD 没看数据范围说他会多项式做法，然后看了一眼数据范围说不会了。

然后 lh 看到了直接秒了，这就是银牌爷的实力吗。

看到 $[\gcd=1]$ 考虑直接套路化式子：

$$
\begin{aligned}
G(n)&=\sum_{i=1}^ni^d[\gcd(n,i)=1]\\\\
	&=\sum_{i=1}^ni^d\sum_{k\mid \gcd(n,i)}\mu(k)\\\\
	&=\sum_{k\mid n}\mu(k)\sum_{k\mid i}^{i\le n}i^d\\\\
	&=\sum_{k\mid n}\mu(k)k^d\sum_{i=1}^{\frac{n}{d}}i^d
\end{aligned}
$$

根据经典结论，自然数幂的前缀和（即形如 $\sum_{i=1}^ni^k$ 的东西）是一个 $k+1$ 次多项式，于是我们可以把 $\sum_{i=1}^{\frac{n}{d}}i^d$ 写成 $\sum_{i=0}^{d+1}f_i(\frac{n}{d})^i$，其中 $f$ 待定，为多项式系数。

$$
\begin{aligned}
&=\sum_{k\mid n}\mu(k)k^d\sum_{i=0}^{d+1}f_i(\frac{n}{d})^i\\\\
&=\sum_{i=0}^{d+1}f_in^i\sum_{k\mid n}\mu(k)k^{d-i}
\end{aligned}
$$

显然 $\mu(k)k^{d-i}$ 是一个积性函数，那么考虑把 $k$ 质因数分解为 $\prod_{j=1}^w p_j^{e_j}$，$p,w$ 即题目输入。

$$
\begin{aligned}
&=\sum_{i=0}^{d+1}f_in^i\sum_{k\mid n}\prod_{j=1}^w\mu(p_j^{e_j})p_j^{e_j(d-i)}\\\\
&=\sum_{i=0}^{d+1}f_in^i\prod_{j=1}^w\sum_{e=0}^{a_j}\mu(p_j^e)p_j^{e(d-i)}
\end{aligned}
$$

这里的 $a_j$ 还是题上给的。这一步转化考虑乘法分配律即可。注意到 $\mu(p^e)$ 有非常好的性质，具体来说，它在 $e=0$ 时取 $1$，$e=1$ 时取 $-1$，$e>1$ 时取 $0$，那么其实 $e$ 只需要枚举到 $1$ 即可，而显然 $a_j$ 全都大于等于 $1$，则：

$$
\begin{aligned}
&=\sum_{i=0}^{d+1}f_in^i\prod_{j=1}^w(1-p_j^{d-i})
\end{aligned}
$$

那么我们只要求出 $f_i$ 就能暴力 $O(wd)$ 得到答案了。

求 $f_i$ 考虑插值即可。具体来说，我们随便选 $d+2$（注意本题是 $d+1$ 次多项式）个点求出点值 $(x_i,y_i)$，则多项式 $F(x)=\sum_{i=1}^{d+2}y_i\prod_{j\ne i}\frac{x-x_j}{x_i-x_j}$，即 $d+2$ 个只有对应点值处取 $y$，其余地方取 $0$（相当于方程的根）的多项式相加，直接暴力是 $O(d^3)$ 的，随便过的，也可以预处理多项式 $\prod x-x_i$，每次除一个多项式乘一个常数即可做到 $O(d^2)$。

复杂度 $O(d^3+wd)$。

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
#define msg(...) void();
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
const int mod=1e9+7;
int ksm(int a,int b){
    int c=1;
    while(b){
        if(b&1) c=1ll*a*c%mod;
        a=1ll*a*a%mod;
        b>>=1;
    }
    return c;
}
int w,d,n,pw[1005][105],inv[1005],ans;
int f[105],nw[105],nxt[105];
void getf(int x,int y){
    mem(nw,0);mem(nxt,0);
    nw[0]=1;
    int p=y;
    forup(i,0,d+1){
        if(i==x) continue;
        p=1ll*p*ksm((x-i+mod)%mod,mod-2)%mod;
        forup(j,0,d+1){
            nxt[j]=1ll*(mod-i)*nw[j]%mod;
            if(j) (nxt[j]+=nw[j-1])%=mod;
        }
        forup(j,0,d+1){
            nw[j]=nxt[j];
            nxt[j]=0;
        }
    }
    forup(i,0,d+1){
        (f[i]+=1ll*nw[i]*p%mod)%=mod;
    }
}
signed main(){
    d=read();w=read();
    int y=0;
    forup(i,0,d+1){
        (y+=ksm(i,d))%=mod;
        getf(i,y);
    }
    int n=1;
    forup(i,1,w){
        int p=read(),a=read();
        n=1ll*n*ksm(p,a)%mod;
        inv[i]=ksm(p,mod-2);
        pw[i][0]=1;
        forup(j,1,d+1){
            pw[i][j]=1ll*pw[i][j-1]*p%mod;
        }
    }
    int pn=1;
    forup(i,0,d+1){
        int val=1ll*f[i]*pn%mod;
        pn=1ll*pn*n%mod;
        forup(j,1,w){
            if(i<=d) val=1ll*val*(mod+1-pw[j][d-i])%mod;
            else val=1ll*val*(mod+1-inv[j])%mod;
        }
        (ans+=val)%=mod;
    }
    printf("%d\n",ans);
}
```

///