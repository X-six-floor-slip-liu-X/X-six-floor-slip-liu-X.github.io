---
comments: true
---

# 后缀自动机（SAM）学习笔记

## 算法引入

约定下标从 $1$ 开始，记 $s$ 以 $l$ 开头以 $r$ 结尾的子串为 $s[l:r]$。

约定大写 $S$ 表示自动机状态集合，$S_0$ 表示初始状态，$\delta(x,c)$ 表示从状态 $x$ 走字符 $c$ 的转移边。

在阅读本文前，请确保你已经了解字符串基础概念（至少你要知道自动机是什么吧），可以阅读我的[这篇博客](./string_base.md)。

我曾经说过一句话：

> 字符串算法就是一堆看起来没什么卵用实际上很有用的算法。

后缀自动机（Suffix Automaton，简称 SAM）虽然目的明确，但是据高二学长 wqs 所说，他做 SAM 题从来没用过它原本的目的。

SAM 被创造出来的目的是一个**能接受指定 $S$ 字符串所有后缀**的自动机，这样就能 $O(|t|)$ 地判断 $t$ 是不是 $s$ 的后缀了。

考虑 KMP 是能接受**以一个完整字符串 $s$ 为后缀的所有字符串** $t$ 的自动机，而由于自动机数学模型每次只能读一个字符，故所有 $s$ 的前缀 $s'$ 对应的状态**均在自动机状态集合 $S$ 中**。

而类比过来，SAM 的定义是能接受 **指定字符串 $s$ 的所有后缀**的自动机，那么显然，这个自动机必须保存 $s$ 的所有子串。所以 SAM 可以用来高效地处理子串信息。

直观上，字符串的 SAM 可以理解为给定字符串的**所有子串**的压缩形式。值得注意的事实是，SAM 将所有的这些信息以高度压缩的形式储存。对于一个长度为 $n$ 的字符串，它的空间复杂度仅为 $O(n)$。此外，构造 SAM 的时间复杂度也仅为 $O(n)$。更准确地说，一个 SAM 最多有 $2n-1$ 个节点和 $3n-4$ 条转移边。

## 算法简述

### 定义

字符串 $s$ 的 SAM 的定义是一个接受 $s$ 所有后缀的**最小的 DFA**（确定有限状态自动机）。

### 应具有的性质

刚刚提到了，SAM 应保存 $s$ 的**所有子串信息**，即任意子串信息都能从初始状态 $S_0$ 经过若干条转移边得到。由于 SAM 是极小的，那么任意一条从 $S_0$ 出发的路径都应该是 $s$ 的一个子串。故 $s$ 的子串与自动机上从 $S_0$ 出发的路径应具有**双射**关系。为方便叙述，本文中称一条路径**对应**一个子串。

由于自动机一次只能读一个字符，所以 $s[1:k]$ 的某个后缀必须通过 $s[1:k-1]$ 的某个后缀加上字符 $s_k$ 转移过来。故为了构建 $s[1:k]$ 的自动机，我们必须先构建 $s[1:k-1]$ 的自动机。换句话说，SAM 的构建应是一个**在线**的过程，对于任意一个时刻 $i$，自动机都应是可以使用的。所以我们考虑构建方法时可以直接从“如何在一个已经建好的 SAM 后面插入一个字符”的方面考虑。

### 如何构建

在本段中，请先不管代码实现以及复杂度问题。

为方便叙述，称一个子串的**出现位置**为它最后一个字符的位置，比如 $s[3:5]$ 的出现位置就是 $5$。

注意到对于多个本质相同的子串，由于我们读入字符串 $t$ 时只考虑它**是不是** $s$ 的子串，而不考虑它的位置。所以本质相同的子串显然可以合并为同一个状态。而转移就是这些子串所有结束位置 $e$ 的下一个字符 $s_{e+1}$。考虑令 $\mathrm{endpos_{\mathit{a}}}$ 表示子串 $a$ 的所有结束位置构成的集合。

