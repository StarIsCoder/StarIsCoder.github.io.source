
---
title: HashMap Infinite Loop
date: 2018/12/7
categories: Java
---

最近在网上看到HashMap无限循环的情况，感觉很有意思，因此记录一下。

## Load factor
先来确认下HashMap默认的容量是16，Load factor则是0.75,Load factor的意思是当容量超过了16 * 0.75（12）的时候，该HashMap就会扩容，也就是说当存入第13个键值对的时候，Hash
map就会扩容，也就是rehashing操作。
```java
/**
* The default initial capacity - MUST be a power of two.
*/
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
* The load factor used when none specified in constructor.
*/
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

![](/assets/Java_HashMapInfiniteLoop/hashmap-rehashing-process.png)

## Rehashing
Rehashing操作中主要干了以下几件事：
 - 建立一个双倍于之前size的数组
 - 将键值对从旧数组转移到新数组
 - 键值对会反向添加因为每次在新的数组中插入键值对都是从头部插入的

![](/assets/Java_HashMapInfiniteLoop/hashmap-transfer-keyvalue.png)

## Infinite Loop
由于HashMap是非线程安全的，因此当线程1和线程2同时想插入第13个键值对的时候可能会发生无限循环的情况。

扩容这一块jdk1.8前后不太一样，在1.8中使用了两个Node来确保transfer之后顺序不会改变。
```java
//jdk 1.8之前
void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}

//jdk 1.8之后

final Node<K,V>[] resize() {
                    ......
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                    ......
}
```

旧数组中index为0的链表结构为 `90 -> 1 -> null`，假设新数组的index依旧为0。

首先线程1执行，准备插入第13个键值对，这时候需要扩容，就会调用resize方法，当执行完了第一行代码`next = e.next;`，这时候线程2开始执行。

线程2全部执行完之后，新数组中index为0的链表结构为 `1 -> 90 -> null`。

这时候1继续执行，90移到新数组，1从队头插入，新index 0变成了`1 -> 90`，然而由于线程2将1的next赋值为90，（正常情况1的next是null），然后1的next也就是90继续移动到新数组，同样的插入队头，这时的新index 0的数据结构变成了`90(由线程2设置的) -> 1 -> 90`。

那么其实变相等于90的next指向1，1的next指向90，如果这时候再来对index为0的链表进行put操作就会陷入无限循环中。
