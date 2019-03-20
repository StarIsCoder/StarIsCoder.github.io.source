---
title: Bytecode Execution Engine
date: 2019/03/19
categories: Java
---

## 运行时栈帧结构

栈帧(stack frame)是用于支持虚拟机方法调用和方法执行的数据结构。它是虚拟机运行时数据区的虚拟机栈的栈元素。栈帧存储了局部变量、操作数栈、方法返回地址等。一个方法的调用到结束对应了一个栈帧的入栈和出栈。

### 局部变量表(Local Variable Table)
局部变量表是一组变量值存储空间，用于存放参数和方法内部定义的局部变量。最小的单位是变量槽（Variable Slot）。虚拟机是通过索引来定位哪个槽的。如果执行的是非static方法，默认第0位是this。

为了节省栈帧空间，局部变量表中的slot是可以复用的。因为方法中定义的局部变量不一定会覆盖整个方法体，比如for循环中的变量。

slot复用可能导致GC问题。
```java
public static void main(String[] args) {
    {
        byte[] holder = new byte[64 * 1024 * 1024];
    }
    System.gc();
}
```
以上代码由于holder所占用的slot一直没人再使用，所以GC没法回收holder。如果希望回收holder，则只需要随便定义一个变量即可。
```java
public static void main(String[] args) {
    {
        byte[] holder = new byte[64 * 1024 * 1024];
    }
    int a = 0;
    System.gc();
}
```
### 操作数栈
也就是操作栈，方法执行过程中会有各种字节码指令向操作栈写入或者读取内容。用1+1来举例的话。先将两个1压入栈中，用过add指令将最顶上的两个元素弹出并相加之后将结果压入栈中。
### 方法返回地址
当一个方法开始执行之后，只有两种方式可以退出，一种是正常结束比如return。另一种则是运行过程中出现了异常。方法退出的实际过程就是等于把栈帧出栈。因此可能会恢复上层的局部变量表和操作栈，并将返回值压入调用者的操作栈中。

## 方法调用
方法调用并不等于方法执行，唯一的任务就是决定调用哪一个方法。

### 解析
在类加载的解析阶段主要就是将符号引用转化为直接引用，但是这样的一个前提就是在真正运行之前就能确定到底调用那个版本的方法并且在运行期间是不可改变的。符合这种“编译期可知，运行期不可变”的方法主要是static和private。

Java虚拟机有5条方法调用字节码指令：
1. invokestatic: 调用静态方法。
2. invokespecial: 调用实例构造器，私有、父类方法。
3. invokevirtual: 调用虚方法。
4. invokeinterface: 调用接口方法。
5. invokedynamic: 运行时动态解析。
以上1和2调用的方法都可以在解析阶段就确定唯一的版本。这类方法称为非虚方法，其他的称为虚方法。比较特殊的是final方法，虽然final是用invokevirtual指令来调用的但是由于它无法被重写因此也属于非虚方法。

### 分派
分派主要分为静态单分派、静态多分派、动态单分派、动态多分派。先来确定两个概念：静态类型和实际类型。
```java
static class Father {

}

static class Son extends Father {

}

Father son = new Son();
```
上述代码中的`Father`叫做静态类型或外观类型，而`Son`叫做实际类型。两者最大的区别在于静态类型是编译期可知的，而实际类型是运行期间才可知的。
#### 静态分派
顾名思义就是根据静态类型来决定方法版本的分派叫做静态分派。比如
```java
public class Test {
    static class Human {
    }

    static class Man extends Human {
    }

    static class Woman extends Human {

    }

    void helloWorld(Human guy) {
        System.out.println("guy");
    }

    void helloWorld(Man man) {
        System.out.println("man");
    }

    void helloWorld(Woman woman) {
        System.out.println("woman");
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        Test test = new Test();
        test.helloWorld(man);
        test.helloWorld(woman);
    }
}
```
```
Result:
guy
guy
```
在确定方法接受者是`test`之后，决定哪个方法的版本只和方法的参数数量和参数类型有关。而方法的参数类型中只有静态类型。

静态分派在虚拟机中的应用就是重载，重载是通过参数的静态类型来决定的。静态分派发生在编译期间，因此确定静态分派的动作不是由虚拟机执行的。

#### 动态分派
动态分派就是根据实际类型来决定方法版本。
```java
public class Test {
    static class Human {
    }

    static class Man extends Human {
        void sayHello() {
            System.out.println("Man");
        }
    }

    static class Woman extends Human {
        void sayHello() {
            System.out.println("Woman");
        }
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        ((Man) man).sayHello();
        ((Woman) woman).sayHello();

    }
}
```
`sayHello`的版本决定于`new`之后的实际类型。这就是动态分派应用在了重写中。

使用javap查看字节码发现这里调用方法的指令用的是`invokevirtual`。这就有点类似于C++中`virtual`关键字的作用。

`invokevirtual`指令的解析过程主要有以下：
1. 找到操作数栈顶第一个元素所指向的对象的实际类型。
2. 如果找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则直接使用这个方法的引用。
3. 否则，从下往上依次检索父类并且验证。
4. 如果还找不到就跑出AbstractMethodError异常

重写的本质就是在运行期间把方法的符号引用解析到不同的直接引用上。

#### 动态分派的实现
由于动态分派比较耗时，因此在方法区中建立一个虚方法表(Virtual Method Table)，对于invokeinterface则是接口方法表(Interface Method Table)，也就是用索引来提高性能，这一点又和C++的VTable非常相似。方法表中每个索引对应方法的实际入口，如果子类未实现则会直接指向父类。

关于Java动态类型、MethodHandle、Reflection之后另写一篇。