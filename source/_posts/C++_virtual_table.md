---
title: virtual table
date: 2019/2/13
categories: C++
---

和同事在讨论设计api接口什么方法该设置为virtual时，他说一般有子类会覆写该方法的时候用virtual修饰下，但是不能全用virtual因为开销会大一些。之后在网上查了和virtual相关的资料。

C++中关于Virtual function的意义在StackOverflow上有很多很好的解释，这里想看下具体的原理。

https://stackoverflow.com/questions/2391679/why-do-we-need-virtual-functions-in-c

有个答案是说：如果没有virtual关键字那么就是early binding，意味着这个方法的实现在编译期间就根据指针的类型来决定了，而如果有virtual关键字那么就是late binding，它的方法的实现是在运行期间根据指针所指向的那个对象所决定的。

假设有两个类：
```c++
class Animal
{
    public:
        void eat() { std::cout << "I'm eating generic food."; }
};

class Cat : public Animal
{
    public:
        void eat() { std::cout << "I'm eating a rat."; }
};
```
定义一个函数来传入这个类
```c++
void func(Animal *xyz) { xyz->eat(); }
```
写入main函数测试下
```c++
Animal *animal = new Animal;
Cat *cat = new Cat;

func(animal); // Outputs: "I'm eating generic food."
func(cat);    // Outputs: "I'm eating generic food."
```
根据early binding，在调用该函数的时候就已经决定了`eat()`的实现是在`Animal`中的。当然如果用virtual修饰的话，那么根据late binding，`eat()`的实现是根据new关键字之后来决定的。

那么为什么用virtual修饰的就是late binding呢，这就涉及virtual table这个late binding的特殊形式，virtual table可以理解为每个有虚函数类的一组索引，这是编译器在编译期间就设置了的，同时编译器会设置一个隐藏的指针指向这个vtable，叫做`*_vptr`。再决定某个函数的具体实现时就是根据这个索引来查找的。

一个基类可以抽象成这样：
```
    class Base
     *_vptr       ---------------------->       Base VTable
     func1()      <----------------------        func1()
     func2()      <----------------------        func2()
```
假设D1继承Base，并且覆写了`func1()`，
```
    class D1
     *_vptr            ---------------------->       Base VTable
     func1()           <----------------------        func1()
     Base::func2()     <----------------------        func2()
```
D1中的VTable的`func1()`指向D1的原因是virtual table的填充原则是选择相对于这个类most-derived的函数，由于D1覆写了`func1()`，因此VTable中的`func1()`指向的是D1中的`func1()`。

main函数如下：
```c++
int main()
{
    D1 d1;
    Base *dPtr = &d1;
    dPtr->function1();
}
```
虽然`dPtr`声明的时候是Base类，但是d1中的VTable赋值给了`dPtr`，或者说`dPtr`的`*_vptr`指向的是d1中的`*_vptr`，因此在调用函数的时候根据`*_vptr`的VTable来查找，而`*_vptr`的VTable其实就是d1的VTable，结果调用的是D1中的`func1()`。

因此virtual function需要分配的更多就是因为多了virtual table。