而容易发现，对于两个子串 $a,b$（钦定 $|a|\ge |b|$），若 $b$ 不是 $a$ 的后缀，那么 $\mathrm{endpos_{\mathit{b}}}\cap \mathrm{endpos_{\mathit{a}}}=\varnothing$，因为任意出现 $b$ 的位置都无法通过向前拓展的方式得到 $a$。而若 $b$ 是 $a$ 的后缀，那么有 $\mathrm{endpos_{\mathit{a}}}\subseteq \mathrm{endpos_{\mathit{b}}}$，因为任意出现了 $a$ 的位置都能通过删掉开头若干个字符得到 $b$，并且显然 $b$ 还可能在其他地方出现。那么显然，对于两个字符串 $a,b(|a|\ge|b|)$，它们 $\mathrm{endpos}$ 的交不是空集就是 $\mathrm{endpos_{\mathit{b}}}$

然后着重考虑多个 $\mathrm{endpos}$ 相同的情况。容易想到若 $\mathrm{endpos}$ 那么对应的转移也相同，或许可以合并为同一个状态，考虑寻找性质。令 $t_1,t_2,\dots t_k$ 为某个满足 $\mathrm{endpos_{\mathit{t_1}}}=\mathrm{endpos_{\mathit{t_2}}}=\cdots=\mathrm{endpos_{\mathit{t_k}}}$ 的极大字符串集合（“极大”是指不存在其它字符串 $\mathrm{endpos}$ 和它们相同），且钦定 $|t_1|\ge |t_2|\ge\dots\ge |t_k|$。首先注意到两长度相同的本质不同子串 $\mathrm{endpos}$ 显然不可能相同，故上面对 $t$ 排列的要求就可以不取等了。由于它们的交不为空，那么所有 $t_i$ 必定都是 $t_1$ 的后缀，不妨从 $t_1$ 每次删掉开头字符考虑，容易发现每删掉一个字符，$\mathrm{endpos}$ 内部的元素是单调变多的，也就是说随着删开头字符，不可能出现 $\mathrm{endpos}$ 从等于 $\mathrm{endpos_{\mathit{t_1}}}$ 变成不等于 $\mathrm{endpos_{\mathit{t_1}}}$ 又变回 $\mathrm{endpos_{\mathit{t_1}}}$ 的情况。换句话说，$\forall i\in[1,k),t_{i+1}$ 必定是由 $t_i$ 删掉开头字符得到的。

考虑把这些 $t_i$ 视为同一个等价类 $x$，那么根据之前的猜想，可以把一个等价类作为一个状态（下文的所有 $x$ 以及带任意下标的 $x$ 若没有另行说明均指代某自动机状态或其代表的等价类）。如果删掉 $t_k$ 的开头字符得到 $t'$，它必定属于另一个等价类，记为 $\mathrm{link_{\mathit{x}}}$。容易发现 $x\to \mathrm{link_{\mathit{x}}}$ 的关系形成了一个以初始状态 $S_0$ 为根的树形结构。其中对于任意一条树边，有 $\mathrm{endpos_{\mathit{x}}}\subsetneq \mathrm{endpos_{link_{\mathit{x}}}}$，对于任意一个 $x$，它的所有儿子对应的 $\mathrm{endpos}$ 必定互不相交（因为如果两 $\mathrm{endpos}$ 相交，那么它们对应的状态必定是祖先和后代的关系）。

