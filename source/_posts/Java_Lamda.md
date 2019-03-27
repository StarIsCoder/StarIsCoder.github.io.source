---
title: Lamda表达式
date: 2018/10/04
categories: Java
---

## final关键字
Lamda表达式中引用的局部变量必须是final或既成事实上的值。
换句话说Lamda表达式需要的是值而不是变量，如果一个变量被final修饰了，那么表示它只会被赋值一次，那么便是个定值，而如果我们对某个普通变量多次赋值，对Lamda表达式来说他不知道到底该用哪个值。
```java
String str = "hello";
str = "world";
Runnable runnable = () -> {
	//Variable used in lambda expression should be final or effectively final
    System.out.println(str);
};
```
```java
String str = "hello";
Runnable runnable = () -> {
	//valid
    System.out.println(str);
};
```
## Predicate接口
该接口表示传入的对象是否满足某种条件，并返回boolean作为真假判断。check下源码，该接口只接受一个参数，返回一个布尔值。
```java
public interface Predicate<T> {

    boolean test(T t);
    .......
}
```
例如我们希望从一个int的列表中找到所有的偶数，那么Predicate可以这么写。
```java
Predicate<Integer> predicate = integer -> {
    return integer % 2 == 0;
};

predicate.negate();// 过滤出所有的奇数
```

## 总结
- Lamda表达式是一个匿名的方法，将行为像数据一样传递。
- Lamda表达式也被称为闭包，未赋值的变量与周边环境隔离起来，进而被绑定到一个特定的值。