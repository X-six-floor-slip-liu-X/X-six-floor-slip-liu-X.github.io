---
comments: true
---

# 反演与筛法相关题目

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

考虑 $a^2+b^2=c^2\Leftrightarrow a^2=(b+c)(b-c)$，那么对于一个 $a$，每个 $d$ 满足 $d\mid a^2,d\equiv \frac{a}{d}\pmod 2,d\ne a$ 都能对应一个 $b-c$（或者 $b+c$，显然同一个情况会在 $b-c,b+c$ 各算一次，除以二即可）。其中 $d\equiv \frac{a}{d}\pmod 2$ 是保证 $b,c$ 是整数，其余两个限制都显然。

设 $\tau(x)$（`\tau`） 表示 $x$ 的约数个数。容易想到 $a$ 的贡献就是 $\frac{\tau(a^2)-1}{2}$，其中 $-1$ 是去掉 $d=a$ 的情况。

但这个没有考虑 $d,\frac{a}{d}$ 奇偶性不同的情况。如果 $a$ 是奇数，那么这两各东西的奇偶性必然相同，可以直接用上面那个式子。但当 $a$ 是偶数时，必须两边各分至少一个 $2$，那么不妨直接将两边均除以 $2$ 来计算，这样就能避免奇偶性的限制了，于是贡献为 $\frac{\tau(\frac{a^2}{4})-1}{2}$。

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