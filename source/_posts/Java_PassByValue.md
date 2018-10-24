---
title: Pass by value or reference
date: 2018/10/24
categories: Java
---

Given a sorted linked list, delete all duplicates such that each element appear only once.
Example 1:
```
Input: 1->1->2
Output: 1->2
```
Example 2:
```
Input: 1->1->2->3->3
Output: 1->2->3
```

解题上并不难，移动的节点时候判断与后一个是否一样，如果一样就跳过。
```java
public static ListNode deleteDuplicates(ListNode head) {
    ListNode current = head;
    while (current != null && current.next != null) {
        if (current.next.val == current.val) {
            current.next = current.next.next;
        } else {
            current = current.next;
        }
    }
    return head;
}
```
不过在main函数里执行的时候，我发现获取最终结果，不需要使用该方法return的ListNode，直接将原链表作为参数传入，方法执行完之后原链表就发生了变化。
```java
//Same code
deleteDuplicates(first);
ListNode resultNode = deleteDuplicates(first);
```
这突然让我有点怀疑java究竟是pass by value还是reference。Java中最突出的特点之一就是没有指针。如果都是按值传递为什么原链表会发生变化呢。最经典的swap说明了Java的按值传递性。
```java
public static void main(String[] args) {
    int i = 10;
    int j = 20;
    swap(i, j);
    System.out.println("i :" + i);
    System.out.println("j :" + j);
}

public static void swap(int i, int j) {
    int tmp = i;
    i = j;
    j = tmp;
}
```
```
i : 10
j : 20
```
但是下面这个例子好像又表示pass by reference
```java
public static void main(String[] args) {
    Dog mDog = new Dog("Rover");
    foo(mDog);
    System.out.println(mDog.getName());
}

public static void foo(Dog someDog) {
    someDog.setName("Max1");     // AAA
    someDog = new Dog("Fifi");  // BBB
    someDog.setName("Rowlf");   // CCC
}
```
```
Max1
```
也就是说Java在传递参数的时候对基础类型和非基础类型的处理是不同的。基础类型的话是完全字面意义上的pass-by-value。而非基础类型的话，其实是创建了一个新的对象newObject（空对象），然后将这个newObject的引用指向传入的参数。如图中所示，图片来源：https://stackoverflow.com/questions/9404625/java-pass-by-reference/9404727#9404727

![](/assets/Java_PassByValue/passbyvalue1.png)

![](/assets/Java_PassByValue/passbyvalue2.png)

## 总结
在Java中，确实一切都是pass by value。但是拷贝是引用还是变量取决于源数据类型。
1. 如果是基础类型，那么参数是pass by value。
2. 如果是对象，那么对象的引用是pass by value。
3. 在方法内部修改对象的引用是不会影响原引用的。
4. 在方法内部修改collection和map类型会影响原数据。


