---
comments: true
---

# 挂分秘籍

你是否厌倦了天天 AK 被同学膜拜？你是否想获得毫不突出的大众分？你是否想得到遥不可及的爆零？请看此篇：挂分秘籍！！！

## 常见错误

数列中有负数要算最大值，但 `mx` 初始化为 $0$。

忘开 `long long`。

线段树/平衡树等数据结构修改时不 `PushUp`。

```cpp title='线段树上二分修改时先修改再引用'
	void Erase(i64 N,i64 l=0,i64 r=maxn,i64 id=1){
		if(l<r&&mark[id]) PushDown(id);
		if(l==r){
			querycnt[id]-=N;
			querysum[id]-=lsh[l]*N;
			return;
		}
		if(N<=querycnt[id<<1]){
			Erase(N,lson);
		}else{
			mark[id<<1]=1;// (1)!
			querycnt[id<<1]=0;
			querysum[id<<1]=0;
			Erase(N-querycnt[id<<1],rson);
		}
		PushUp(id);
	}
/// 正确写法
	void Erase(i64 N,i64 l=0,i64 r=maxn,i64 id=1){
		if(l<r&&mark[id]) PushDown(id);
		if(l==r){
			querycnt[id]-=N;
			querysum[id]-=lsh[l]*N;
			return;
		}
		if(N<=querycnt[id<<1]){
			Erase(N,lson);
		}else{
			Erase(N-querycnt[id<<1],rson);
			mark[id<<1]=1;
			querycnt[id<<1]=0;
			querysum[id<<1]=0;
		}
		PushUp(id);
	}
```

1. $\leftarrow$ SB。

线段树单点修改 `querymin[id]=P`（其中 $P$ 是要改的位置，要改成的值是 $X$）。

高精度把高位和低位搞反。

启发式合并写成 `if(sz[u]>sz[u]) swap(u,v);`

```cpp title='对不存在的迭代器自增'
set<int> s;
//do something...
for(set<int>::iterator it=st;it!=ed;++it){
    s.erase(it);
}

//正确写法
set<int> s;
//do something...
for(set<int>::iterator it=st;it!=ed;){
    ++it;
    s.erase(prev(it));
}
```

```cpp title='错误的 popcount' hl_lines="5 14"
int ppcnt(int x){
	int res=0;
	while(x){
		x-=x&-x;
		res+=x;
	}
	return res;
}
// 正确写法
int ppcnt(int x){
	int res=0;
	while(x){
		x-=x&-x;
		res++;
	}
	return res;
}
```

待补充...