---
layout: post
title:  "算法-不想交集"
date:   2017-09-16 10:58:52
categories: algorithm 算法
published: true
comments: true
thread: 20170916101155555
---
算法-不相交集
---
## 1 定义
`关系`：对于集合S中任意元组(a,b)， a、b属于S， aRb或者为true或者为false，则称在集合S上定义关系R。

`等价关系`（也用符号`～`表示）：
1. 自反性。对于所有a属于S，aRa;
2. 对称性。aRb当且仅当bRa;
3. 传递性。aRb且bRc 则 aRc;

`不相交集`: 元素a属于集合S，a的`等价类`是S的一个子集，它包含所有与a有等价关系的元素。`等价类`形成了对集合S的划分，不同的`等价类`之间的交集为空，
使得这些集合`不相交`。
### 1.1 操作
有两种操作可以进行。
- 第一种是find操作，返回给定元素的等价类的名字。
- 第二种是添加关系。添加a~b
  - 查看a、b是否有关系
  - 分别对a、b执行find查看返回是否一样。不一样执行union操作。

我们可以用tree结构来表示一个集合，root可以表示集合的名字。由于仅有上面的两个操作而没有顺序信息，因此我们可以将所有的元素用1-N编号，编号可以用hashing方法。

进一步可以发现对于这两个操作无法使其同时达到最优，也就是说当Find以常数最坏时间运行时，Union操作会很慢，同理颠倒过来。因此就有了2种实现方式。

- 使Find运行快
  在数组中保存每个元素的等价类的名字，将所有等价类的元素放到一个链表中
- 使Union运行快
  使用树来表示每一个集合，根节点表示集合的名字。数组元素P[i]表示元素i的父亲，若i为root，则P[i]=0。

### 1.2 示例
- 寄宿学校中的宿舍和舍友。a是a的舍友；a和b是舍友，b和a也是；a、b舍友，b、c舍友，则a、c也是舍友。
- 电器连通性。

## 2 实现
### 2.1 普通算法
按大小求并||按高度求并
- find 效率 O（N）
- union效率 O（1）
```java
void union(int root1, int root2)
{
    S[root1] = root2;
}

int find(int x)
{
    if(S[x]<=0)
        return x;
    else
        return find(S[x]);
}
```

### 2.2 灵巧合并算法
按大小求并||按高度求并（`按秩求并`）
- find 效率 O（logN）
- union效率 O（1）

```java
// 按大小
void union(int root1, int root2)
{
  if (S[root2] <= S[root1]) {
      S[root1] = root2;      
      S[root2] = S[root1] + S[root2];
  } else {
      S[root2] = root1;      
      S[root1] = S[root1] + S[root2];
  }
}

int find(int x)
{
    if(S[x]<=0)
        return x;
    else
        return find(S[x]);
}

// 按高度
void union(int root1, int root2)
{
  if (S[root2] < S[root1])
      S[root1] = root2;    
  else {
      if (S[root1] == S[root2])
          S[root1]--;      
      s[root2] = root1;    
  }
}
```

### 2.3 路经压缩
路径压缩不完全与按高度求并兼容，因为路经压缩可以改变树的高度。此时，对于每棵树的高度是估算的高度，也称为秩（Rank），也可以称为按秩求并。
```java
void union(int root1, int root2)
{
  if (S[root2] < S[root1])
      S[root1] = root2;    
  else {
      if (S[root1] == S[root2])
          S[root1]--;      
      s[root2] = root1;    
  }
}

int find(int x)
{
    if(S[x]<=0)
        return x;
    else
        return S[x]=find(S[x]);
}
```


## 3 适用场景
迷宫生成：5*5迷宫，每一个格子当成一个集合，所有集合union成一个时，迷宫就生成。
