---
title: Constructor&Destructor
date: 2018/9/4
categories: C++
---

## 构造函数

构造函数的定义就不写了，直接看下例子,这里的构造函数中初始化了number这个变量。

_插播一段：这里使用了ifndef和define来防止多重包含。即使多个文件include了这个头文件也只会包含一次。_
```c++
#ifndef Stock00_h
#define Stock00_h

class Stock {
private:
    int number_;
public:
    void display();
    void set(int);
    Stock(int);
};

void Stock::display()
{
    std::cout << number_ << std::endl;
}
Stock::Stock(int number)
{
    number_ = number;
}
#endif /* Stock00_h */
```
如何使用：
```c++
int main(int argc, const char * argv[]) {
    Stock stock1(100);                //隐式
    Stock stock2 = Stock(100);        //显式
    Stock stock3 {100};               //类似初始化列表的情况
    stock1.display();
    return 0;
}
```
Warning:
* 如果该类定义了构造函数，那么当实例化该类的时候则必须调用它的构造方法
* 构造函数的参数名不能与类成员相同。一种做法是在数据成员前面加上m_前缀，还有一种做法是在成员名称后面加后缀_。

## 析构函数
析构函数是和构造函数对应的，一个是创建过程中调用，析构函数则是在对象销毁时调用的，不过大部分情况都是由系统自己调用，很少有显式调用的情况。析构函数不包含任何参数。
* 静态存储类对象：程序结束时调用。
* 自动存储类对象：函数结束时调用。
* new创建的对象：调用delete来释放内存时调用。

## const成员函数
如果定义了一个const的类，调用该类的非const方法则会报错，因为不能保证调用的对象不被修改：
```c++
const Stock stock1 {100};
stock1.display();

void Stock::display()
{
    std::cout << number_ << std::endl;
}
```
解决方式在函数最后加上const关键字:
```c++
void display() const;

void Stock::display() const
{
    std::cout << number_ << std::endl;
}
```

## this指针
之前在尝试写成员变量的get/set方法时，发现在c++中this并不代表一个类，而是一个类的指针。并且可以表示隐式的访问，
```c++
stock1.topval(stock2);

Stock & Stock::topval(Stock &s)
{
    if (this->number_ > s.number_) {
        return *this;
    } else {
        return s;
    }
}
```
可以看到在这个函数中，this表示stock1。

基于C++ Primer plus第十章