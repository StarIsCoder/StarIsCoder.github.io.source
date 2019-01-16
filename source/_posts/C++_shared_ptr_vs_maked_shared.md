---
title: make_shared vs shared_ptr
date: 2019/1/16
categories: C++
---

最近在使用shared_from_this的时候发生了crash，具体原因是throw_bad_weak_ptr，查了Stack Overflow，root cause是因为想在object A上使用shared_from_this,但是却没有一个shared_ptr指向他，这违反了使用shared_from_this的前提条件：至少有一个shared_ptr被创建并指向A。

查了下代码发生确实在实例化的时候是用new出来的裸指针
```c++
ObjectA m_ObjectA = new ObjectA();
```
应该变成
```c++
shared_ptr<ObjectA> m_ObjectA = make_shared<ObjectA>();
```

但是shared_ptr又有两种创建方式
```c++
std::shared_ptr<Object> p1 = std::make_shared<Object>("foo");
std::shared_ptr<Object> p2(new Object("foo"));
```
最主要的差别在于make_shared执行一次堆分配(将shared_ptr执行的两次堆分配合在一起)，而shared_ptr会执行两次堆分配。由于make_shared只执行一次堆分配因此效率更高。但make_shared只有一次堆分配，因此只有当两者都没有被引用时才会释放。
- Shared_ptr：
   1. 引用计数
   2. 对象数据
- Make_shared：
   1. 引用计数和对象数据

