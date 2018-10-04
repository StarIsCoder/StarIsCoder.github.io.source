---
title: std move
date: 2018/9/14
categories: C++
---

最近在查看别人搭的框架时，看到在传值的时候使用了std::move。因此学习一下这个函数的使用。
首先google查看官方解释：

In particular, std::move produces an xvalue expression that identifies its argument t. It is exactly equivalent to a static_cast to an rvalue reference type.

简单来说就是将一个值转化为右值。目的是为了提升效率，减少拷贝的数量。也就是进行深拷贝。
## 右值
那么右值又是什么？还是查看微软的官方解释吧。

_Every C++ expression is either an lvalue or an rvalue. An lvalue refers to an object that persists beyond a single expression. You can think of an lvalue as an object that has a name. All variables, including nonmodifiable (const) variables, are lvalues. An rvalue is a temporary value that does not persist beyond the expression that uses it._

在c++中，表达式是要么左值要么右值，左值表示他对一个对象的应用远远超过了一个表达式（可以忽略），其实也可以**把左值看成一个有名字的对象**。所有的变量，常量都是左值。右值是一个临时变量，并不会比表达式保存的更久。

这个左和右是相对于等于号的。等于号左边必然是一个变量，右边是一个表示式或者是一个临时变量。感觉还是不太好用语言来解释这玩意。贴一段官方的sample。
```c++
// Correct usage: the variable i is an lvalue.  
i = 7;  
  
// Incorrect usage: The left operand must be an lvalue (C2106).  
7 = i; // C2106  
j * 4 = 7; // C2106  
  
// Correct usage: the dereferenced pointer is an lvalue.  
*p = i;   
  
const int ci = 7;  
// Incorrect usage: the variable is a non-modifiable lvalue (C3892).  
ci = 9; // C3892  
  
// Correct usage: the conditional operator returns an lvalue.  
((i < 3) ? i : j) = 7;  
```
## move
还是看回move函数，比如在写交换函数的时候我们通常这么写。
```c++
void swap(string &,string &);
int main(int argc, const char * argv[]) {
    string x = "abc";
    string y = "wang";
    swap(x, y);
    cout << x << endl;
    cout << y << endl;
    return 0;
}
void swap(string & x,string & y)
{
    string tmp = x;
    x = y;
    y = tmp;
}
```
这样的话其实在执行交换的过程中会产生三个copy的对象，会影响性能。不过有时候我们传递了某个值之后就不再不需要他了，如果留着一个拷贝的话也会占用空间，因此可以使用move函数来表示这个值以后没用了，已经传走了。
```c++
void swap(string & x,string & y)
{
    string tmp = move(x);
    x = move(y);
    y = move(tmp);
}
```
这样就不会有三分拷贝，而是值的转移。不过这么写似乎看不出值已经传走了。用个vector试一下
```c++
int main(int argc, const char * argv[]) {
    vector<string> v;
    string str = "test";
    v.push_back(str);
    cout << "str :" << str << endl;
    for (int i = 0; i<v.size(); i++) {
        cout << "vector" << i << " :" << v[i] << endl;
    }
    cout << "after move" <<endl;
    v.push_back(move(str));
    cout << "str :" << str << endl;
    for (int i = 0; i<v.size(); i++) {
        cout << "vector" << i << " :" << v[i] << endl;
    }
    return 0;
}
```
```
Result:
str :test
vector0 :test
after move
str :
vector0 :test
vector1 :test
```
可以看到str已经为空并且作为右值存在了vector中。
## 总结
并不是只在传递值的时候可以用move，更关键的是当你想把某个左值转化成右值的时候需要用到这个函数。

因为查看该函数的源码就是将传入的值转为右值引用。
```c++
move(_Tp&& __t) _NOEXCEPT
{
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}
```