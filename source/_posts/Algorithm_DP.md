---
title: Dynamic Programming
date: 2018/12/3
categories: Algorithm
---

## 动态规划
动态规划的原则就是讲复杂问题分为若干个简单的子问题依次求解，在某些地方有那么一点像递归，不过由于递归的循环调用会使得某些计算重复，使用动态规划的优化思路就是保存已经计算好的值。

## Fibonacci
Fibonacci以前都是用递归写的，但是这样的话就会有很多重复的值或者是已经计算好的值没有重复利用。
```java
private void fib() {
    fibresult[0] = 1;
    fibresult[1] = 1;
    for (int i = 2; i < n; i++) {
        fibresult[i] = fibresult[i - 1] + fibresult[i - 2];
    } 
}
```
不过如图中所示，fib(4)调用了两次，fib(3)调用了三次等等。

![](/assets/Algorithm_DP/fib_tree.png)

因此如果能把fib(4)和fib(i)保存起来的话会好很多。
```java
private void fib() {
    DP[0] = DP[1] = 1; 
    for (i = 2; i <= n; i++) {
        DP[i] = DP[i-1] + DP[i-2];
    }
}
```

## 卖酒的最高收益
酒架上有N瓶酒，每瓶酒的价格不同，分别为pi，酒的价格随着年份增长，比如一瓶酒原来10，假设今年为year 1，等到了year y，酒的价格就变为了10 * y。

现在你想把这些酒全卖了，但是有两个规定，一是一年只能卖一瓶，二是每次都只能从酒架上的最左边或者最右边卖酒。要求出最高收益。

比如有这四瓶酒，p1=1, p2=4, p3=2, p4=3，那么最高收益就是1 * 1 + 3 * 2 + 2 * 3 + 4 * 4 = 29。

很明显需要用到递归，每一次都取最左边的收益和最右边收益之间的最大值即可。
```java
private static int maximumSellWineProfie(int year, int begin, int end) {
    if (begin > end) {
        return 0;
    }
    return Math.max(maximumSellWineProfie(year + 1, begin + 1, end) + year * p[begin],
            maximumSellWineProfie(year + 1, begin, end - 1) + year * p[end]);
}
```

不过year这个变量似乎是用不上的，因为他是可以根据酒瓶的数量计算出来的。因此将year去掉的话
```java
private static int maximumSellWineProfie(int begin, int end) {
    if (begin > end) {
        return 0;
    }
    int year = p.length - (end - begin + 1) + 1;
    return Math.max(maximumSellWineProfie(begin + 1, end) + year * p[begin],
                    maximumSellWineProfie(begin, end - 1) + year * p[end]);
}
```

ok，现在开始有点像之前的Fibonacci了，那么接下来的任务就是将计算的值保存下来，因为有两个变量所以需要一个二维数组。
```java
int N;
int p[N];
int cache[N][N];

private int profit(int be, int en) {
    if (be > en)
        return 0;
    // initial value
    if (cache[be][en] != -1)
        return cache[be][en];
    int year = N - (en-be+1) + 1;
    return cache[be][en] = max(
        profit(be+1, en) + year * p[be],
        profit(be, en-1) + year * p[en]);
}
```
可以发现动态规划也不是一步实现的，而是先用递归或者循环，然后删除一些不需要的变量，避免重复的计算加以优化即可。
