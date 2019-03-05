---
title: Volatile
date: 2018/9/27
categories: Java
---
## 原子性
原子性操作是指不能被线程调度机制中断的操作，一旦操作开始，在可能发生的上下文切换之前执行完。

java中对除了long和double以外的基本类型进行读写操作都是原子性的，因为long和double是64位的，JVM有可能将其分为两个32位的。不过新的jdk也实现了long和double的原子读写。

这里说的读写操作并不包括自加等。因为自加其实是分成了两步来走。
```java
i++

0:    aload_0
1:    dup
2:    getfield
5:    iconst_1
6:    iadd
7:    putfield
10:   return
```
这个操作有put和get，如果在put和get之间发生了变化，那么得到的结果可能不正确，因此不是原子性操作。当然java中有AtomicInteger、AtomicLong、AtomicBoolean等原子类。

## Volatile修饰词具有如下两种属性
### 可见性
通俗的说就是当这个值被修改了，那么所有的读取操作都可以看到这个修改。

对一个普通的变量i来说，正常的读写操作是先将变量从主内存拷贝到工作内存（缓存）中，进行一定的操作之后，再写入主内存，但是什么时候写入是不确定的。那么由于两个线程工作在两个cpu上，如果cpu1修改了i，但是没有及时写入主内存，而cpu2直接去读取i，结果就是这个i不是最新的。

用volatile修饰了的变量，在被修改之后会强制立即写入主内存。这样能保证在读取的时候一定是最新的。并且读取操作也是直接从主内存中读取。

因此volatile保证了对于其他线程的可见性。

### 有序性
在JVM中，编译器和处理器可能会对指令进行重排，或许是为了优化，但是如果被volatile修饰的变量则会告诉编译器不要重排指令，按照顺序执行就可以，避免了并发产生的问题。
```java
int a = 1;
int b = 2;

a++;
b++;
```
以上代码可以重排列成如下，因为a与b之间没有依赖。如果有依赖并且重排的话则会产生并发问题。
```java
int a = 1;
a++;

int b = 2;
b++;
```

但是在多线程中就不一定了，如下代码就可能出现并发问题：
```java
int a = 0;
bool flag = false;

public void write() {
a = 2; //1
flag = true; //2
}

public void multiply() {
if (flag) { //3
int ret = a * a;//4
}

}
```
如果有两个线程同时执行`write`和`multiply`方法，ret不是一定为4.如果`write`方法中的1和2重排序下，先赋值flag再赋值a，那么在`multiply`中可能读到a的值为0；

当然也可以使用synchronized和lock来保证有序，因为在某个时间段只有一个线程能执行被修饰的代码。


优先选择一定是sychronized，除非只有一个变量，否则都不应该只使用volatile，他并不能保证并发的冲突不发生。同样的也不能改变自加不是原子性的本质。

## happens-before
说到重排序，插播一段happens-before原则，这个原则是为了保证线程A所执行的action对其他线程执行的action是可见的。如果没有这个原则，那么JVM就会随心所欲的修改action执行的顺序。

以下是happens-before的几种规矩
1. 一个线程内的action
```
state 1
state 2	
state 3     // All states happen before state n

...

state n
```
2. 监视锁
在B线程获取锁之前，A线程会先将锁释放(同一个锁)。
3. 被volatile修饰的变量
线程A和线程B都用到了被volatile修饰的变量，那么写的操作一定发生在读之前。
4. 线程开始规则
`B.start()`一定发生在B的`run`方法中所有语句之前。
```java
Thread B = new Thread();
B.start();

//Thread B
public void run() {
    //state 1
    //state 2
}
```
5. 线程join规则
调用了`join`方法，则需等待子线程的所有语句执行完才能只能`join`之后的代码。
6. 传递性
如果A happens-before B，并且B happens-before C，那么A happens-before C。