---
title: Singleton
date: 2018/8/10
categories: Kotlin
---
## Object
在Kotlin中，object是一个自带单例模式的数据类型，而在java中object是所有类的父类。如下是object分别在java和kotlin的代码：
```kotlin
object ExampleObject {
    fun example() {
    }
}
```
```java
public final class ExampleObject {
   public static final ExampleObject INSTANCE;

   public final void example() {
   }

   static {
      ExampleObject var0 = new ExampleObject();
      INSTANCE = var0;
   }
}
```
可以看到，kotlin中的object是对应java中的静态单例模式。因此如果想在kotlin中写出单例模式，可以直接使用object关键字即可。同时因为使用的是static实现单例模式，因此也保证了线程安全。后续会介绍更好的写法。

## Java Singleton
这里其实对java中使用static来实现单例模式不是很清晰，主要的问题在于为什么static修饰的对象指向的是同一个内存地址。即使我这么写
```java
public class Single {
    public static Single single = new Single();
}

public static void main(String[] args) {
        Single single = Single.single;
        System.out.println(single);
        Single single1 = Single.single;
        System.out.println(single1);
}
```
Result:
```
Single@2b193f2d
Single@2b193f2d
```
静态单例模式还有另一种方式，区别在于只有调用getInstance方法时才会去实例化Single对象。
```java
public class Single {

    private static class SingleHoleder {
        private static Single single = new Single();
    }

    public static Single getInstance() {
        return SingleHoleder.single;
    }
}
```
这边static关键字能实现单例模式的主要原因在于被static修饰的变量或对象具有唯一性。那么为什么具有唯一性呢，这就得查看static的原理了。

## Static关键字
根据定义，被static所修饰的变量有且只有该变量的一份拷贝，并且能在类中公用。换句话说是一个全局的变量。静态变量在加载类的时候就已经分配好了内存，因此该变量是属于类的而不是某个new出来的对象的。在往下到jvm还未了解到,本文关于static到此结束。

## 更好的单例模式
```kotlin
public class Singleton private constructor() {
    init { println("This ($this) is a singleton") }    

    private object Holder { val INSTANCE = Singleton() }

    companion object {
        val instance: Singleton by lazy { Holder.INSTANCE }
    }
    var b:String? = null
}
```
* 将构造方法私有化，其他地方无法实例化该类。
* 使用lazy关键字延迟instance的实例化，优化性能。
* 使用companion关键字实现java中static的效果。

## kotlin中的companion
由于kotlin中object默认是单例模式，因此没有static关键字，那么如果我们想要全局变量该怎么做呢，答案就是companion。
```kotlin
class Test private constructor() {
    companion object {
        val i = 1
        val instance = Test()
    }
}

fun main(args: Array<String>) {
    println(Test.i)
}
```
```
Result : 1
```
这样就实现了static的效果。再来看一下decompile后的代码
```java
public final class Test {
   private static final int i = 1;
   @NotNull
   private static final Test instance = new Test();
   public static final Test.Companion Companion = new Test.Companion((DefaultConstructorMarker)null);

   public static final class Companion {
      public final int getI() {
         return Test.i;
      }

      @NotNull
      public final Test getInstance() {
         return Test.instance;
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```
可以看到变量i被static所修饰。并且companion是对所有类公开的，因此适合用来实现单例模式。

_Warning:一个类中只能有一个被companion修饰的object。_

参考文档：

https://antonioleiva.com/objects-kotlin/

https://medium.com/@adinugroho/singleton-in-kotlin-502f80fd8a63

https://www.journaldev.com/18662/kotlin-singleton-companion-object



