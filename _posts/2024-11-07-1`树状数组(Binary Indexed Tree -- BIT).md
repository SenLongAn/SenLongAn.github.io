---
title: 1`前缀/区间查询(前缀和，树状数组BIT)，并查集，单调栈，字典树
date: 2024-11-07 12:00:00 +0800
categories: [算法与数据结构]
tags: []     # TAG names should always be lowercase
math: true
---

# 前缀和

* 一维：
  * 预计算前缀和数组：prefix[i] = prefix[i-1] + nums[i]，prefix[i] = arrx[0] + ……arr[i]
  * 区间[L, R]和：sum(L, R) = prefix[R] - prefix[L-1]，通常定义 prefix[-1] = 0
* 二维：
  * ![1736083266125](/assets/img/blog/Games202/2d区域查询.png)
  * 预计算前缀和数组：pref[i][j] = pref[i-1][j] + pref[i][j-1] - pref[i-1][j-1] + matrix[i][j]，从左上角 (0, 0) 到右下角 (i, j)所有元素的和
  * 区间[L(x1, y1), R(x2, y2)]：sum(L, R) = pref[x2][y2] - pref[x1-1][y2] - pref[x2][y1-1] + pref[x1-1][y1-1]
* 前缀和
  * 使用场景：一维/二维静态数组上高效区间查询
  * 复杂度：初始化O(n)，单点更新O(n)（重新计算前缀数组），区间查询O(1)
  * 优点：实现简单，查询高效，支持一维二维
  * 缺点：不适合动态更新

# 树状数组BIT

## lowbit

lowbit（x）：

```c++
int lowbit(int x){ x & -x;}
```

非负整数x 的二进制形式 从低位到高位的第一个1 和从最低位开始 组成的二进制数 对应的整数

例：lowbit（44）= lowbit(0010 1100) = 0000 0100(2进制) = 4（10进制）

## BIT

![1730981732408](/assets/img/blog/other/树状数组.png)

BIT：给定1维数组a[]映射（lowbit）为树形结构t[]

父子关系：x的父节点 = x + lowbit(x) (二进制运算)（根据元素索引构建而非元素值）

任意树节点值：保存a数组元素[i-lowbit(i)+1, i]区间的总和，保存树孩子（最接近的子孙）的和 + a[i]

使用场景：一维动态数组上高效区间查询

复杂度：初始化O(n log n)，单点更新O(log n)，区间查询O(log n)

优点：适合动态更新

缺点：初始化时间和查询时间相对前缀和较慢，不支持二维

## 数据

```c++
int n;//元素个数
int a[n];//一维数组
int t[n];//tree树
```

* a[i]数组元素（也可以理解为叶节点），t[i]树节点 

## 单点修改

```c++
void add(int* t, int index, int k, int size) {
    for (int i = index; i <= size; i += lowbit(i)) {
        t[i] += k;
    }
}
```

* 任意a数组元素值+k，都要从它对应的树节点开始，向上对所有父节点值+k，

## 初始化

```c++
int* buildTree(const int* a, int n) {
    int* t = new int[n + 1]();
    for (int i = 0; i < n; i++) {
        add(t, i + 1, a[i], n);
    }
    return t;
}
```

* 创建树所有节点（父子关系默认建立好的）
* 遍历所有数组元素，设置每个树节点值，这里传入i+1，是因为树的节点索引从1开始算的（这样才能应用lowbit）

## 前缀查询

```c++
-> = t[7]+t[6]+t[4], 
而 6 = 7 - lowbit(7), 4 = 6 - lowbit(6) 
= t[7] + t[（7-lowbit(7)）] + t[（（7-lowbit(7)）-lowbit(（7-lowbit(7)）) ]
```

如果要计算sum(7)->a[1]--a[7]的和

```c++
int query(int* t, int index){
	int sum = 0;
	for(int i=index; i; i-=lowbit(i)){// i > 0
		sum+=t[i];
	}
	return sum;
}
```

## 区间查询

```c++
int range_query(int* t, int L,int R)
{
	return query(t, R) - query(t, L);
}
```

求[L,R]的区间和

利用前缀和相减的性质：[ L , R ] = [ 1 , R ] − [ 1 , L − 1 ]

## 区间修改

