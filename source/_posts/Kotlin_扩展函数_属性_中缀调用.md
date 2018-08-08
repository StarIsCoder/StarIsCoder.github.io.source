---
title: Kotlin-扩展函数/属性/中缀调用
date: 2018/07/28
categories: Kotlin
---
## Kotlin - 扩展函数
扩展函数：*它是一个类的成员函数，不过是定义在类的外面。*
比如说定义一个String的扩展函数计算该字串的最后一个字符
```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
```
当我们想调用该函数的时候
```kotlin
println("kotlin".lastChar())
```
Warning: 扩展函数不能访问到私有的或者受保护的成员。

扩展函数不存在重写的情况，当一个类的成员函数和扩展函数拥有相同的签名时，成员函数会被优先使用。
```kotlin
class Button {  
    fun test() {  
        println("This is member fun")  
    }  
}
val button: Button = Button()  
button.test()
```
*Result : This is member fun*
## Kotlin - 扩展属性
扩展属性和扩展函数类似，就像接收者的一个普通成员属性一样。
Warning: 必须定义getter函数，不能初始化，因为没有地方存储初始值。
```kotlin
var StringBuilder.lastChar: Char  
    get() = get(this.length - 1)  
    set(value: Char) {  
        this.setCharAt(length - 1, value)  
    }
val sb = StringBuilder("kotlin?")  
sb.lastChar = '!'  
println(sb)
```
Result : kotlin!
具体在实战中如何使用暂时不知。不过在Collection中已经默认增加了很多扩展函数，例如last（用来获取列表中最后一个元素）或者max（获取集合中的最大值）。这些在IDE中都会在补全功能中显示出来。

## Kotlin - 中缀调用
中缀调用：调用函数的时候不加分隔符，直接在对象和参数之间的调用方式。
```kotlin
val map = mapOf(1 to "one", 7 to "seven")
```
等价于
```kotlin
val map = mapOf(1.to("one"), 7.to("seven"))
```
中缀调用可以与*只有一个参数*的函数一起使用，并且需要用infix修饰符来标记它。
例如上述to函数的声明
```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```
个人理解增加函数调用的可读性。