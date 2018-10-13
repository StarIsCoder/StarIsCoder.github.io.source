---
title: Delegation
date: 2018/10/13
categories: Kotlin
---

##  类代理
面向对象的特性中由于继承的存在关系，因此当扩展一个类并自行重新定义一些细节的时候，代码就会变的依赖于这个子类，而如果后期父类更新了，那么子类的实现可能与父类背道而驰，因此kotlin中默认将类修饰为final。但有的时候确实需要添加一些特定情况下才发生的新功能。

因为在接口中定义的方法必须在实现类中全部实现，因此如果我们只是在原有的类上加一些新功能，那么会导致有很多重复的代码。

在kotlin中可以通过by省略掉很多不需要的代码。如下代码中的test方法不用重写直接使用代理，也就是circle的test方法即可。
```kotlin
fun main(args: Array<String>) {
    val redcircle = RedCircle(Circle())
    redcircle.draw()
    redcircle.test()
}

interface Shape {
    open val descpition: String
    fun draw()
    fun test()
}

class Circle : Shape {
    open override val descpition: String
        get() = "Circle"

    override fun draw() {
        println("draw $descpition")
    }

    override fun test() {
        println("test")
    }
}

class RedCircle(circle: Circle) : Shape by circle {
    override fun draw() {
        println(Circle().descpition + " is red")
    }

}
```
```
Result:circle is red
```
当然因为这里类比较简单，如果非常多的话例如collection接口，要实现五个方法，然而我们只是想新增一个方法，就会显得代码非常冗余
```kotlin
class MyCollection :Collection<String>{
    override val size: Int
        get() = TODO("not implemented") //To change initializer of created properties use File | Settings | File Templates.

    override fun contains(element: String): Boolean {
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }

    override fun containsAll(elements: Collection<String>): Boolean {
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }

    override fun isEmpty(): Boolean {
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }

    override fun iterator(): Iterator<String> {
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }
}
```
照例还是看下代理模式的java源码，这边只关注redCircle
```java
public final class redCircle implements Shape {
   // $FF: synthetic field
   private final circle $$delegate_0;

   public void draw() {
      String var1 = (new circle()).getDescpition() + " is red";
      System.out.println(var1);
   }

   public redCircle(@NotNull circle circle) {
      Intrinsics.checkParameterIsNotNull(circle, "circle");
      super();
      this.$$delegate_0 = circle;
   }

   @NotNull
   public String getDescpition() {
      return this.$$delegate_0.getDescpition();
   }

   public void test() {
      this.$$delegate_0.test();
   }
}
```
可以看到test方法由系统自动生成，并且是通过调用传入的circle对象来实现的。

像kotlin的by关键字非常类似于java中的装饰器。本质上都是将原始类作为属性传入并保存，如果想保留原有的设计，那么直接调用原始类的方法即可。
