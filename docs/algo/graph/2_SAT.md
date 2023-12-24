---
comments: true
---

# 2-SAT 问题学习笔记

## 前言

有些时候我们会遇到一些由多个形如“$S$ 集合中至少有一个满足”限制的问题，我们按集合 $S$ 的大小称作“k-SAT（k-适定性问题，k-Satisfiability）”，而 $k>2$ 的已经被证明是 NPC 问题，此处我们只考虑 $k=2$ 的情况。

## 算法简述

考虑图论建模。

具体来说，我们能把这样的限制规约到位运算，然后再转化若干个为形如“若 $a$ 为真，必然有 $b$ 为真”的形式，此处简写为 $(a,b)$：

- $a\lor b=1:(\lnot a,b),(\lnot b,a)$
- $a\lor b=0:(a,\lnot a),(b,\lnot b)$（其实这个限制等价于要求 $a,b$ 均为假，然后若要求某个数为假可以这样建边）
- $a\land b=1:(\lnot a,a),(\lnot b,b)$（同上）
- $a\land b=0:(a,\lnot b),(b,\lnot a)$
- $a\oplus b=1:(a,\lnot b),(b,\lnot a),(\lnot a,b),(\lnot b,a)$（$\oplus$ 表示异或）
- $a\oplus b=0:(a,b),(\lnot a,\lnot b)$

我们可以给所有的 $(a,b)$ 连一条 $a\to b$ 的边，表示若 $a$ 在点集中，必须保证 $b$ 也在点集中。然后给所有的 $a,\lnot a$ 分别建一个点，然后 2-SAT 问题就转化成了选择一个点集，使得不存在 $a,\lnot a$ 均在该点集中。

容易发现这是一个和连通性有关的问题，可以考虑求强连通分量，若 $a,\lnot a$ 在同一强连通分量中就无解，否则有解。

## 例题

//// admonition | [P4782 【模板】2-SAT](https://www.luogu.com.cn/problem/P4782)

模板题。

/// details | 参考代码
    open: False
    type: success

```cpp
const int N=2e6+5,inf=0x3f3f3f3f;
int n,m,ans[N];
vector<int> e[N];
int dfn[N],low[N],Tm,blg[N],csc,ist[N];
stack<int> stk;
void Tarjan(int x){
	low[x]=dfn[x]=++Tm;
	stk.push(x);ist[x]=1;
	for(auto i:e[x]){
		if(!dfn[i]){
			Tarjan(i);
			low[x]=min(low[x],low[i]);
		}else if(ist[i]){
			low[x]=min(low[x],dfn[i]);
		}
	}
	if(dfn[x]==low[x]){
		++csc;
		while(stk.top()!=x){
			ist[stk.top()]=0;
			blg[stk.top()]=csc;
			stk.pop();
		}
		ist[stk.top()]=0;
		blg[stk.top()]=csc;
		stk.pop();
	}
}
signed main(){
	n=read();m=read();
	forup(i,1,m){
		int x=read(),a=read(),y=read(),b=read();
		e[x+(1-a)*n].push_back(y+b*n);
		e[y+(1-b)*n].push_back(x+a*n);
	}
	forup(i,1,n*2){
		if(!dfn[i]) Tarjan(i);
	}
	forup(i,1,n){
		if(blg[i]==blg[i+n]){
			puts("IMPOSSIBLE");
			return 0;
		}
	}
	puts("POSSIBLE");
	forup(i,1,n){
		printf("%d ",(blg[i]>blg[i+n]));
	}
}
```

///

////

//// admonition | [CF1697F Too Many Constraints](https://www.luogu.com.cn/problem/CF1697F)

考虑转化成 2-SAT 问题，参考[这篇博客]()。

////

//// admonition | [QOJ#5504 Flower Garden](https://qoj.ac/problem/5504)

不太一样的 2-SAT 问题，因为在 2-SAT 层面是必定有解的，这时候可以考虑用图论性质解决其他条件，参考[这篇博客]()

////