对于一个等价类 $x$，它必定有一个最长的字符串 $\mathrm{longest_{\mathit{x}}}$，它的长度记作 $\mathrm{len_{\mathit{x}}}$。以及最短的字符串 $\mathrm{shortest_{\mathit{x}}}$，它的长度记作 $\mathrm{minlen_{\mathit{x}}}$。只要找到了这两个字符串就能确定等价类 $x$ 内所有字符串。注意到 $\mathrm{len_{link_{\mathit{x}}}}+1=\mathrm{minlen_{\mathit{x}}}$（除非 $x=S_0$），所以我们其实不需要维护 $\mathrm{minlen_{\mathit{x}}}$ 和 $\mathrm{shortest_{\mathit{x}}}$。在后文我会说明其实 $\mathrm{longest_{\mathit{x}}}$ 也不需要维护。此外，容易发现从任意结点 $x$ 每次跳 $\mathrm{link_{\mathit{x}}}$ 直到 $S_0$，路径上所有等价类内字符串恰好是从 $\mathrm{longest_{\mathit{x}}}$ 开始每次删掉开头字符直到变为空串的所有字符串。

根据前文的猜想，考虑已经构建好了 $s[1:k-1]$ 的 SAM，如何插入 $s_k$，令 $c=s_k$。

首先显然地，子串 $s[1:k]$ 此时必定只出现了一次。那么新建一个结点 $x$ 包含所有**第一次出现** 的字符串，显然它们必然都是 $s[1:k]$ 的后缀，那么设 $x_0$ 为 $s[1:k-1]$ 所在的自动机状态。那么有 $\mathrm{len_{\mathit{x}}}=\mathrm{len_{\mathit{x_0}}}+1$ 显然所有**有可能**存在转移边 $\delta(x',c)=x$ 的 $x'$ 必定是 $x_0$ 的**祖先**（包括它自己）。

考虑从 $x_0$ 开始一步一步往上跳 $\mathrm{link}$，最下面的一些状态本来就不存在转移边 $\delta(x',c)$，可以直接令 $\delta(x',c)=x$。考虑找到第一个存在 $\delta(x',c)$ 的 $x'$，记这个 $x'$ 为 $p$，又记 $\delta(p,c)=q$（注意可能找不到这个 $x'$，即 $\delta(S_0,c)$ 不存在，但是如果 $x'=S_0$ 也要进行下一步）。接下来分两种情况：

1. $\mathrm{len_{\mathit{q}}}=\mathrm{len_{\mathit{p}}}+1$

说明 $q$ 内所有字符串均能通过 $p$ 及其祖先内的字符串加上字符 $c$ 得到，即它们全都是子串 $s[1:k]$ 的后缀，由于 $p$ 是 $x_0$ 的祖先中最深的拥有 $\delta(p,c)$ 转移边的结点，也就是说 $\mathrm{longest_{\mathit{q}}}$ 是从 $s[1:k]$ 开始依次删除开头字符能得到的第一个 $\mathrm{endpos}$ 与 $x$ 不同的字符串，即 $q=\mathrm{link_{\mathit{x}}}$。

2. $\mathrm{len_{\mathit{q}}}\ne \mathrm{len_{\mathit{p}}}+1$。

此时 $q$ 只有一部分（更具体地说，是长度小于 $len_p+1$ 的那一部分）能通过 $p$ 及其祖先内的字符串加上字符 $c$ 得到，考虑把这一部分分裂出来，得到一个新状态 $q'$，然后剩下的字符串视作 $q''$。

考虑 $q',q''$ 的信息（$\mathrm{len},\mathrm{link},\delta$）应该怎么赋。

首先对于 $\delta$，由于 $q$ 的转移边 $q'$ 和 $q''$ 显然都能转移（在对应位置都出现过），所以直接复制即可。

然后是 $\mathrm{link}$，显然 $\mathrm{link_{\mathit{x}}}=\mathrm{link_{\mathit{q''}}}=q',\mathrm{link_{\mathit{q'}}}=\mathrm{link_{\mathit{q}}}$。

