---
comments: true
---

# 字符串基础定义

## 前言

wqs 给我们讲字符串时我才发现有一些基础性的东西没有搞懂，但他今天讲的其他东西我都好像写过 blog 了，所以先来复习一下字符串基础的定义。

## 正文

### 字符集

**字符集**是一个集合，通常用符号 $\Sigma$（`$\Sigma$`）表示，字符集 $\Sigma$ 中的元素建立了全序关系，即任意两个元素 $\alpha,\beta$ 都可比较大小，不是 $\alpha<\beta$ 就是 $\alpha>\beta$。字符集 $\Sigma$ 中的元素称为字符。

### 字符串

一个**字符串** $S$ 是将 $n$ 个字符顺次排列形成的序列，$n$ 称为 $S$ 的长度，记为 $|S|$。

如果字符串下标从 $1$ 开始计算，$S$ 的第 $i$ 个字符表示为 $S_i$；

如果字符串下标从 $0$ 开始计算，$S$ 的第 $i$ 个字符表示为 $S_{i-1}$。

但其实我有些时候用小写 $s$ 或者其他字母表示一个字符串，这个要根据语境判断。

### 子序列

字符串 $S$ 的**子序列**是从 $S$ 中将若干元素提取出来并不改变相对位置形成的序列，即 $S_{p_1},S_{p_2},\dots,S_{p_k}，1\le p_1< p_2<\cdots< p_k\le|S|$。

或者可以理解为“删掉”一些元素剩下的东西。

### 子串

**子串**是一段连续的子序列，即满足 $\forall i=[i,k),p_{i+1}=p_i+1$ 的子序列 $S_{p_1},S_{p_2},\dots,S_{p_k}$，我通常用 $S_{i\sim j}$ 来表示从 $i$ 到 $j$ 的子串，有时候也会用 $i>j$ 的 $S_{i\sim j}$ 表示空串。

### 前后缀

**前缀**是一个开头恰好是字符串开头的子串，即 $S_{1\sim i}$。**后缀**则是结尾恰好是字符串结尾的子串，即 $S_{i\sim n}$。**真前缀**是除去整个字符串 $S$ 以外的其它前缀，**真后缀**的定义类似。

### 字典序

第 $i$ 个字符作为第 $i$ 关键字进行大小比较得到的字符串的大小关系，空字符小于字符集内任何字符（比如 $a< aa$）。

### 字符串匹配

**字符串匹配**又称**模式匹配**。该问题可以概括为“给定字符串 $S$ 和 $T$，在主串 $S$ 中寻找子串 $T$”。字符 $T$ 称为模式串。当模式串不止一个的时候称为**多模式匹配**。

字符串匹配有很多种方法，比较常用的有字符串哈希，KMP 和 AC 自动机。

### 字符串哈希

我们定义一个把字符串映射到整数的函数 $f$，这个 $f$ 称为是 **Hash 函数**。

哈希是经典乱搞做法，我们要求 $\forall S=T,f(S)=f(T)$，且对于 $S\ne T$，尽量避免 $f(S)=f(T)$，当出现这种情况时，我们称之为**哈希冲突**。

通常做法是取一个不大的质数（但要大于字符集大小）$B$，和一个大质数 $P$，然后把每个字符映射成一个非 $0$ 整数，最后把字符串视作一个 $B$ 进制模以 $P$ 的大数来算。

### 自动机

OI 中所说的**自动机**一般都指**确定有限状态自动机**。

自动机是 OI、计算机科学中被广泛使用的一个数学模型，其思想在许多字符串算法中都有涉及，因此推荐在学习一些字符串算法（KMP、AC 自动机、SAM）前先完成自动机的学习。学习自动机有助于理解上述算法。 ~~虽然我学完 AC 自动机还不知道自动机是什么~~

首先理解一下自动机是用来干什么的：自动机是一个对信号序列进行判定的数学模型。

一个**确定有限状态自动机**（DFA）是一个五元组 $(\Sigma,S,S_0,F,\delta)$，其中：

1. **字符集** $\Sigma$：自动机的输入信号只能包含这些字符。
2. **状态集合** $S$：如果把一个 DFA 看成一张有向图，那么 DFA 中的状态就相当于图上的顶点。这点我在 AC 自动机的讲解中有提到过。
3. **起始状态** $S_0$：$S_0\in S$，是一个特殊的状态。
4. **接受状态集合** $F$：$F\subseteq Q$，是一组特殊的状态。$F$ 中的状态称为**接受状态**
5. **转移函数** $\delta$（`$\delta$`）：$\delta$ 是一个接受两个参数返回一个值的函数，其中第一个参数和返回值都是一个状态，第二个参数是字符集中的一个字符。如果把一个 DFA 看成一张有向图，那么 DFA 中的转移函数就相当于顶点间的边，而每条边上都有一个字符。

当一个 DFA 读入一个字符串时，从初始状态起按照转移函数一个一个字符地转移。如果读入完一个字符串的所有字符后位于一个接受状态，那么我们称这个 DFA **接受**这个字符串，反之我们称这个 DFA **不接受**这个字符串。

DFA 的作用就是识别字符串，一个自动机 $A$，若它能识别（接受）字符串 $S$，那么 $A(S)=\mathrm{True}$，否则 $A(S)=\mathrm{False}$。

如果一个状态 $v$ 没有字符 $c$ 的转移，那么我们令 $\delta(v,c)=\mathrm{null}$，而 $\mathrm{null}$ 只能转移到 $\mathrm{null}$，且 $\mathrm{null}$ 不属于接受状态集合。无法转移到任何一个接受状态的状态都可以视作 $\mathrm{null}$，或者说，$\mathrm{null}$ 代指所有无法转移到任何一个接受状态的状态。

容易发现，Trie 是一个典型的自动机（五个元素都挺显然的），KMP 可以视作一个自动机（前四个元素显然，除去匹配下一个字符以外，其余的 $\delta(x,c)$ 就是不断跳 $next$ 直到下一个是 $c$，这个可以写成数学语言）。

然后大概就没了。