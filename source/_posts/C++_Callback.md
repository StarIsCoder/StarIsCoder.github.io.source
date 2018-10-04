---
title: Basic Callback
date: 2018/9/21
categories: C++
---
项目中需要提供api和一个被动接受的回调，C++的回调方式主要有三种：
* 传递函数指针
* 实现接口
* lamda表达式

### 实现接口
新建一个接口类，ITokenGetCallback.hpp，添加需要其他类实现的虚函数。
```c++
#ifndef ITokenGetCallback_h
#define ITokenGetCallback_h

class ITokenGetCallback {
public:
    virtual ~ITokenGetCallback() {}
    virtual void onGetToken(const unsigned char *) = 0;
};
#endif /* ITokenGetCallback_h */
```
_BTW:一般在构造接口的时候需要把析构函数也虚化，原因是为了确保对象在被回收之后析构函数能被调用。这就涉及到虚函数的具体实现意义了。_

贴一个stackoverflow上的例子。
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
```c++
Animal *animal = new Animal;
Animal *cat = new Cat;
animal->eat(); // outputs: "I'm eating generic food." 
cat->eat(); // outputs: "I'm eating generic food."
```
即使我们给他赋值子类，但是函数的调用还是基于指针的类型，这里可以类比到析构函数。

添加一个实现该接口的类
```c++
void TokenGet::onGetToken(const unsigned char * token){
    std::cout << "on get token" << token << std::endl;
}
```
剩余要做的事情就是把这个类传递到想调用的地方。可以使用set或者作为构造函数的参数传入。
```c++
//通过函数参数传递
TokenGet* callback = new TokenGet();
gClient->StartRecording(callback);

auto callback = std::shared_ptr<TokenGet>(new TokenGet());
gClient->setCallback(callback);
```
之后在想调用的地方调用这个call即可。接口的方式属于OOP的一种模式吧，这方面和java很类似。

### lamda表达式
这种方式一般用在次数比较少的回调。
用std的function包装一个方法并设置返回值和参数。
```c++
using OnVolumeCallback = std::function<void(int volume)>;
```
然后将该函数作为参数放在你想使用的函数中。
```c++
void APIClient::getSpeakerVolume(OnVolumeCallback callback)
{
    //do something
    callback(volume);
}
```
在调用这个函数的地方实现这个lamda表达式
```c++
gClient->getSpeakerVolume([](int volume){
        std::cout << "volume: " << volume << std::endl;
    });
```
相当来说lamda表达式更简单一点。不过如果是多次回调还是应该用接口的方式，接口的方式在层次上也更分明。

lamda表达式是这样定义的
```
[ capture clause ] (parameters) -> return-type  
{   
   definition of method   
} 
```

其中parameters和return-type可以省略。关于这两者和函数具体实现没什么好多说的，第一次知道还有捕获列表这样的东西。


捕获列表就是用[]包起来的列表，可以传引用、指针、变量的拷贝，不过无法传递右值。通俗来说就是被这个捕获列表包起来的参数可以在lamda表达式中使用。
```c++
int j = 5;
auto f = []{
    j = 1; //invalid,Variable 'j' cannot be implicitly captured in a lambda with no capture-default specified
};
```
如果不在捕获列表中添加j的话是无法访问的。可以理解为用非传统的方式把变量作为参数传递进入函数。