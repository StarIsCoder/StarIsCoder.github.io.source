---
title: 函数与数组
date: 2018/8/17
categories: C++
---
## 单个数组作为函数的参数
C++中将数组名看成是第一个元素的地址。如下：
```c++
int cookies[8] = {1,2,4,8,16,32,64,128};
cout << cookies;
```
```
0x7ffeefbff540
```
但是该规矩有一个特例，就是在函数原型中，int *arr和int arr[]表示同一个意思。如下：
```c++
int sum_arr(int *arr,int n);
```
```c++
int sum_arr(int arr[],int n);
```
也就是说在这个特例中将数组作为了参数，其实是将数组第一个元素的地址传递给了函数，而并没有传递数组的内容。

**总结：传递常规变量时，函数将使用该变量的拷贝；但传递数组时，函数将使用原来的数组。其实传递数组时也传递了一个值，这个值被赋予给了一个新变量，也即是地址。
这么做的好处是能节省开支，如果该数组很大，那么要对它进行拷贝则费时费力。**

_Warning :为将数组类型和元素数量告诉数组处理函数，请通过两个不同的参数来传递它们：_
```c++
void fillArray(int arr[], int size); //good
void fillArray(int arr[size]); //bad
```

如果在某个函数中，所传入的数组不希望被修改，则可以使用const关键字修饰该数组。这表示指针arr指向的是常量数据，在该函数中不能使用ar修改该数据，当然，这并不是说原始数组必须是常量。如下：
```c++
void show_array(const int arr[], int n);
```

## 多个数组作为函数的参数
实际上我们也可以通过传入两个地址来表示数组在这两个地址的区间内。
```c++
using namespace std;
int sum_arr(int *start,int *end);
int main(int argc, const char * argv[]) {
    int cookies[8] = {1,2,4,8,16,32,64,128};
    int total = sum_arr(cookies, cookies + 8);
    int wrongTotal = sum_arr(cookies, cookies + 9);
    cout << total;
    cout << endl;
    cout << wrongTotal;
    
}
int sum_arr(int start[], int end[]){
    int sum = 0;
    int *pt;
    for (pt = start; pt != end; pt++) {
        sum = sum + *pt;
    }
    return sum;
}
```
```
Result:
255
-272632185
```
_即使超出数组也不会报错，因为c++中只在初始化的时候检查数组是否越界。_

## 函数和字符串
将字符串作为参数来传递的时候，同样传递的是第一个字符的地址，字符串有内置的结束字符。因此想要判断字符串是否读取完毕则可以使用*str是否为0。如下：
```c++
using namespace std;
int count(char * str);
int main(int argc, const char * argv[]) {
    char * str = "xingxing";
    int g = count(str);
    cout << g;
}
int count(char * str)
{
    int count = 0;
    while (*str) {
        if (*str == 'g') {
            count++;
        }
        str++;
    }
    return  count;
}
```

## 函数和结构体
函数中使用结构体比较简单，将结构体当作一个对象来使用就可以了。不赘述。

## 函数指针
C++中函数也有地址，函数的地址是存储其机器语言代码的内存的开始地址，有时候会将函数地址作为参数传给另一个函数。
### 获取函数地址
使用函数名且不带任何参数就是使用函数的地址。例如test()是一个函数，那么test就是该函数的地址。
```c++
show(test()) //传递test函数的返回值
show(test)   //传递test函数的地址
```
### 声明函数指针
声明一个函数指针就是将该函数原型中的函数名替换为指针名即可。
```c++
int test(int); //原型
int (*pf)(int); //函数指针声明
pf = test; //函数指针赋值
```
如果使用该函数指针来调用函数的话则可以这样：
```c++
pf(5);
(*pf)(5);
```

## C++PrimerPlus习题
### 7.6
```c++
void Fill_array(double [], int size);
void Show_array(double [], int size);
void Reverse_array(double [], int size);

using namespace std;

int main(int argc, const char * argv[]) {
    int max = 5;
    double array[max];
    Fill_array(array, max);
    Show_array(array,max);
    Reverse_array(array, max);
    Show_array(array, max);
    return 0;
}
void Fill_array(double arr[], int size)
{
    for (int i = 0; i < size; i++) {
        cout << "Enter double :";
        cin >> arr[i];
    }
}

void Show_array(double arr[], int size)
{
    for (int i = 0; i < size; i++) {
        cout << arr[i];
        cout << endl;
    }
}

void Reverse_array(double arr[], int size)
{
    for (int i = 0; i < size / 2 ; i++) {
        double tmp = arr[i];
        arr[i] = arr[size - i -1];
        arr[size - i -1] = tmp;
    }
}
```
### 7.10
```c++
double calculate(double,double,double (*pf)(double,double));
double add(double,double);
int main(int argc, const char * argv[]) {
    double d = calculate(2.2,2.8,add);
    cout << d << endl;
    return 0;
}

double add(double x,double y)
{
    return x + y;
}

double calculate(double i,double j,double (*pf)(double,double))
{
    return pf(i,j);
}
```


_基于C++ Primer plus第七章。_