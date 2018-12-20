---
title: Power of N
date: 2018/12/19
categories: Algorithm
---

在判断是否为n的次方时发现有很多种有意思的解法。当然最容易想到的是一定是不断除n，观察最后是否能除干净。这里以2为例子，之后都可以讲2替换成3或者i来使用loop。

```java
public  boolean isPowerOfTwo(int n) {
    if (n < 1) return false;
    while (n % 2 == 0) {
         n /= 2;
    }
    return n == 1;
}
```

## 2的次方
由于如果n是2的次方，那么转换成2进制一定会是1 + n个0的形式。因此可以考虑将其转为2进制然后再判断
```java
public  boolean isPowerOfTwo(int n) {
    if (n < 1) return false;
    String str = Integer.toBinaryString(n);
    System.out.println(str);
    char array[] = str.toCharArray();
    if (array[0] != '1') return false;
    for (int i = 1; i < array.length; i++) {
        if (array[i] != '0') return false;
    }
    return true;
}
```

网上还看到一种比较tricky的，那就是如果n是2的次方，那么n-1对应的二进制一定全是1，只要将两者做与运算即可
```java
public  boolean isPowerOfTwo(int n) {
    if (n < 1) return false;
    int ret = n & (n - 1);
    return ret == 0;
}
```

还可以使用正则表达式来对n的二进制进行判断
```java
public  boolean isPowerOfTwo(int n) {
    if (n < 1) return false;
    String str = Integer.toBinaryString(n);
    return str.matches("^10*$");
}
```

## 3的次方
Java中提供了将传入的数转化成你想要的进制数，`Integer.toString(n, radix)`，意思是将n转化为radix进制的数。
```java
Integer.toString(8, 2) -> log2(8) = 3, return 1000
Integer.toString(81, 3) -> log3(81) = 4, return 10000
```

因此可以使用这个api然后判断是否为1 + n个0的形式
```java
public static boolean isPowerOfThree(int n) {
    String baseChange = Integer.toString(n, 3);
    return baseChange.matches("^10*$");
}
```

还有就是用数学上的log公式了，logb(n) / logb(m) = logm(n),同时由于Math的log方法返回的是double类型，需要用 % 1 是否为零来判断是否为integer类型。
```java
public static boolean isPowerOfThree(int n) {
    return (Math.log10(n) / Math.log10(3)) % 1 == 0;
}
```

最后一种也非常的tricky，由于int类型是有范围的，因此可以取最大的3的次方数来除以n。

```java
public boolean isPowerOfThree(int n) {
    return n > 0 && 1162261467 % n == 0;
}
```

## 4的次方
由于4的次方和2的次方，因此可以在2的次方上过滤。首先4的次方一定也是1 + n个0的形式，但不同的是它的零的个数一定是偶数。
```java
public  boolean isPowerOfFour(int num) {
    return Integer.bitCount(num) == 1 && (Integer.toBinaryString(num).length() - 1) % 2 == 0;
}
```
还有种方式同样是用二进制，由于4的次方数转化成2进制之后，1所在的位置一定是奇数位，从0计数
eg.
8  -> 1000
16 -> 10000
32 -> 100000
因此只需要过滤到偶数位的即可

num&(num-1)是为了判断是否为2的次方，而由于5 -> 0101，是用来过滤偶数位的1。

```java
return num > 0 && (num&(num-1)) == 0 && (num & 0x55555555) != 0;
```