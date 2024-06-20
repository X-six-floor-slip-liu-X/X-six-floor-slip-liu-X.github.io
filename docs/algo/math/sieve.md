---
comments: true
---

# 筛法入门

神秘算法。

## 符号与约定

约定 $p$ 表示素数，$\prod_i p_i^{c_i}$ 表示某个数的质因数分解。

$f\ast g$ 表示 $f$ 与 $g$ 的狄利克雷卷积。大写字母 $F(i)$ 表示对应小写字母 $f(i)$ 的前缀和，即 $F(n)=\sum_{i=1}^nf(i)$（同理，$G(i),g(i)$ 等也有类似关系）。

$[]$ 为艾佛森括号，表示若内部表达式为真，则值为 $1$，否则为 $0$。

## 线性筛

大部分人最早接触的筛法，最初级的用法是用来筛质数。

事实上一般还是线性筛用的最多。

### 质数

考虑一个合数含有至少两个质因子，那么从小到大枚举质数，枚举它的所有倍数打一个合数标记。后面每次遇到没标记的就是质数进行，否则就是合数。这个是埃式筛，根据神秘证明复杂度是 $O(n\log \log n)$。

如果每个合数只被标记一次，复杂度就能缩到 $O(n)$ 了。一个想法是让每个合数只被它的最小质因子标记一次。那么先枚举 $i$，再枚举之前找到的质数 $p$，给 $ip$ 打标记。如果第一次遇到 $i\bmod p=0$ 说明现在 $p$ 就是 $i$ 的最小质因子，如果再往后枚举 $p$，那么 $ip$ 就不是被最小质因子（即 $i$ 的最小质因子）标记的。所以遇到 $i\bmod p=0$ 就退出循环即可。

### 求积性函数的值

这玩意只用来筛素数也太浪费了。容易发现每次 $i,p$ 要么互质，要么 $p$ 是 $i$ 的最小质因子。那么对于一个积性函数 $f(i)$，前一种情况有 $f(ip)=f(i)f(p)$，后一种情况有 $f(ip)=f(\frac{i}{p^c})f(p^{c+1})$，其中 $c$ 是 $i$ 中 $p$ 的指数。

多数情况下如果要计算积性函数一般 $f(p^c)$ 是好求的。于是可以记录每个数最小质因子的指数以及最小质因子以外部分的 $f$ 值，就能比较方便地求任意积性函数了。复杂度 $O(n)$，因为每次访问下一个数是 $O(1)$ 的，每个数至多访问一遍。

### 求欧拉函数/莫比乌斯函数的值

这两个也是积性函数，但是拎出来单独说一下。

欧拉函数比较特殊，根据通项公式 $\varphi(n)=\prod_{i}p_i^{c_i-1}(p_i-1)$ 有 $\varphi(p^c)=p\cdot \varphi(p^{c-1})$，于是在 $i\bmod p=0$ 时可以直接计算。

莫比乌斯函数就更特殊了，因为在某质因子质数大于 $1$ 时值取 $0$，所以遇到 $i\bmod p=0$ 时 $\mu(ip)=0$。

## 杜教筛

难度陡然上升。

为什么杜教筛叫筛呢？（我个人理解）注意到线性筛是通过更小的多个 $f(n)$ 用较小的复杂度计算出更大的单个 $f(n)$，虽然真正计算过的函数值变多了，但是你真正算出 $f(n)$ 所用的时间变少了。杜教筛也是如此，通过多计算几个值的方式降低整体复杂度。

### 主要思路

考虑一对数论函数 $f,g$，有如下关系：

$$
\sum_{i=1}^n(f\ast g)(i)=\sum_{i=1}^n\sum_{xy=i}g(x)f(y)=\sum_{i=1}^ng(i)F(\left\lfloor\frac{n}{i}\right\rfloor)
$$

因为每一对 $xy\le n$ 都会产生 $g(x)f(y)$ 的贡献，假如把前缀和拆开就会发现上述式子成立。

于是有：

