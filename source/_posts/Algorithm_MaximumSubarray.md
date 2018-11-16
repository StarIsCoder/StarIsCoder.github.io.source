---
title: Maximum Subarray
date: 2018/10/19
categories: Algorithm
---

Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

Example:
```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```
先来一个暴力穷举法：
```java
int sum = 0;
int max = nums[0] > 0 ? 0 : nums[0];
for (int i = 0; i < nums.length; i++) {
    for (int j = i; j < nums.length; j++) {
        sum += nums[j];
                max = Math.max(sum, max);
        }
        sum = 0;
    }
return max;
```
这里是从0 ~ i递增的方式累加，重新换种递减的方式
```java
int sum = 0;
int max = nums[0] > 0 ? 0 : nums[0];
for (int i = 0; i < nums.length; i++) {
    for (int j = i; j >= 0; j--) {
        sum += nums[j];
        max = Math.max(sum, max);
    }
    sum = 0;
    }
return max;
```
在累加的过程中可以把这个内层的循环优化一下。例如有个数组{1,2,3,-5,4}
```
1 + 2 + 3 + -5 + 4 =  (1 + 2 + 3 + -5) + 4
1 + 2 + 3 + -5 = (1 + 2 + 3) + -5
1 + 2 + 3 = (1 + 2) + 3
1 + 2 = (1) + 2
1 = 1
```
如果抽象一下的话就是
```
sum(i) = A[i] + sum(i - 1)
```
那么如果是求之前总和的最大值，就是比较一下A[i] + sum(i - 1) 和 A[i]或者也可以说是sum(i - 1)是否大于零
```java
maxSum(i) = max(A[i], A[i] + sum(i - 1))
```
因此优化后的代码为
```java
int max = nums[0];
int maxSum[] = new int[nums.length];
maxSum[0] = nums[0];
for (int i = 1; i < nums.length; i++) {
    maxSum[i] = Math.max(nums[i], maxSum[i - 1] + nums[i]);
    max = Math.max(maxSum[i], max);
}
return max;
```
由于maxSum的数组不是必要的，可以用一个变量来代替。
```java
 int max = nums[0];
int maxSum = nums[0];
for (int i = 1; i < nums.length; i++) {
    maxSum = Math.max(nums[i], maxSum + nums[i]);
    max = Math.max(maxSum, max);
}
return max;
```

参考文章：http://theoryofprogramming.com/2016/10/21/dynamic-programming-kadanes-algorithm/