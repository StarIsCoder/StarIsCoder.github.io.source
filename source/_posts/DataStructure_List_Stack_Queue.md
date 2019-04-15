---
title: List/Stack/Queue
date: 2019/04/15
categories: DataStructure
---

### 表
#### 数组实现
数组的get操作花费常数时间，不过插入和删除却有可能有巨大开销。因为最坏的情况下，在位置0的插入将会把之后所有的元素后移一位。而删除第一个元素将把之后所有的元素前移一位。相反如果删除和插入都在表的高端，开销则没有那么大。

Java中用数组实现的列表为ArrayList，对get和set的调用花费常数时间，但插入和删除开销较大，除非是在末端进行。

以下为自己用数组实现的List，用`mCount`表示list中的元素个数。
```java
//扩展数组
public void expandArray() {
    T old[] = mItems;
    mItems = (T[]) new Object[mItems.length * 2];
    for (int i = 0; i < size(); i++) {
        mItems[i] = old[i];
    }
}

public void add(int index, T value) {
    if (size() == mItems.length) {
        expandArray();
    }
    for (int i = size(); i > index; i--) {
        mItems[i] = mItems[i - 1];
    }
    mItems[index] = value;
    mCount++;
}

public T remove(int index) {
    T needToRemove = mItems[index];
    for (int i = index; i < size(); i++) {
        mItems[i] = mItems[i + 1];
    }
    mCount--;
    return needToRemove;
}

public T get(int index) {
    return mItems[index];
}

public void clear() {
    mCount = 0;
    mItems = (T[]) new Object[DEFAULT_CAPACITY];
}

//将Iterator作为内部类实现
class MyArrayListIterator<T> implements Iterator<T> {
    private int current = 0;

    @Override
    public boolean hasNext() {
        return current < size();
    }

    @Override
    public void remove() {
        MyArrayList.this.remove(--current);
    }

    @Override
    public T next() {
        return (T) mItems[current++];
    }
}
```

<https://github.com/StarIsCoder/DataStructure-BasicAlgorithm/blob/master/src/base/datastructure/MyArrayList.java>

#### 链表实现

使用链表查找开销较大，因为需要遍历整个list，但是对于删除和插入只需要操作某一个节点的`next`即可（需要知道该节点的位置），由于对第一项和最后一项的操作属于特殊操作，因此一般都会新增头节点和尾节点（也就是说一个空的list有两个节点），在Java的LinkedList中有`addFirst`、`addLast`等api。

以下为简单实现，用`mSize`表示元素个数:
```java
public class MyLinkedList<T> implements Iterable<T> {
    private Node<T> head;
    private Node<T> tail;
    private int mSize;

    public MyLinkedList() {
        init();
    }

    private void init() {
        head = new Node<>(null, null, null);
        tail = new Node<>(null, head, null);
        head.next = tail;
        mSize = 0;
    }

    public int size() {
        return mSize;
    }

    public void add(T data) {
        add(size(), data);
    }

    public void add(int index, T data) {
        addBefore(getNode(index), data);
    }

    public void addBefore(Node<T> p, T data) {
        Node<T> newNode = new Node<>(data, p.prev, p);
        newNode.prev.next = newNode;
        p.prev = newNode;
        mSize++;
    }

    public void remove(int index) {
        Node<T> needToRemove = getNode(index);
        needToRemove.prev.next = needToRemove.next;
        needToRemove.next.prev = needToRemove.prev;
        mSize--;
    }

    private Node<T> getNode(int index) {
        Node<T> p;
        if ((size() / 2) > index) {
            p = tail;
            for (int i = size(); i > index; i--) {
                p = p.prev;
            }
        } else {
            p = head.next;
            for (int i = 0; i < index; i++) {
                p = p.next;
            }
        }
        return p;
    }

    @Override
    public Iterator<T> iterator() {
        return new MyLinkedListIterator();
    }

    class MyLinkedListIterator implements Iterator<T> {
        private Node<T> current = head.next;
        private int count = 0;

        @Override
        public boolean hasNext() {
            return current != tail;
        }

        @Override
        public T next() {
            T nextItem = current.data;
            current = current.next;
            return nextItem;
        }
    }

    private static class Node<T> {
        public Node(T data, Node<T> prev, Node<T> next) {
            this.data = data;
            this.prev = prev;
            this.next = next;
        }

        public T data;
        public Node<T> prev;
        public Node<T> next;
    }
}

```

**对LinkedList来说不使用增强for循环那么搜索操作花费的时间为O（N^2），因为get需要O（N），但使用增强for循环的则是O（N），因此对于搜索操作来说，不管是Array还是Link都是O(N)，都是低效的操作。**



<https://github.com/StarIsCoder/DataStructure-BasicAlgorithm/blob/master/src/base/datastructure/MyLinkedList.java>

### 栈

栈是限制插入和删除只能在一个位置上进行的表，特点是LIFO，一般只有push和pop操作，至于栈的实现用数组和链表都可以实现:

```java
//数组实现
 public T pop() {
    if (count > 0) {
        return items[count--];
    }
    return null;
}

public void push(T data) {
    items[++count] = data;
}

//链表实现，bottom是栈底的元素
public void push(V data) {
    Node<V> newNode = new Node<>(data);
    Node<V> temp = bottom;
    while (temp.next != null) {
        temp = temp.next;
    }
    temp.next = newNode;
}

public V pop() {
    Node<V> temp = bottom;
    if (temp.next == null) return null;
    while (temp.next.next != null) {
        temp = temp.next;
    }
    V v = temp.next.value;
    temp.next = null;
    return v;
}
```

栈的应用，例如括号匹配，JVM中的栈帧，现代计算机的指令。

<https://github.com/StarIsCoder/DataStructure-BasicAlgorithm/blob/master/src/base/datastructure/MyArrayStack.java>

<https://github.com/StarIsCoder/DataStructure-BasicAlgorithm/blob/master/src/base/datastructure/MyLinkedStack.java>

### 队列
队列和栈有点类似，不过是FIFO，主要操作有enqueue和dequeue，也可以用数组或者链表来实现，以下为数组实现：
```java
//用了first和last两个指针来表示头和尾
public void enqueue(T data) {
    if (last == mItems.length - 1) {
        last = -1;
    }
    mItems[++last] = data;
}

public T dequeue() {
    T temp = mItems[first++];
    if (first == mItems.length)
    	//绕回到开头
        first = 0;
        return temp;
}
```
队列有个问题，那就是下面这种情况，看上去都满了但其实有一些是空着的，最简单的办法就是绕回到开头，当然队列也有需要扩充的情况。
```
[- - - - - - - 9 7 6]
               f   e
```
队列的应用比如有Android中的MessageQueue等等。

<https://github.com/StarIsCoder/DataStructure-BasicAlgorithm/blob/master/src/base/datastructure/MyArrayQueue.java>