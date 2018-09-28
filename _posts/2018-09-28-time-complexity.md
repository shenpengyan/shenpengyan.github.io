---
layout:     post
title:      "算法之复杂度分析(极客时间《数据结构与算法之美》课程1-4学习笔记)"
subtitle:   "time-complexity"
date:       2018-09-28 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - 软件工程
---


#### 数据结构和算法的定义

##### 广义上：
1. 数据结构是指一组数据的存储结构
2. 算法是操作数据的一组方法

##### 狭义上，就是下面这张图

![shujujiegou](https://shenpengyan.github.io/img/in-post/time-complexity/shujujiegou.png)

##### 重点：

10个数据结构：
数组、链表、栈、队列、散列表、二叉树、堆、跳表、图、Trie树

10个算法：递归、排序、二分查找、搜索、哈希算法、贪心算法、分治算法、回溯算法、动态规划、字符串匹配算法。


#### 时间复杂度分析

方法：
1. 只关注循环执行次数最多的一段代码
2. 加法法则：总复杂度等于量级最大的那段代码的复杂度
3. 乘法法则：嵌套代码的复杂度等于嵌套内外代码复杂度的乘积

大O表示法：只关心循环执行次数最多的代码，且需要值为变量，无论常量的取值多大，在变量非常大的时候，都可以忽略。

#### 常见的复杂度量级

按数量级递增：

常数阶O(1)、对数阶O(logn)、线性阶O(n)、线性对数阶O(nlogn)、平方阶O(n^2) O(n^3)... k次方阶O(n^k)
指数阶O(2^n)、阶乘阶O(n!)

##### 1. O(1)

示例代码：

```
int i = 8;
int j = 6;
int sum = i + j;
```

只要代码的执行时间不随 n 的增大而增长，这样代码的时间复杂度我们都记作 O(1)。或者说，一般情况下，只要算法中不存在循环语句、递归语句，即使有成千上万行的代码，其时间复杂度也是O(1)


##### 2. O(logn)、O(nlogn)

示例代码：

```
i=1;
while (i <= n)  {
  i = i * 2;
}
```

实际上，不管是以 2 为底、以 3 为底都是可以相互转换的，log3n 就等于 log32 * log2n, 因此统一叫做O(logn)

##### 3. O(m+n)、O(m*n)

如果有两个变量m，n，则分析时m，n都不能丢。

##### 之前的时间复杂度的全称是渐进时间复杂度，表示算法的执行时间与数据规模之间的增长关系

#### 最好、最坏、平均、均摊时间复杂度

```
// n 表示数组 array 的长度
int find(int[] array, int n, int x) {
  int i = 0;
  int pos = -1;
  for (; i < n; ++i) {
    if (array[i] == x) {
       pos = i;
       break;
    }
  }
  return pos;
}

```

最好时间复杂度是O(1)， 最坏时间复杂度是O(n)

平均时间复杂度：需要考虑每个值出现的概率情况，来做计算。大多数情况下，不需要区分最好、最坏、平均时间复杂度三种情况。


上面的例子：
假设出现在数组中和不出现在数组中的概率各1/2，则平均时间复杂度的计算过程为：

![](https://shenpengyan.github.io/img/in-post/time-complexity/time1.png)

均摊时间复杂度

```
 // array 表示一个长度为 n 的数组
 // 代码中的 array.length 就等于 n
 int[] array = new int[n];
 int count = 0;
 
 void insert(int val) {
    if (count == array.length) {
       int sum = 0;
       for (int i = 0; i < array.length; ++i) {
          sum = sum + array[i];
       }
       array[0] = sum;
       count = 1;
    }

    array[count] = val;
    ++count;
 }

```

只有当count == array.length时，是O(n), 其他时候，是O(1), 从概率的角度，O(n)的概率是1/(n+1)，O(1)的概率为n/n+1，则均摊时间复杂度为O(1), 均摊时间复杂度是一种特殊的平均时间复杂度。

> 总结：时间复杂度，关注变量和每个变量的最高阶
