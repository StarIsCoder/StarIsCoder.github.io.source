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

### 访问标志
常量池之后是access_flags，这个标志用于识别一些类或者接口层次的访问信息，例如这个class是类还是接口，是public还是abstract等等。使用javap也可以看到标志。
```
flags: (0x0021) ACC_PUBLIC, ACC_SUPER
```

### 类索引、父类索引、接口索引集合
类索引和父类索引都是一个u2类型的数据，而接口索引集合是一组u2类型的数据。Class文件中由这三项来确定继承关系。

类索引和父类索引各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过这个类描述符常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的全限定名字符串。

例如图中this_class的索引是13，而13的索引又指向了55，55则代表这个类名。

![](/assets/Java_ClassFileDetail/this_class.jpg)

![](/assets/Java_ClassFileDetail/this_class2.jpg)

### 字段表集合
field_info用于描述接口或者类中声明的变量。字段包括类级变量以及实例级变量，不包括方法中的临时变量。

字段表的结构用access_flags来表示作用域、是否final、是否static等等，另外用了两个索引来表示这个变量的名称和类型（有映射的字符，例如int对应I）。另外还有两个字段存储额外的信息，如果我们给一个变量赋了初始值，那么这两个字段就会有对应的值。

### 方法表集合
类似于字段表集合，由于方法没有volatile和transient关键字，因此access_flags中没有ACC_VOLATILE标志和ACC_TRANSIENT标志。而增加了synchronized、native等修饰方法的关键字。由于和字段表集合大同小异，不多赘述。

但是需要注意的是方法中的代码被单独存放在了方法表中的code字段下，可以看到下图是用javap反编译出的main方法。access_flags是ACC_PUBLIC和ACC_STATIC，descriptor表示传入的参数和返回值。code字段下是方法中的代码。

![](/assets/Java_ClassFileDetail/javap_method.jpg)