接下来是 $\mathrm{len}$，根据定义可得 $\mathrm{len_{\mathit{q'}}}=len_{p}+1,\mathrm{len_{\mathit{q''}}}=\mathrm{len_{\mathit{q}}}$。

然后从 $p$ 往上的一段链本来转移到 $q$，由于从 $p$ 分裂开，故这些 $\delta'(p',c)$ 一直到某位置的 $\delta(p',c)$ 是 $q$ 的祖先为止都要重新赋值为 $q'$。

具体实现的时候，可以只新建一个 $q'$，然后 $q''$ 直接从 $q$ 上修改得到。

然后比如你可能要维护 $\mathrm{endpos}$ 的各种信息。在离线的情况下，假如你只维护 $\mathrm{endpos}$ 的大小那比较简单，每次新建的结点 $x$ 大小是 $1$（加上子树信息，它可能在后面得到儿子），其余结点就是一个子树和（考虑等价类的性质）。如果要维护 $\mathrm{endpos}$ 内的元素，需要用一个叫“线段树的可持久化合并”的东西，具体过程大概就是在线段树合并时新建一个版本。如果强制在线，可能需要用 LCT 维护（虽然我不知道 LCT 是什么），因为所有点的 $\mathrm{link}$ 随时都会改变。

### 复杂度证明

前文说过 SAM 的状态数和转移数都是 $O(n)$ 的，并且点数上界是 $2n-1$，边数是 $3n-4$。

---

首先点数很好证，初始只有一个结点 $S_0$，每一次插入字符至多增加两个结点（$x,q'$），但是前两次显然不可能分裂出 $q'$。能卡到这个上界的字符串必定形如 `abbbbb...`，因为第三次及以后每次都会分裂出一个 $q'$。

---

然后是边数。容易发现对于任意一个状态 $q$，都存在一个转移 $(p,q)$ 使得 $\mathrm{len_{\mathit{q}}}=\mathrm{len_{\mathit{p}}}+1$。我们称这样的转移是**连续的**。

容易发现连续的边必定能组成一棵以 $S_0$ 为根的外向树，故这一部分边数上界是 $2n-2$。考虑非连续转移的数量。

注意到对于一个非连续转移 $u\to v$，必定存在唯一的一条只走连续转移从 $S_0$ 到 $u$ 的路径。然后从 $v$ 开始，走到以 $S_0\to v$ 路径对应的子串开头的后缀中最长的一个所对应的接收状态。容易发现 $u\to v$ 是这一个后缀中**第一条**非连续转移边。我们可以由此建立一个映射关系 $f(e)$，表示以 $e$ 为第一条非连续转移边对应的后缀。容易发现若 $f(a)=f(b)$，那么必然有 $a=b$（因为一个后缀在自动机上最多只存在一条**第一条**转移边）。这说明非连续转移边数量**小于等于**后缀数量。并且容易发现，对于 $s[1:n]$ 后缀，它对应的路径中间显然不可能有非连续转移边（$s[1:n]$ 每次走的路径就是每次插入的 $x_0\to x$），故我们能得到转移数上界为 $3n-3$。但是由于状态数上界 $2n-1$ 只会在形如 `abbb...` 的字符串中产生，但此时边界数小于 $3n-3$。故我们能得到转移数的上界是 $3n-4$，字符串 `abbb...bbbc` 就能卡到这个上界，需要注意不一定能卡到上界的字符串都形如这样。

---

接下来思考构建过程中的时间复杂度。

首先对于暴力从 $x_0$ 向上跳找到 $p$，由于每跳一次就会新增一条转移边，所以这一部分总复杂度就是 $O(n)$ 的。

然后下一步若不分裂 $q$ 复杂度显然是 $O(1)$ 的。若要分裂 $q$，$\mathrm{len},\mathrm{link}$ 的更新都是 $O(1)$ 的，$\delta$ 的更新确实没办法，是 $O(|\Sigma|)$ 的（但是如果你用 `map` 存边就是 $O(\log |\Sigma|)$ 的，因为你每次操作都会新增一个转移，均摊是插入次数 $O(1)$ 的，但是你插入一条边是 $O(\log |\Sigma|)$ 的）。最后考虑本来指向 $q$ 的转移边更新为 $q'$ 的复杂度。

仍沿用之前实现过程中的变量名。

首先 $p$ 必定是 $\mathrm{link_{\mathit{x_{\mathrm{0}}}}}$ 或其祖先，然后一直到某个 $p'$ 使得 $\delta(p',c)\ne q$ 为止的一段都需要赋值。这个 $p'$ 必然是 $\mathrm{link_{link_{\mathit{x_{\mathrm{0}}}}}}$ 或其祖先，并且它会转移到 $\mathrm{link_{link_{\mathit{x}}}}$，即 $\mathrm{len_{link_{link_{\mathit{x}}}}}=\mathrm{len_{\mathit{p'}}}+1$。那么每跳一次都会使 $\mathrm{len_{\mathit{p'}}}$ 至少减少 $1$，而转移到的 $\mathrm{link_{link_{\mathit{x}}}}$ 就会成为下一次“可能的最大的 $p'$”，那么假如你取一个指针维护这个东西，他就至多会变化 $O(n)$ 次（虽然一般不这么写）。

/// details | [模板题](https://www.luogu.com.cn/problem/P3804)参考代码
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
using i64=long long;
#define gc getchar()
inline int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=1e6+5,inf=0x3f3f3f3f;
i64 n,ans;
char str[N];
vector<int> e[N<<1];
int lnk[N<<1],len[N<<1],endpossz[N<<1],tr[N<<1][26];
int lst,cntn;
void dfs(int x){
	for(auto i:e[x]){
		dfs(i);
		endpossz[x]+=endpossz[i];
	}
	if(endpossz[x]>1) ans=max(ans,1ll*len[x]*endpossz[x]);
}
void Work(){
	lnk[0]=-1;// (1)!
	forup(i,1,n){
		int x=++cntn,c=str[i]-'a';
		len[x]=len[lst]+1;endpossz[x]=1;
		int p=lst;
		lst=x;
		while(~p&&!tr[p][c]){
			tr[p][c]=x;
			p=lnk[p];
		}
		if(p==-1){
			lnk[x]=0;
			continue;
		}
		int q=tr[p][c];
		if(len[q]==len[p]+1){
			lnk[x]=q;
		}else{
			int _q=++cntn;
			len[_q]=len[p]+1;endpossz[_q]=0;
			lnk[_q]=lnk[q];lnk[q]=lnk[x]=_q;
			forup(i,0,25){
				tr[_q][i]=tr[q][i];
			}
			while(~p&&tr[p][c]==q){
				tr[p][c]=_q;
				p=lnk[p];
			}
		}
	}
	forup(i,1,cntn){
		e[lnk[i]].push_back(i);
	}
	dfs(0);
}
signed main(){
	scanf(" %s",str+1);
	n=strlen(str+1);
	Work();
	printf("%lld\n",ans);
}
```

1. 把 $\mathrm{link_{0}}$ 设为 $-1$，此处 $-1$ 相当于 $\mathrm{null}$ 状态，即一个不能到达任何接收状态的状态。这样实现起来也更方便。

///

### 其它性质

然后在彻底理解 SAM 的正确性后，我们还能发现一些新的性质。

显然 $\mathrm{link}$ 关系会形成一棵树，我们称之为 parent tree。

SAM 肯定是一个 DAG。因为任意一个转移 $(p,q)$，都有 $\mathrm{len_{\mathit{q}}}\le \mathrm{len_{\mathit{p}}}+1$，显然不会形成环。并且这个 DAG 的总路径数小到令人发指，是 $O(n^2)$ 级别（原串中本质不同子串个数）的（随机 DAG 路径数是指数级别的）。因为任意一个是原串子串的字符串都有唯一一条路径与它对应，而对于任意一条不是原串子串的字符串，都必定不存在一条对应的路径。

并且，同一个等价类内部不可能出现“存在两字符串 $s,t$，使得 $s$ 在 $t$ 中能匹配两次”的情况。因为等价类的意义是 $\mathrm{endpos}$ 相同，如果 $s$ 能匹配两次 $t$ 那么必然 $t$ 在前面一次也出现了，这就无限递归了。

## 常见用法

考虑直接报菜名（排序不分先后，但是大致是按难度排的）。

1. 先给定文本串，再给定模式串的多模式匹配

建出文本串 $s$ 的 SAM 后读入模式串 $t_i$ 即可，如果能完整读入说明匹配。复杂度 $O(|s|+\sum |t_i|)$。

2. 本质不同子串个数

经典 $O(n\log n)$ SA 例题，但是 SAM 可以做到 $O(n)$，还能求出每个前缀的答案。

本质不同子串个数就是每个等价类的 $\mathrm{len}-\mathrm{minlen}+1=\mathrm{len}-\mathrm{len_{link}}$，每次插入的时候动态维护即可，另外显然分裂 $q$ 的时候不会增加本质不同子串个数。

例题 [P4070 [SDOI2016] 生成魔咒](https://www.luogu.com.cn/problem/P4070)。

3. 所有本质不同子串总长度

我觉得看了上一个这一个你也能会，等差数列求和即可。

4. 出现次数

就是 $\mathrm{endpos}$ 的大小，parent tree 上的子树和。

具体来说，每次新增的 $x$ 自带一个 $\mathrm{endpos}$，其余均为 $0$，然后做子树和。

5. 第一次出现位置

和上一个一样，换成求最小值。

6. 最小循环移位

容易发现字符串 $S+S$ 包含字符串 $S$ 的所有循环移位作为子串。

所以问题简化为在 $S+S$ 对应的后缀自动机上寻找最小的长度为 $|S|$ 的路径，这可以通过平凡的方法做到：我们从初始状态开始，贪心地访问最小的字符即可。

总的时间复杂度为 $O(|S|)$。

7. 所有出现位置

**线段树的可持久化合并**（这个名字我看一次笑一次）。

用动态开点线段树维护 $\mathrm{endpos}$，然后每个点往父亲合并就是线段树合并。

但是我们在合并的同时可能还要保留儿子的信息，怎么办呢？容易想到可持久化。

具体来说，在线段树合并的基础上，每次新增一个结点。

感觉很抽象啊，建议看代码：

/// details | 参考代码
	open: True

```cpp linenums="1" hl_lines="3"
	int Merge(int l,int r,int u,int v){
		if(!u||!v) return u|v;
		int id=++cntt;
		ls[id]=Merge(l,mid,ls[u],ls[v]);
		rs[id]=Merge(mid+1,r,rs[u],rs[v]);
		PushUp(id);
		return id;
	}
```

///

然后你就对每个点得到了一棵保存所有 $\mathrm{endpos}$ 的线段树。

复杂度 $O(n\log n)$。

8. 两个字符串的最长公共子串

考虑对 $s$ 建 SAM，然后读入 $t$。

如果不能拓展，就暴力跳 $\mathrm{link}$ 知道有对应转移（或者到 $\mathrm{null}$）为止，相当于删掉开头。

显然跳的次数不会超过 $O(|t|)$。

注意转移不一定是连续的，一个比较方便的做法是求出 $t$ 每个右端点是和 $s$ 的公共子串的最小左端点。

复杂度 $O(|s|+|t|)$。

9. 多个字符串的最长公共子串

这玩意貌似不算常见。

考虑把所以字符串连在一起，在其间加上**互不相同的没在原字符集中出现过的字符**。

$$T=C_1+S_1+C_2+S_2\dots +C_k+S_k$$

然后对 $T$ 建 SAM。

容易发现若某子串 $s$ 在 $S_i$ 中出现当且仅当存在一条从以 $C_i$ 结尾的状态，且不经过以其余 $C$ 结尾的状态，能到达 $s$ 所在等价类的路径。

那么 DAG 上 DP，状压统计可达性即可。

这种题 $k$ 不会太大。

10. 区间本质不同子串

代码比较毒瘤，但思路其实还算好，参考[这篇博客](../../records/2024Jan.md/#p6292)