---
title: Java_NIO
date: 2018/9/15
categories: Java
---

## NIO
NIO有说是new io，也有说是non-blocking io，Whatever，nio是jdk 1.4之后引入的，目的是为了提高io速度，并且旧的io也已经被重新实现过了。记得以前还在odm的时候，经常会碰到ANR的原因是系统在执行io操作导致的，因此看来优化一下io还是有必要的。
## Channel和Buffer
之所以将channel和buffer放在一起是因为这两个是提高io速度的关键。旧的io形式是面向流的，例如经常用的InputStream和OutputSteam，比如在我们读取文件的时候，每次从流中读取一个字节，说的不恰当一点就是在文件和程序之间建立一个通道，并且是一个单向的通道，每一次把文件中的东西拷贝到程序中。

最致命的就是Stream的操作是阻塞的，在调用read方法的时候我们什么都做不了，只能等待它执行完了之后返回。

因此channel和buffer诞生了，这边结合Thinking in java中对channel和buffer的描述加以理解，把channel当成矿场，而把buffer当成矿场中的坑，那么我们在读取的时候其实是先把矿场里的煤矿丢进坑里，然后去拿坑里的煤矿，写数据的时候反之即可。这样形成的读写模式是非阻塞的。

```java
public static void main(String[] args) {
    try {
        //把stream转化成矿场（channel）
        FileChannel channel = new FileOutputStream("data.txt").getChannel();
        //把我们要加入的煤矿丢进坑（buffer），之后再把坑中的煤矿丢尽矿场
        channel.write(ByteBuffer.wrap("First".getBytes()));
        channel.close();

        //same as above
        channel = new RandomAccessFile("data.txt", "rw").getChannel();
        //把矿车移到最后一个坑，然后继续挖坑填矿
        channel.position(channel.size());
        channel.write(ByteBuffer.wrap("Second".getBytes()));
        channel.close();

        channel = new FileInputStream("data.txt").getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        channel.read(buffer);	
        buffer.flip();
        while (buffer.hasRemaining()) {
            System.out.print((char) buffer.get());
        }
    } catch (Exception e) {
        e.printStackTrace()
    }
}
```
再来看下buffer，这里用的字节buffer，当然也有FloatBuffer、 DoubleBuffer等等。
刚才说过了buffer类似于矿场中的坑，但是我们不可能把整个矿场都挖出坑来，因此我们在使用buffer的时候需要分配一个大概的数量，也就是allocate方法。通过源码在allocate方法中它其实是返回了一个HeapByteBuffer，这是个继承ByteBuffer的类。
```java
public static ByteBuffer allocate(int var0) {
    if (var0 < 0) {
        throw new IllegalArgumentException();
    } else {
        return new HeapByteBuffer(var0, var0);
    }
}
```
通过一系列的调用最后它给三个属性赋值，分别是capacity、limit、position
```java
Buffer(int var1, int var2, int var3, int var4) {
    if (var4 < 0) {
        throw new IllegalArgumentException("Negative capacity: " + var4);
    } else {
        this.capacity = var4;
        this.limit(var3);
        this.position(var2);
        if (var1 >= 0) {
        if (var1 > var2) {
            throw new IllegalArgumentException("mark > position: (" + var1 + " > " + var2 + ")");
        }

        this.mark = var1;
        }

    }
}
```

* capacity：一共有多少个坑

* limit：最多有多少个坑

* position：矿车在第几个坑

显然在刚开始分配的时候，capacity和limit是一样的，position是零。
当我们执行了填写数据的代码之后，矿车肯定是在数据字节大小的位置上。例如我们要读的文件存着test，那么矿车这时候是在第四个位置上。
**BTW：这里从1开始计数是因为，在初始分配的时候就已经默认生成了一个坑，零号位的坑。**
那么这个时候如果我们想要从坑中拿煤矿的话，至少要将矿车移动到最前面。也就是flip方法的作用。看一下源码就知道它到底干了啥。

```java
public final Buffer flip() {
	//把limit减少，提升效率
    this.limit = this.position;
    //将矿车移到开头
    this.position = 0;
    this.mark = -1;
    return this;
}
```

buffer大部分的api都是对这三个属性操作的。例如clear方法。

```java
public final Buffer clear() {
    this.position = 0;
    this.limit = this.capacity;
    this.mark = -1;
    return this;
}
```
## 总结
旧的io和nio主要有以下几点不同
* io面向流，nio面向buffer
* io是阻塞，nio非阻塞
* nio有selector，可以理解为管理多个矿场。
关于selector还处于一知半解的状态，希望以后在项目中能碰到。
