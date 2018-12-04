---
title: Stream
date: 2018/10/04
categories: Java
---

传统的for循环其实是使用迭代器iterator来实现的，例如从列表中取出所有的偶数。
```java
for (int i : integerList) {
	if (i % 2 == 0) {
	System.out.println(i);
	}
}
```
用iterator来写就是，用文字来描述就是从列表中取出一个进行判断是否要输出，这是种串行化操作，叫做外部迭代。给人感觉就是都混在了一起。 
```java
Iterator<Integer> iterator = integerList.iterator();
while (iterator.hasNext()) {
	int i = iterator.next();
	if (i % 2 == 0) {
		System.out.println(i);
    }
}
```
而使用的stream是内部迭代。这时整个逻辑被分割开来，filter只负责过滤，foreach负责每个item如何处理。
```java
integerList.stream()
		.filter(integer -> integer % 2 == 0)
		.forEach(integer -> System.out.println(integer));
```
- 惰性求值：只是用来描述stream，比如filter。

- 及早求值：希望得到返回结果，例如foreach或者count。



**判断惰性和及早只需要看返回值，如果返回值是stream，那么就是惰性，否则则是及早。**

大致知道一些常用的，例如map（映射成别的值），filter（过滤），collect（生成新的列表）等等。尽量以后在敲代码的时候多用Lamda表达式，一是为了练习，二是Lamda表达式确实看起来舒服很多。

## 使用stream排序
使用stream方法中的sorted方法可以给一个stream排序，也可以自定义，默认升序。
```java
list.stream().sorted() 

list.stream().sorted(Comparator.reverseOrder()) 

list.stream().sorted(Comparator.comparing(Student::getAge)) 

list.stream().sorted(Comparator.comparing(Student::getAge).reversed()) 

//最后需要将排序完的结果转化成list
list.stream().sorted().collect(Collectors.toList());
```
