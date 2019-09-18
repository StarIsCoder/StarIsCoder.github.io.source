---
title: const pointer
date: 2019/9/18
categories: C++
---

### cont int *
const int * 表示的是一个指针 - 指向一个int常量，因此这个指针所指向的东西是只读的。但指针本身是可写的。

```c++
int i = 100;
const int* pt = &i;// 表示指向一个int常量的指针
*pt = 101; // error: Read-only variable is not assignable
int j = 101;
pt = &j; // good
```

### int* const pt
int * const 表示的是一个常量指针 - 指向int类型，因此这个指针是只读的，但是他所指向的东西是可以改变的。
```c++
int i = 100;
int j = 101;
cout << &i << endl;
int* const pt = &i;// 表示一个指向int类型的常量指针
pt = &j; // Cannot assign to variable 'pt' with const-qualified type 'int *const'
*pt = 1000; // good
```


### Summary
关于c或者c++这种指针类型的解释是有规律的，只需要倒过来读就可以了，`*`表示a pointer to(一个指针 - 指向) 
- `int*` == pointer to int == 一个指针 - 指向int类型
- `int const *` == pointer to const int == 一个指针 - 指向int常量
- `int * const` == const pointer to int == 一个常量指针 - 指向int类型
- `int const * const` == const pointer to const int == 一个常量指针 - 指向int常量

当`const`处于首位的时候，放在类型的两边的都可以
- `const int *` == `int const *`
- `const int * const` == `int const * const`

Clockwise/Spiral Rule：

http://c-faq.com/decl/spiral.anderson.html