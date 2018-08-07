---
title: C++ Pointer Note
date: 2018-08-07 17:45:00
categories: C++
---

# Pointer
## Basic Concept
Pointer is also a variable. But it stores the address of some value, not the value itself. Here is demo:
```c++
int main(int argc, const char * argv[]) {
    int i = 6;
    //assign the address of i to pointer
    int *pointer = &i;
    int *pointer1 = new int;
    return 0;
}
```
The operator * is called indirect value or dereferencing.

_**Warning: Before using operator * to the pointer, The pointer must be initialized to a certain and approiate address.**_

## Operator new 
In C, we could use malloc to allocate memory.But in c++, we use operator new.

There ara some general format to allocate memory by using operator new.

_typeName * pointer_name = new typeName;_

Once we don't need this pointer,we should delete it or will cause memory leak. **delete** and **new** should be paired.
```c++
int *ps = new int;
...
delete ps;
```
It only release the memory pointed by `ps`, won't delete `ps` it self. So you still can allocate another memeroy to `ps`. 

## Dynamic Array
Still use operator new to create dynamic array, and it returns an address of the first element. if we want to get an address of the second element,the pointer array shoule plus one.Here is the demo:
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
Program ended with exit code: 0
```
