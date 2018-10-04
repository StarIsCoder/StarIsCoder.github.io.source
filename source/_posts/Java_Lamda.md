---
title: Lamda表达式-Preview
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

## 总结
- Lamda表达式是一个匿名的方法，将行为像数据一样传递。
- Lamda表达式也被称为闭包，未赋值的变量与周边环境隔离起来，进而被绑定到一个特定的值。