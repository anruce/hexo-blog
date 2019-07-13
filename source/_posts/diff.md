---
title: React diff 算法
date: 2019/04/01
tags: 
toc: false
comments: true
categories: web
---

理解react的diff算法

<!-- more -->

![jsworke](/images/passive/diff.png)

## 同级节点的移动 增加 删除的具体实现


### 场景一 新旧集合中存在相同节点但位置不同时，如何移动节点

```
旧  a b c d
新  b a d c
```

React先从新组合中取得b，然后判断旧中是否存在相同节点b，当发现存在节点b后，就去判断是否移动b
涉及到两个变量 index 和lastIndex

* index:b在集合里下标 此时 index = 1
* lastIndex：类似于一个map的索引，一开始默认值是0，它会与map中的元素进行比较，比较完后，更新当前的值（取index和lastIndex的较大数）

#### 比较规则：
如果 index < lastIndex 那此元素就需要移动
在旧组合里将该元素移动到下标为lastIndex的位置

具体的计算过程看下图

![jsworke](/images/passive/diff2.jpeg)

### 场景一 新集合中有新加入的节点，旧集合中有删除的节点
规则同上


```
旧  a b c d
新  b e c a
```
比较b 此时 index = 1 lastindex = 0  1>0 b不移动 更新 lastindex为1
当比较到e的时候，发现旧组合中不存在，故在旧组合下标为1的位置 创建E，更新lastIndex=1
...
对比到a 因为是最后一个 所以 diff操作结束

新组合对比完成后 再去对旧集合遍历 判断新集合没有，但旧集合有的元素（如d），删除d，diff操作结束





