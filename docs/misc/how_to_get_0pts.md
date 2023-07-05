---
comments: true
---

# 挂分秘籍

你是否厌倦了天天 AK 被同学膜拜？你是否想获得毫不突出的大众分？你是否想得到遥不可及的爆零？请看此篇：挂分秘籍！！！

## 常见错误

> 数列中有负数要算最大值，但 `mx` 初始化为 $0$。

> 忘开 `long long`。

待补充...

## STL 相关错误。

```cpp title='对不存在的迭代器自增'
set<int> s;
//do something...
for(set<int>::iterator it=st;it!=ed;++it){
    s.erase(it);
}
```


```cpp title='正确写法'
set<int> s;
//do something...
for(set<int>::iterator it=st;it!=ed;){
    ++it;
    s.erase(prev(it));
}
```