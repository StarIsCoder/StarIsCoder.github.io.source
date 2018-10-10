---
title: 并行化
date: 2018/10/10
categories: Java
---

## 并行和并发的区别
- 并行：并行更着重于硬件层面，在不同的cpu上同时工作。
- 并发：并发更着重与软件层面，描述的情况是两个以上的action可能同时发生。

## stream的并行化处理
调用parallelStream方法即可将stream并行化。
```java
//list中的元素求和
result = linkedList.stream().parallel().mapToInt(integer -> integer).sum();
```
对应的串行化则是sequential()方法。
```java
result = linkedList.stream().sequential().mapToInt(integer -> integer).sum();
```

那么我们check下两者的运行时间
```java
public static void main(String[] args) throws Exception {
    List<Long> times = new ArrayList<>();

    List<Integer> linkedList = new LinkedList<>();
    for (int i = 0; i < 100000000; i++) {
        linkedList.add(10);
    }
    int result = 0;
    for (int i = 0; i < 10; i++) {
        long startTime = System.currentTimeMillis();
        //Parallel
        result = linkedList.stream().parallel().mapToInt(integer -> integer).sum();
        //Sequential
        result = linkedList.stream().sequential().mapToInt(integer -> integer).sum();
        System.out.println(result);
        times.add(System.currentTimeMillis() - startTime);
    }
    System.out.println(getAverage(times));

}

static private long getAverage(List<Long> times) {
    long average = times.stream().reduce((acc, element) -> acc + element).get() / times.size();
    return average;
}
```
```
Parallel Result:1754
Sequential Result:249
```
可以看到并行并不是一定性能更好，并行能提升性能的前提在于数据量足够大，主要有四点
- 数据大小：如上例，只有输入的数据足够大的时候才有用。
- 源数据结构：上述例子使用的数据结构是LinkedList,之后用ArrayList看看效果。
- 装箱：处理基本类型要比装箱类型要快。
- 核的数量：如果只有一个cpu，那就没有并行的意义。换句话说核越多，性能提升幅度越大。

换成ArrayList试下
```java
List<Integer> linkedList = new ArrayList<>();
    for (int i = 0; i < 100000000; i++) {
        linkedList.add(10);
    }
    int result = 0;
    for (int i = 0; i < 10; i++) {
        long startTime = System.currentTimeMillis();
        result = linkedList.stream().parallel().mapToInt(integer -> integer).sum();
        times.add(System.currentTimeMillis() - startTime);
    }
    System.out.println(getAverage(times));
```
```
Result:86
```
可以看到性能一下子就提升了。因此数据结构对数据并行化影响很大。

因为求和原理是fork/join框架，也就是fork递归细分问题，最后通过join将数据整合，因此数据结构分解的性能至关重要。
- T1:ArrayList、数组或IntStream,支持随机读取。
- T2:HashSet、TreeSet等等。
- T3:上述的LinkedList，还有长度未知的BufferedReader.lines，因为不知从哪里开始分解。

## 数据的并行化处理
需要将一个数组作为参数传入Arrays.parallelSetAll()方法，不过需要注意的是**传入的数组会被修改**，并不是创建一个新的数组。
```java
public static void main(String[] args) throws Exception {
    double[] doubles = new double[10];
    for (int i = 0; i < 10; i++) {
        doubles[i] = i;
    }
    Arrays.parallelSetAll(doubles, i -> i + 0.5);
    for (double d : doubles) {
        System.out.println(d);
    }
}
```
```
Result:
0.5
1.5
2.5
3.5
4.5
5.5
6.5
7.5
8.5
9.5
```
## 并行化的陷阱
下面代码在串行和并行情况下结果不一样
```java
List<Integer> integerList = new LinkedList<>();
integerList.add(1);
integerList.add(2);
integerList.add(3);
integerList.add(4);
int i = integerList.stream().parallel().reduce(5, (acc, element) -> acc * element);
System.out.println(i);
```
原因在于并行化了之后每个元素都执行了reduce(5, (acc, element) -> acc * element)。

原来是5 * 1 * 2 * 3 * 4，并行化后变成了5 * (5 * 1) * (5 * 2) * (5 * 3) * (5 * 4)。
正确的应该是
```java
int i = integerList.stream().parallel().reduce(1, (acc, element) -> acc * element) * 5;
```