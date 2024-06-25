---
comments: true
---

# 多项式基础

[参考资料 1](https://www.luogu.com/article/qdzrwqrd)

[参考资料 2](https://xxeray.gitlab.io/post/generating-function-lecture-notes)

## 符号与约定

所有操作均默认是在模意义 $\pmod{998244353}$ 下进行的，因为 NTT 需要模这个数。

称多项式“项数”为其次数加一（因为还有 $x^0$ 这一项）。

用 $F[i]$ 表示多项式 $F(x)$ 中 $x^i$ 这一项的系数。

默认 $n$ 代表某多项式的项数。

很多时候都用 $n'$ 表示一个和 $n$ 相关的代数对象，但是因为在函数中这个符号会和导数产生歧义，故用 $F_{\ast}(x)$ 表示一个和 $F(x)$ 相关的函数。

## 多项式四则运算

- 加减法

多项式的加减法定义为：

$$
F(x)\pm G(x)=\sum_{i\ge 0}(F[i]\pm G[i])x^i
$$

就是对应项系数相加减，可以直接 $O(n)$。

- 乘法

多项式乘法的定义为：

$$
F(x)\times G(x)=\sum_{i\ge 0}\sum_{j\ge 0}F[i]G[j]x^{i+j}=\sum_{i\ge 0}x^i\sum_{j=0}^i F[j]G[i-j]
$$

其实就是**加法卷积**。直接暴力是 $O(n^2)$ 的。但是可以用 NTT 优化到 $O(n\log n)$，移步[这里](./FFT_NTT.md)阅读。你在这里看到乘法的时候应该反应过来需要用 NTT 实现。

需要注意的是 $n$ 项多项式乘以 $m$ 项多项式会得到 $n+m$ 项多项式，这点在后面会再次说明。

- 除法

参见后文“多项式求逆”。

- 关于运算的合理性

本文所讨论的多项式均为**形式幂级数**，但限于篇幅，并没有给出其严格定义和应用。

它只是辅助分析的数学工具，所谓多项式中的“未知数”并没有实际意义，只是一个**代数对象**。比如在生成函数中我们用代数对象的幂次来标记数列的下标。

所以我们其实只关注多项式的系数，故可以用**数列加法**和**加法卷积**来定义多项式的运算。

- 界

多数时候，我们只关注多项式的前若干项，所以我们通常给多项式设置了一个次数上界，称为**界**，相当于 $\bmod\; x^n$，表示我们只关注 $x^0$ 项到 $x^{n-1}$ 项。

由于多项式加法不会对代数对象的幂次产生影响，故显然有：

$$
(F(x)\bmod x^n+G(x)\bmod x^n)\bmod x^n=(F(x)+G(x))\bmod x^n
$$

而由于乘法只会从低次项向高次项转移，故也有：

$$
((F(x)\bmod x^n)\times (G(x)\bmod x^n))\bmod x^n=(F(x)\times G(x))\bmod x^n
$$

但是如刚才所说，注意 $(F(x)\bmod x^n)\times (G(x)\bmod x^n)$ 有 $2n-1$ 项，在 NTT 时应该把 $n'$ 设为 $2n-1$ 而非 $n$，不要产生像计数题里面对某个数取模就不用担心爆 `long long` 种惯性思维。

- 多项式的复合

$$
F(G(x))=\sum_{i\ge 0}F[i]G(x)^{i}
$$

就是把 $F$ 中的所有 $x$ 换成 $G(x)$。

具体实现视情况而定。

- 多项式的求导

$$
F'(x)=\sum_{i\ge 1}F[i]ix^{i-1}
$$

相当于把每个 $x^i$ 换成 $i\cdot x^{i-1}$，和函数的求导是一样的。

- 多项式的积分

$$
F(x)=C+\sum_{i\ge 1}\frac{F'[i]x^{i+1}}{i}
$$

其中 $C$ 是常数项。

容易发现其实就是求导的逆运算。

容易发现求导和求积都能 $O(n)$ 算。

## 多项式进阶运算

### 泰勒展开

设 $F^{(i)}(x)$ 表示 $F(x)$ 的 $i$ 阶导，有：

$$
F(x)=\sum_{i\ge 0}\frac{F^{(i)}(x_0)}{i!}(x-x_0)^i
$$

其中 $x_0$ 是你任意取的一个数。

这个东西可以把一些神秘式子换成多项式，在一些证明中非常有用。

具体推导十分复杂，此处略过。

### 牛顿迭代法

一个研究多项式时非常好用的工具，若要阅读后文请确保你理解了这一段。

对于一个已知函数 $F(x)$，若存在一个与它相关的函数 $H(x)$（此处“相关”指存在 $G(H)=0$，且 $G$ 的定义式中含有 $F$，具体例子见后文。其中 $G(H)$ 表示多项式的复合，严谨一点的写法应该是 $G(H(x))$，但是这样看起来太臃肿了），我们可以用牛顿迭代求出 $G(x)\pmod{x^n}$。

有：

$$
H\equiv H_{\ast}-\frac{G(H_{\ast})}{G'(H_{\ast})}\pmod{x^n}
$$

其中 $H_{\ast}$ 满足 $G(H_{\ast})\equiv 0\pmod{x^{\frac{n}{2}}}$，需要钦定 $n$ 是偶数。

注意这里的 $G'(H_{\ast})$ 是以 $H_{\ast}$ 为主元的，即 $\frac{\operatorname{d} G(H_*(x))}{\operatorname{d} H_*(x)}$。

/// admonition | 证明
    type: quote

根据定义，显然有 $H(x)\equiv H_{\ast}(x)\pmod{x^{\frac{n}{2}}}$。将 $G(H)$ 在 $H_{\ast}(x)$ 处泰勒展开：

$$
G(H)=\sum_{i\ge 0}\frac{G^{(i)}(H_{\ast})}{i!}(H-H_{\ast})^i
$$

注意到 $H$ 与 $H_{\ast}$ 的前 $\frac{n}{2}$ 项相同。于是 $H-H_{\ast}$ 就只有后 $\frac{n}{2}$ 项。那么容易发现当 $c\ge 2$ 时，有 $(H-H_{\ast})^c\equiv 0\pmod{x^n}$。于是上面这个式子就只有前两项有用了，于是有：

$$
\begin{aligned}
G(H)&\equiv G(H_{\ast})+G'(H_{\ast})(H-H_{\ast}) &\pmod{x^n}\\\\
G'(H_{\ast})H&\equiv G(H)-G(H_{\ast})+G'(H_{\ast})H_{\ast} &\pmod{x^n}\\\\
H&\equiv \frac{G(H)-G(H_{\ast})+G'(H_{\ast})H_{\ast}}{G'(H_{\ast})} &\pmod{x^n}\\\\
H&\equiv H_{\ast}+\frac{G(H)-G(H_{\ast})}{G'(H_{\ast})} &\pmod{x^n}\\\\
H&\equiv H_{\ast}+\frac{G(H)-G(H_{\ast})}{G'(H_{\ast})} &\pmod{x^n}\\\\
\end{aligned}
$$

欸！我们注意到 $G(H)\equiv 0\pmod{x^n}$，于是：

$$
\begin{aligned}
H&\equiv H_{\ast}-\frac{G(H_{\ast})}{G'(H_{\ast})} &\pmod{x^n}\\\\
\end{aligned}
$$

需要注意 $\frac{1}{G'(H_{\ast})}$ 只需要求出 $\bmod\; x^{\frac{n}{2}}$ 的答案就行了。因为 $G(H_{\ast})$ 的前 $\frac{n}{2}$ 项必为 $0$，那么 $\frac{1}{G'(H_{\ast})}$ 任何一个超出 $\frac{n}{2}$ 的项和 $G(H_{\ast})$ 乘起来都会超过 $x^n$ 的界。

///

### 多项式求逆

注意到我们没有为多项式定义“除法”，事实上，我们需要用乘法逆元来实现这个东西。

具体来说，定义多项式的单位元为 $1$（即只有常数项系数为 $1$，其余系数均为 $0$ 的多项式），那么对于一个多项式 $F$，用 $F^{-1}$ 来表示它的逆元，代表 $F\times F^{-1}=1$。

容易构造出 $G(H)=F\times H-1$，这样当 $H=F^{-1}$ 时 $G$ 的值就能取 $0$ 了。

根据牛顿迭代的公式：

$$
\begin{aligned}
F^{-1}&\equiv F_{\ast}^{-1}-\frac{G(F_{\ast}^{-1})}{G'(F_{\ast}^{-1})} &\pmod{x^n}\\\\
F^{-1}&\equiv F_{\ast}^{-1}-\frac{F\times F_{\ast}^{-1}-1}{G'(F_{\ast}^{-1})} &\pmod{x^n}
\end{aligned}
$$

注意这个 $G'(F_{\ast}^{-1})$ 是以 $F_{\ast}^{-1}$ 为主元的，所以可以把 $F_{\ast}^{-1}$ 看作代数对象，那么 $F$ 就是一次项系数，$-1$ 就是常数项。所以求导之后只剩下一次项系数，故 $G'(F_{\ast}^{-1})=F$，于是上式可以转化为：

$$
\begin{aligned}
F^{-1}&\equiv F_{\ast}^{-1}-\frac{F\times F_{\ast}^{-1}-1}{F} &\pmod{x^n}\\\\
F^{-1}&\equiv F_{\ast}^{-1}-F^{-1}(F\times F_{\ast}^{-1}-1) &\pmod{x^n}
\end{aligned}
$$

刚才我们说过，这个 $F^{-1}$ 只需要取前 $\frac{n}{2}$ 位，那容易发现这就是 $F_{\ast}^{-1}$，于是就能把上面的东西化成非常优美的形式：

$$
\begin{aligned}
F^{-1}&\equiv F_{\ast}^{-1}-F_{\ast}^{-1}(F\times F_{\ast}^{-1}-1) &\pmod{x^n}\\\\
F^{-1}&\equiv F_{\ast}^{-1}(2-F\times F_{\ast}^{-1}) &\pmod{x^n}\\\\
\end{aligned}
$$

然后 NTT $+$ 倍增就可以做到 $T(n)=T(\frac{n}{2})+O(n\log n)=O(n\log n)$ 了。

/// details | 参考代码
    open: False
    type： success

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
#define gc getchar()
int read(){
	int x=0,f=1;char c;
	while(!isdigit(c=gc)) if(c=='-') f=-1;
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
	return x*f;
}
#undef gc
const int N=1<<19,inf=0x3f3f3f3f,mod=998244353,inv3=332748118;
int ksm(int a,int b){
	int c=1;
	while(b){
		if(b&1) c=1ll*a*c%mod;
		a=1ll*a*a%mod;
		b>>=1;
	}
	return c;
}
int n,f[N],pf[N],nf[N],rev[N];
void trans(int *a,int p,int type){
	int w=31^__builtin_clz(p);
	forup(i,0,p-1) rev[i]=(rev[i>>1]>>1)|((i&1)<<(w-1));
	forup(i,0,p-1) if(i<rev[i]) swap(a[i],a[rev[i]]);
	for(int len=1;len<p;len<<=1){
		int wn=ksm(type==1?3:inv3,(mod-1)/(len<<1));
		for(int i=0;i<p;i+=(len<<1)){
			int nw=1;
			forup(j,0,len-1){
				int x=a[i+j],y=1ll*nw*a[i+len+j]%mod;
				a[i+j]=(x+y)%mod;
				a[i+len+j]=(x-y+mod)%mod;
				nw=1ll*nw*wn%mod;
			}
		}
	}
	if(type==-1){
		int inv=ksm(p,mod-2);
		forup(i,0,p-1) a[i]=1ll*a[i]*inv%mod;
	}
}
void solve(int nn){
	pf[0]=ksm(f[0],mod-2);
	for(int n=2;n<=nn;n<<=1){
		forup(i,0,n-1) nf[i]=f[i];
		trans(nf,n+n,1);trans(pf,n+n,1);
		forup(i,0,n+n-1) pf[i]=1ll*pf[i]*((2-1ll*nf[i]*pf[i]%mod+mod)%mod)%mod;
		trans(pf,n+n,-1);
		forup(i,n,n+n-1) pf[i]=0;
	}
}
signed main(){
	n=read();
	forup(i,0,n-1) f[i]=read();
	int p=1;
	while(p<n) p<<=1;
	solve(p);
	forup(i,0,n-1){
		printf("%d ",pf[i]);
	}
}
```

///

### 多项式 ln

请原谅我为了子目录的显示效果没有在小标题上加 $\LaTeX$。

多项式的 $\ln$ 和 $\exp$（以 $e$ 为底的指数）定义为：

$$
\ln F(x)=-\sum_{i\ge 0}\frac{(1-F(x))^i}{i}
$$
$$
\exp F(x)=\sum_{i\ge 0}\frac{F(x)^i}{i!}
$$

这个定义是用麦克劳林级数求出来的，感兴趣的可以自己推一推。

为什么要用麦克劳林级数的定义呢？因为这个只用了加法和乘法，可以方便取模。

考虑 $(\ln F(x))'=\frac{F'(x)}{F(x)}$（以 $x$ 为主元）。于是求出 $\frac{F'(x)}{F(x)}$ 之后对它求积就行了。瓶颈在于求 $\frac{1}{F(x)}$。

### 多项式 exp

$\exp$ 是 $\ln$ 的逆运算，所以考虑牛顿迭代（下文 $A,B$ 均为 $A(x),B(x)$，为防止公式臃肿省去括号）。

设 $B=\exp A$，容易构造 $G(B)=\ln B-A$。

那么 $G'(B)=\frac{1}{B}=B^{-1}$，于是有：

$$
\begin{aligned}
B &\equiv B_{\ast}-\frac{G(B_{\ast})}{G'(B_{\ast})} &\pmod{x^n}\\\\
B &\equiv B_{\ast}-(\ln B_{\ast}-A)B_{\ast} &\pmod{x^n}\\\\
B &\equiv (1-\ln B_{\ast}+A)B_{\ast} &\pmod{x^n}\\\\
\end{aligned}
$$

然后就能 $O(n\log n)$ 倍增做了。