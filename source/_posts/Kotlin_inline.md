---
title: Inline
date: 2019/6/3
categories: Kotlin
---

C++中有`inline`关键字，传统函数中我们调用函数的时候，其实是将该指令的内存地址保存下来，并将函数参数复制到堆栈。当函数运行完之后再跳回去。这样来回跳跃会加大开销。而如果一个函数被`inline`修饰的话，在编译的时候则是直接将该函数的body拷贝到执行的地方。

Kotlin中也有相同的关键字，一般用于高阶函数，想知道有什么变化，还是看对应的Java文件更清晰点。

(IntelliJ的Decompiler不太靠谱，所以这里用的JD-GUI)

源文件：

```kotlin
class User {
    fun nonInlined(block: () -> Unit) {
        println("before")
        block()
        println("after")
    }

    inline fun inlined(block: () -> Unit) {
        println("before")
        block()
        println("after")

    }
}

fun main(args: Array<String>) {
    val user = User()

    user.nonInlined { println("noninline") }

    println("**********")

    user.inlined { println("inline") }
}



```

```java
public final class UserKt
{
  public static final void main(@NotNull String[] args) {
    Intrinsics.checkParameterIsNotNull(args, "args"); User user = new User();
    
    user.nonInlined((Function0)UserKt$main$1.INSTANCE);
    
    String str1 = "**********"; boolean bool1 = false; System.out.println(str1);
    
    User this_$iv = user; int $i$f$inlined = 0;
  
    String str2 = "before"; boolean bool2 = false; System.out.println(str2);
    int $i$a$-inlined-UserKt$main$2 = 0; String str3 = "inline"; boolean bool3 = false; System.out.println(str3);
    str2 = "after"; bool2 = false; System.out.println(str2);
  }
  
  static final class UserKt$main$1 extends Lambda implements Function0<Unit> {
    public static final UserKt$main$1 INSTANCE = new UserKt$main$1();
    
    public final void invoke() {
      String str = "noninline";
      boolean bool = false;
      System.out.println(str);
    }
    
    UserKt$main$1() { super(0); }
  }
}
```

可以看到用`inline`修饰的部分直接copy了函数体到调用的地方。和C++上的作用一样。

关于这篇文章：<https://stackoverflow.com/questions/44471284/when-to-use-an-inline-function-in-kotlin>解释的非`inline`方法会创建一个实例结合这个例子来看有些不太匹配的地方。文章中说Kotlin中调用高阶函数会创建一个匿名内部类，类似这样：

```java
nonInlined(new Function() {
    @Override
    public void invoke() {
        System.out.println("do something here");
    }
});
```



但是从JD decompile的结果来看，它是创建了一个静态的实例。猜测是新的Kotlin版本进行了优化。

```java
static final class UserKt$main$1 extends Lambda implements Function0<Unit> {
    public static final UserKt$main$1 INSTANCE = new UserKt$main$1();
    
    public final void invoke() {
      String str = "noninline";
      boolean bool = false;
      System.out.println(str);
    }
    
    UserKt$main$1() { super(0); }
  }
```

需要注意的是`inline`修饰的函数无法调用`private`修饰的方法或者变量

```kot
inline fun inlined(block: () -> Unit) {
    dummy()     //compilation error
    println(i)     //compilation error
}
```

Anyway，`inline`关键字和C++中的一样，都是为了减少内存开销。但它的副作用在于额外增加了字节码。因此如果一个函数体的逻辑比较复杂则不那么适合用`inline`。