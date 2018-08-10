---
title: C++ Pointer Note
date: 2018/8/7
categories: C++
---

## 指针
### 基本概念
指针也是一种变量，特殊之处在于它存储的是变量的地址，如下：
```c++
int main(int argc, const char * argv[]) {
    int i = 6;
    //assign the address of i to pointer
    int *pointer = &i;
    int *pointer1 = new int;
    return 0;
}
```
操作符*被叫做间接值或者解除应用

_**Warning: 一定要在对指针使用*之前，将指针初始化为一个确定的、适当的地址**_

### 运算符new 
在c中，使用malloc来分配内存，在c++中则使用new关键字。
为一个数据对象获得并制定分配内存的通用格式：
_typeName * pointer_name = new typeName;_

一旦我们不需要该指针，应该使用delete关键字将该指针释放，new和delete必须成对出现，否则会发生memory leak。
```c++
int *ps = new int;
...
delete ps;
```
这将释放ps指针所指向的内存，但不会删除ps本身，也就是说可以继续将ps指向其他地址。

### 动态数组
使用new关键字来创建动态数组，并且返回的是该数组第一个元素的地址，如果想要获得第二个元素的地址，则需要将该指针加1，如下：
```c++
int main(int argc, const char * argv[]) {
    // insert code here...
    int *array = new int[10];
    array[0] = 1;
    array[1] = 3;
    cout << *array;
    cout << endl;
    array = array + 1;
    cout << *array;
    cout << endl;
    return 0;
}
```
Result:
```
1
3
```
### 动态结构
结构体同样可以使用new关键字。不同的是无法使用句点运算符，取而代之的是->运算符，如下：
```c++
struct things {
    int price;
    int sum;
};
int main(int argc, const char * argv[]) {
    // insert code here...
    things t = {3,18};
    things * tpt = &t;
    //things * tpt = new things;
    cout << tpt->price;
    cout << endl;
    return 0;
}
```
Result:
```
3
```
### 指针与字符串
一般来说如果给cout提供一个指针的话，他将打印地址，但是如果指针的类型为char *，那么cout的指针其实指向的是char*的第一个字符的地址，并且会依次往后直到结束符。如下：
```c++
int main(int argc, const char * argv[]) {
    char * c = "dog";
    cout << "c :" << c;
    cout<< endl;
    cout <<"*c :" << *c;
    cout << endl;
    cout <<"第一个字符的地址 ：" << (int*)c;
    cout << endl;
    return 0;
}
```
```
c :dog
*c :d
第一个字符的地址 ：0x100001edc
```
