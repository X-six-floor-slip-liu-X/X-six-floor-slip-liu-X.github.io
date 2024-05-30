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