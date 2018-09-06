---
title: Kotlin Constructor
date: 2018/9/7
categories: Kotlin
---
## 构造函数
最近在自定义布局的时候发现kotlin的构造函数和java差的很多，因此重新学习记录下，先从简单的开始，模仿java中的写法
```kotlin
class User {
    constructor(i: Int)
    constructor(i: Int, j: Int)
    constructor(name: String, i: Int, j: Int)
}
```
```java
public final class User {
   public User(int i) {
   }

   public User(int i, int j) {
   }

   public User(@NotNull String name, int i, int j) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
   }
}
```
可以看到这种写法和java差不多。kotlin还允许在class定义的时候直接写构造函数，这种构造函数叫主构造函数。

```kotlin
class User constructor/*可省略*/(i: Int) {

}
```
```java
public final class User {
   public User(int i) {
   }
}
```
像这种直接在类名旁边的叫primary constructor，而类似于java那种写法的构造函数叫Secondary Constructors。主构造函数只能有一个，次构造函数可以有多个。但是由于主构造函数不能加代码，因此kotlin提供了init关键字
```kotlin
class User constructor/*可省略*/(i: Int) {
    init {
        println("primary constructor i = $i")
    }
}
```
```java
public final class User {
   public User(int i) {
      String var2 = "primary constructor i = " + i;
      System.out.println(var2);
   }
}
```
那如果想同时有主构造函数和次构造函数怎么办呢,按照官方api说的是需要或直接或间接的委托于主构造函数，一开始没看懂，后来看了转成的java源码，原来他说的委托就是在次构造函数中调用主构造函数
```kotlin
class User(name: String) {
    init {
        println("primary")
    }
    constructor() : this(String())
    constructor(i: Int) : this(String())
    constructor(j: Int, string: String) : this(string)
}
```
```java
public final class User {
   public User(@NotNull String name) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
      String var2 = "primary";
      System.out.println(var2);
   }

   public User() {
      this(new String());
   }

   public User(int i) {
      this(new String());
   }

   public User(int j, @NotNull String string) {
      Intrinsics.checkParameterIsNotNull(string, "string");
      this(string);
   }
}
```
从这边可以看到init代码块中的代码是在主构造函数中的，不过由于次构造函数都会调用主构造函数，所以也无需纠结这个init代码块是否会被调用。