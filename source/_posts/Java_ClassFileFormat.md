---
title: Class File Format
date: 2018/10/23
categories: Java
---

## 概述
Class文件是一组以8位字节为基础单位的二进制流，可以用Hex Friend等工具打开。
CLass结构： 
- 无符号数：基本类型，u1,u2,u4,u4分别代表一个字节、两个字节、四个字节、八个字节的无符号数。
- 表：多个无符号数或者其他表作为数据项构成的复合数据类型，习惯以_info结尾，整个class文件本质上就是一张表。

![](/assets/Java_ClassFileDetail/java-class-file-internal-structure.png)

## class具体结构
### 魔数
每个class文件的头4个字节称为魔数，它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件，大部分文件存储标准都是通过魔数来进行身份验证的。因为扩展名可以随意改动。对Class文件来说魔数值为**0xCAFEBABE**。

![](/assets/Java_ClassFileDetail/Class_MagicNumber.jpg)

### 版本号
在魔数之后的4个字节存储的是class文件的版本号，前两个字节是次版本号，后两个是主版本号。Java版本号是从45开始的，例如JDK1.1能支持版本号45.0 ~ 45.65535的Class文件，JDK1.2则能执行45.0 ~ 46.65535的Class文件。这边示例的class文件是用JDK11，因此应该是55。

![](/assets/Java_ClassFileDetail/Class_VersionNumber.jpg)

### 常量池
常量池：常量池可以理解为class文件中的资源仓库，它是class文件结构中与其他项目关联最多的数据类型。

由于一个类中有多少方法和变量是不确定的，因此常量池入口有一项u2类型的数据，代表常量池容量计数值。不过这个容量计数是从1开始的。也就是说如果这个值是57，那么就有56个常量，之所以把0空出来是因为如果某个常量的索引不引用任何一个常量就可以用0来表示。图中的容量计数值为59，因此一共有58个常量。

![](/assets/Java_ClassFileDetail/Class_NumberOfConstant.jpg)

常量池主要存放两大类常量：字面量和符合引用
-  字面量：Java语音的常量概念，final修饰的关键字，字符串等等
-  符号引用：
   - 类和接口的全限定名
   - 字段的名称和描述符
   - 方法的名称和描述符

常量池中的常量每一个都是一个表，并且表的结构数据不同，有的是三列，有的是两列，为了区分它们的结构，这些表的第一位是一个u1类型的标志位，代表属于哪种表的类型也就是说代表哪一种常量类型。例如如果是1那么表示是utf8类型的常量，如果是10那么就是方法的符号引用。

![](/assets/Java_ClassFileDetail/Constant_pool.png)

查看下第一个常量是0A，也就是10，10对应常量池中的项目类型是CONSTANT_Methodref_info，而这个类型一共有三个参数，第一个是tag，也就是10，第二个和第三个都是u2类型，如图中是0E（14）和1B（27），分别表示指向声明方法和指向名称的索引。

![](/assets/Java_ClassFileDetail/FirstElement.jpg)

可以使用javap反编译来确认下。第一个常量是这样的，和用hex friend计算出来是一样的。
```
 #1 = Methodref          #14.#27 
```

![](/assets/Java_ClassFileDetail/javapResult.jpg)
