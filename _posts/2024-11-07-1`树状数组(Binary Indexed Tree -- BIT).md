---
title: 1`树状数组(Binary Indexed Tree -- BIT)
date: 2024-11-07 12:00:00 +0800
categories: [算法与数据结构]
tags: []     # TAG names should always be lowercase
math: true
---
# 树状数组(Binary Indexed Tree -- BIT)

我们知道一维数组，也知道树形结构，那么树状数组是结合了它们的特点，一种更复杂的数据结构

## lowbit

lowbit（x）：

```c++
int lowbit(int x){ x & -x;}
```

== 非负整数x 的二进制形式 1的最低位以及后面0 组成的二进制数 对应的整数

== x的二进制 & x的二进制 所有位取反 + 1

== x & -x

例：lowbit（44）= lowbit(0010 1100) = 0000 0100(2进制) = 4（10进制）

## BIT

![1730981732408](/assets/img/blog/other/树状数组.png)

BIT：给定1维数组a[]映射（lowbit）为树形结构t[], a[]作为叶节点，t[i]对应二进制i，t[]的元素个数 == a[]的元素个数 == n

它相比线段树的代码更简洁，它的核心是 二进制低位技术（Lowbit）和差分数组，利用二进制规律将数组分成多个区间块，每个块存储部分和，通过跳跃式，实现高效更新和查询

BIT支持：

* 单点修改，前缀查询，区间查询，常用于动态更新数组，求前缀和/区间值问题
* 区间修改，单点查询，常用于动态更新数组区间，求某点值问题
* 而对于区间修改，区间查询通常用线段树解决

x的父节点 = x + lowbit(x) 

## BIT的数据

```c++
int n;//元素个数
int a[n];//一维数组
int t[n];//tree树
```

## BIT--初始化

遍历n次，对每个t[i]单点修改，其中k值为a[i]

```c++
int* buildTree(const int* a, int n) {
    int* t = new int[n + 1]();//因为从1开始
    for (int i = 0; i < n; i++) {
        add(t, i + 1, a[i], n + 1);
    }
    return t;
}
```

## BIT--单点修改

为了维护树形数组，当我们修改任意一个a[x]的值，即a[x] + k，都要修改所有t[x] + k以及所有父节点的值 + k

```c++
int add(int* t, int index, int k, int size)
{
	for(int i=index; i<=size; i+=lowbit(i))
	    t[i]+=k;
}
```

## BIT--前缀查询

如果要计算sum(7)->a[1]--a[7]的和

```c++
-> = t[7]+t[6]+t[4], 
而 6 = 7 - lowbit(7), 4 = 6 - lowbit(6) 
= t[7] + t[（7-lowbit(7)）] + t[（（7-lowbit(7)）-lowbit(（7-lowbit(7)）) ]
```

如果没有树形结构，那么我们要遍历7个数组元素求和，而现在只需要计算3次，即可获得前7个元素的和

```c++
int query(int* t, int index){
	int sum = 0;
	for(int i=index; i; i-=lowbit(i)){// i > 0
		sum+=t[i];
	}
	return sum;
}
```

## BIT--区间查询

如何求[L,R]的区间和呢？

利用前缀和相减的性质：[ L , R ] = [ 1 , R ] − [ 1 , L − 1 ]

```c++
int range_query(int* t, int L,int R)
{
	return query(t, R) - query(t, L);
}
```

## BIT--区间修改

对区间[L,R]+k，只需要对L和R更新差分数组

维护一个差分数组：

* c[0] = a[0]
* c[i] = a[i] - a[i-1]

```c++
//初始化差分数组
int* init(int* a, int n){
    int* c = new int[n]();
    c[0] = a[0];
    for (int i = 1; i < n; i++) {
        c[i] = a[i] - a[i-1];
    }
    return c;
}
//区间修改
void add(int* c, int index,int k)
{
	for(int i=index; i<=n; i+=lowbit(i))
	{
        c[i]+=k;
    }
}
void range_add(int L, int R, int val) {
    add(L, val);       
    add(R + 1, -val);  
}
```

## BIT--单点查询

求出c数组的前缀和

```c++
int query(int* c, int index)
{
	int ans=0;
	for(int i=index;i;i-=lowbit(i)) {
        ans+=c[i];
    }
	return ans;
}
```

## 练习题

> 面试题 10.10. 数字流的秩
> 315. 计算右侧小于当前元素的个数