$$
\begin{aligned}
\sum_{i=1}^n(f\ast g)(i)&=\sum_{i=1}^ng(i)F(\left\lfloor\frac{n}{i}\right\rfloor)\\\\
\sum_{i=1}^n(f\ast g)(i)-\sum_{i=2}^{n}g(i)F(\left\lfloor\frac{n}{i}\right\rfloor)&=g(1)F(n)\\\\
\frac{\sum_{i=1}^n(f*g)(i)-\sum_{i=2}^{n}g(i)F(\left\lfloor\frac{n}{i}\right\rfloor)}{g(1)}&=F(n)\\\\
\end{aligned}
$$

假如 $f\ast g$ 和 $g$ 的前缀和都好算（可以 $O(1)$ 或者低复杂度算），那么对前面这一堆数论分块，就能转化成 $O(\sqrt{n})$ 个子问题。

于是得到了杜教筛的基本思路。构造 $g$ 使得 $f\ast g$ 和 $g$ 的前缀和都好求。

注意到 $\left\lfloor\frac{\left\lfloor\frac{a}{n}\right\rfloor}{m}\right\rfloor=\left\lfloor\frac{a}{nm}\right\rfloor$，所以需要计算的 $F(n')$ 都是 $\left\lfloor\frac{n}{i}\right\rfloor$ 中出现过的。那么全部取出来从小往大计算即可（或者记忆化）。

复杂度证不来，就是 $\left\lfloor\frac{n}{i}\right\rfloor$ 的每个取值开根号加起来，可以证明是 $O(n^{\frac{3}{4}})$ 的。

一种优化是（在能做的情况下）直接预处理 $F(i)$ 到 $n^{\frac{2}{3}}$（一般来说 $f$ 是积性函数，可以线性筛），然后对于小的直接引用，大的做上面的方法，可以证明复杂度是 $O(n^{\frac{2}{3}})$ 的。

### 莫比乌斯函数

最常用的一集。

有经典式子 $\varepsilon=\mu\ast I$。而 $\varepsilon$ 和 $I$ 的前缀和都巨好求无比（$\sum_{i=1}^n\varepsilon(i)=1,\sum_{i=1}^nI(i)=n$）。于是可以直接算。

### 欧拉函数

怎么哪都是你们两个。

有经典式子 $\varphi=id\ast \mu$。故 $id=\varphi \ast I$，于是直接套板子。

### 其他函数

很随缘啊，取决于能不能构造出 $g$。

但是不要局限于筛。假如你能构造出 $g,h$ 满足 $f=g\ast h$，而 $g,h$ 都好求前缀和，其实可以直接用 $\sum_{i=1}^n(f\ast g)(i)=\sum_{i=1}^ng(i)F(\left\lfloor\frac{n}{i}\right\rfloor)$ 直接数论分块算，复杂度还是 $O(n^{\frac{1}{2}})$ 的。

/// details | [模板题](https://www.luogu.com.cn/problem/P4213)参考代码
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
using pii=pair<i64,i64>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
i64 read(){
	i64 x=0,f=1;char c;
	while(!isdigit(c=gc)) if(c=='-') f=-1;
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
	return x*f;
}
#undef gc
const i64 N=1e7+5,inf=0x3f3f3f3f;
i64 n,mu[N],summu[N],phi[N],sumphi[N];
vector<i64> pri;
void init(i64 n){
	mu[1]=1;phi[1]=1;
	forup(i,2,n){
		if(!phi[i]){
			phi[i]=i-1;mu[i]=-1;
			pri.push_back(i);
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			if(i%j){
				phi[i*j]=phi[i]*(j-1);
				mu[i*j]=-mu[i];
			}else{
				phi[i*j]=phi[i]*j;
				mu[i*j]=0;
				break;
			}
		}
	}
	forup(i,1,n){
		summu[i]=summu[i-1]+mu[i];
		sumphi[i]=sumphi[i-1]+phi[i];
	}
}
map<i64,pii> sum;
pii calc(i64 n){
	if(n<=1e7){
		return mkp(summu[n],sumphi[n]);
	}
	if(sum.count(n)) return sum[n];
	i64 smu=1,sphi=1ll*(1+n)*n/2;
	for(i64 l=2,r=0;l<=n;l=r+1){
		r=n/(n/l);
		pii res=calc(n/l);
		smu-=(r-l+1)*res.fi;
		sphi-=(r-l+1)*res.se;
	}
	smu/=1;sphi/=1;//象征性除以 1，防止复制板子的时候忘了。
	sum[n]=mkp(smu,sphi);
	return mkp(smu,sphi);
}
void solve(){
	n=read();
	pii res=calc(n);
	printf("%lld %lld\n",res.se,res.fi);
}
signed main(){
	init(1e7);
	i64 t=read();
	while(t--){
		solve();
	}
}
```

///

## Powerful Number 筛（PN 筛）

PN 筛用于亚线性地求解一些积性函数的前缀和。

### Powerful Number

~~Poweful Number，Number 中的 Number，数字中的数字，数字中的征服者，数字的王~~

若一个正整数 $n=\prod p_i^{c_i}$，$\forall i$ 均有 $c_i>1$，则称 $n$ 为一个 Powerful Number（注意 $1$ 也满足这个条件）。

显然所有 PowerFul Number 都能写成 $a^2b^3$，考虑偶数就全放 $a$ 里，奇数就拆三个放 $b$ 里。

那么 $n$ 以内的 Powerful Number1 个数就是 $\sum_{a=1}^{\sqrt{n}}\sqrt[3]{\frac{n}{a^2}}$。根据一些我不会的证明这玩意是 $O(\sqrt{n})$ 级别的。

### PN 筛

首先我们需要构造一个好求前缀和的积性函数 $g$，满足对于所有质数 $p\le n$ 均有 $g(p)=f(p)$（非质数点位不作要求，比如若 $f$ 在质数位置都取 $1$ 时，$g$ 可能可以取 $I$）。

考虑构造一个 $h\ast g=f$。根据狄利克雷卷积的性质，有 $h(1)=1$ 且 $h$ 也是积性函数。下面探讨一些 $h$ 的性质。

对于任意质数 $p$，显然有 $f(p)=h(p)g(1)+h(1)g(p)$。由于 $g(1)=h(1)=1$（非 $0$ 积性函数的性质），则 $f(p)=h(p)+g(p)$。又由于 $g(p)=f(p)$，所以 $h(p)=0$。而注意到 $h$ 为积性函数，容易发现在非 Powerful Number 的位置均有 $h(i)=0$（否则就会带一个一次质因子，根据积性 $h(i)=0$）。

关于 $h$ 的构造，因为它是积性函数，所以其实也只需要考虑 $p^k$ 就行了。能求出 $h$ 的通项公式当然好，但如果求不出来也有办法。考虑 $f(p^k)=\sum_{i=0}^kh(p^i)g(p^{k-i})$，所以 $h(p^k)=f(p^k)-\sum_{i=0}^{k-1}h(p^i)g(p^{k-i})$。又因为 $O(\sqrt{n})$ 内指数数量约为 $O(\frac{\sqrt{n}}{\log n})$ 级别的，素数的幂次上界是 $\log n$ 的。所以直接暴力复杂度是 $O(\frac{\sqrt{n}}{\log n}\log^2n)=O(\sqrt{n}\log n)$，还是非常小的，而且这个上界显然非常松吧。另外空间复杂度是 $O(\frac{\sqrt{n}}{\log n}\log n)=O(\sqrt{n})$ 的。当然如果能求出 $h$ 的通项公式那么复杂度会低一点。

由于 $F(n)=\sum_{i=1}^n(h\ast g)(i)=\sum_{i=1}^nh(i)G(\left\lfloor\frac{n}{i}\right\rfloor)$，而 $h$ 只在 PN 处有值。则大力搜索 $i$（枚举 $\sqrt{n}$ 以内的质数再从 $2$ 开始枚举其幂次）然后乘上 $g$ 的前缀和即可。复杂度 $O((p+q)\sqrt{n})$，其中 $p,q$ 分别是求 $g$ 前缀和的复杂度和求 $h$ 单点值的复杂度。

## 结语

感觉学了杜教筛和 PN 筛就差不多了啊。

这篇博客提到的筛法的核心是构造 $g,h$ 函数（除了线性筛），这个就和做几何题一样，想到了就简单，没想到就暴毙，感觉很考验经验啊。