对a数组区间[L,R]元素值都+k，需要利用差分数组：（每个元素维护i值和i-1值的差）

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
void add(int* c, int index, int k, int n) {
    for (int i = index; i <= n; i += lowbit(i)) {
        c[i] += k;
    }
}
void range_add(int* c, int L, int R, int val, int n) {
    add(c, L, val, n);
    if (R + 1 <= n) {
        add(c, R + 1, -val, n);
    }
}
```

## 前缀查询

求出c数组的前缀和

```c++
int query(int* c, int index){
    int sum = 0;
    for(int i=index; i; i-=lowbit(i)){// i > 0
    sum+=c[i];
    }
    return sum;
}
```

# 并查集

思想：使用树形数据结构，每棵树维护一个集合，集合合并==树合并，是否同一集合 == 是否同一树

使用场景：用于处理不相交集合的合并和查询问题

```c++
vector<int> parent;
void init(int n)//初始化：让每个节点的父节点为自身
{
    for (int i = 1; i <= n; ++i)
        parents[i] = i;
}
int findRoot(int x)//递归向上查找节点x所在树的根节点
{
    return x == parents[x] ? x : findRoot(parents[x]);
}
void merge(int i, int j)//合并集合
{
    parents[findRoot(i)] = findRoot(j);
}
bool isConnected(p,q)//判断两个节点是否属于同一集合
{
    return findRoot(p) == findRoot(q);
}
int findRoot(int x)//路径压缩
{
    return x == parents[x] ? x : (parents[x] = findRoot(parents[x]));
}
```

* 路径压缩：查询一个节点的根节点，如果深度很大，那么要花费很多时间，如何做到O(1)的时间查询呢？每次findRoot都相当于对树的重构，将所有所属同一集合的元素，都直接指向根

# 单调栈

* 思想：从 栈底 到 栈顶 的元素值 是单调递增 / 单调递减
* 单调递增栈：
  * 从 栈底 到 栈顶 的元素值 是单调增的
  * 入栈：只有比栈顶元素大的元素才能直接进栈，否则需要先将栈中比当前元素大的元素出栈，再将当前元素入栈
* 使用场景：快速求解某个元素左边/右边第一个比它大/小的元素

**问题：**

对于给定的整数数组nums

找到每个元素右侧第一个比自己大的数的下标，如果没有，填-1

找到每个元素左侧第一个比自己大的数的下标，如果没有，填-1

```c++
vector<int> solve(vector<int>& nums) {
  int n = nums.size();
  vector<int> res(n, -1);//元素初始为-1
  stack<int> st;

  for (int i = 0; i < n; ++i) {
    while (!st.empty() && nums[i] > nums[st.top()]) {
      res[st.top()] = i;//出栈时操作
      st.pop();
    }
    st.push(i);
  }
  return res;
}

vector<int> solve(vector<int>& nums) {
  int n = nums.size();
  vector<int> res(n, -1);
  stack<int> st;

  for (int i = 0; i < n; ++i) {
    while (!st.empty() && nums[i] > nums[st.top()]) {
      st.pop();
    }
    res[i] = st.empty() ? -1 : st.top();//入栈时操作
    st.push(i);
  }
  return res;
}
```

总结：

| 求右侧第一个比自己大的元素 | 单调递减栈，出栈时操作 |
| -------------------------- | ---------------------- |
| 求右侧第一个比自己小的元素 | 单调递增栈，出栈时操作 |
| 求左侧第一个比自己大的元素 | 单调递减栈，入栈时操作 |
| 求左侧第一个比自己小的元素 | 单调递增栈，入栈时操作 |

# 字典/前缀树

思想：使用树形数据结构（每个节点有26个next指针），保存所有字符串，将每个字符串的每个字符，按照字典顺序存放

使用场景：快速查找所有单词中是否有某前缀，支持动态更新（添加新单词）

```c++
struct Node{
    Node* nexts[26];
    unordered_set<string> words;
    Node(){
        for (int i = 0; i < 26; i++) {
            nexts[i] = nullptr;
        }
    }
};

class Trie {
public:
    Trie() {
        root = new Node();
    }
    
    void insert(string word) {
        Node* ptr = root;
        for(auto& c : word){
            if(ptr->nexts[c - 'a'] == nullptr)ptr->nexts[c - 'a'] = new Node();
            ptr = ptr->nexts[c - 'a'];
        }
        ptr->words.insert(word);
    }
    
    bool search(string word) {
        Node* ptr = root;
        for(auto& c : word){
            if(ptr->nexts[c - 'a'] == nullptr)return false;
            ptr = ptr->nexts[c - 'a'];
        }
        return ptr->words.find(word) != ptr->words.end() ? true : false;
    }
    
    bool startsWith(string prefix) {
        Node* ptr = root;
        for(auto& c : prefix){
            if(ptr->nexts[c - 'a'] == nullptr)return false;
            ptr = ptr->nexts[c - 'a'];
        }
        return true;
    }

private:
    Node* root;
};
```