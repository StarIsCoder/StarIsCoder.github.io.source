---
title: 装饰器
date: 2018/10/13
categories: Java
---

以奶茶店为例子，如果有红茶和绿茶两种，那么应该是这样的
```java
abstract class milktea {
    String description;

    public String getDescription() {
        return description;
    }

    void show() {
        System.out.println(getDescription());
    }
}

class greenMilktea extends milktea {
    public greenMilktea() {
        description = "green milktea";
    }
}

class redMilktea extends milktea {
    public redMilktea() {
        description = "red Milktea";
    }
}
```
都知道奶茶中可以加各种奇奇怪怪的东西，像燕麦，珍珠之类的，如果要加这些东西该怎么写比较好呢。

第一种将所有能加的东西排列组合，并增加对应的类
- GreenMilkTeaWithPearl
- GreenMilkTeaWithSugar
- GreenMilkTeaWithSugarAndPearl
- RedMilkTeaWithPearl
- RedMilkTeaWithSugar
- RedMilkTeaWithSugarAndPearl

这才只有两种配料，因此这种方式肯定不可取。

第二种添加成员属性
```java
abstract class Milktea {
	//配料
    boolean addSugar;
    boolean addPearl;
    boolean addIce;
    
    String description;

    public String getDescription() {
        return description;
    }

    void show() {
        System.out.println(getDescription());
    }
}
```
这种相对好一点但是也有很多冗余的代码。其实最好的方式还是按照奶茶店的模式来，将红茶和绿茶作为饮品，而珍珠、糖之类的作为配料单都拿出来。也就是先把茶给制作出来，然后在对这个茶加不同的配料或者是对这个奶茶进行装饰。

那么我们做茶的配方（代码）是没有变的。我们需要增加配料就可以了。这里加了冰激淋和糖。
```java
class Icecream extends Ingredients {
    Milktea mt;

    public Icecream(Milktea mt) {
        this.mt = mt;
    }

    @Override
    public String getDescription() {
        return mt.getDescription() + " add Icecream";
    }
}

class Sugar extends Ingredients {
    Milktea mt;

    public Sugar(Milktea mt) {
        this.mt = mt;
    }

    @Override
    public String getDescription() {
        return mt.getDescription() + " add Sugar";
    }
}
```
有个疑问为什么要传入奶茶对象呢，因为当然要知道冰激凌要加在哪一杯奶茶中啊。
```java
public static void main(String[] args) {
	//没有任何配料的奶茶
    Milktea mt = new GreenMilktea();
    mt.show();
	
	//制作一杯红茶
    Milktea mt2 = new RedMilktea();
   	//加冰淇淋
    mt2 = new Icecream(mt2);
    //加糖
    mt2 = new Sugar(mt2);
    //交给客户
    mt2.show();
}
```
```
Result:
green Milktea
red Milktea add Icecream add Sugar
```
看起来就像是把一个对象作为参数传递进去并包装了一下，通过层层嵌套的形式将一个对象包装起来。如果又来一个新配料也好办，使用方式都是一样的，新增一个类，将想要装饰的类传进去。
```java
......
psvm {
    mt2 = new Pearl(mt2);
}


class Pearl extends Ingredients {
    Milktea mt;

    public Pearl(Milktea mt) {
        this.mt = mt;
    }

    @Override
    public String getDescription() {
        return mt.getDescription() + " add Pearl";
    }
}
```
## 总结
好处：
- 完全不用修改原有的代码，容易扩展新的功能。
- 原始类可以非常简单，之后根据需要对原始类进行包装。

坏处：
- 由于是嵌套的初始化形式，因此看起来很复杂。如果层数多了跟代码会特别困难。

不管怎么说设计模式这种东西一定不能生搬硬套，而是要灵活应用。

