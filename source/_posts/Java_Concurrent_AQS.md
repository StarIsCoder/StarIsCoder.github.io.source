---
title: Concurrent AQS
date: 2018/09/26
categories: Java
---

除了使用synchronized关键字，还可以显式使用lock对象来解决并发问题。
## ReentrantLock
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    //add code you want to lock
    count++;    
} finally {
    lock.unlock();
}
```
ReentrantLock是一个能和synchronized实现同样功能的对象，只是在实现方面更为灵活一点，比如尝试获取锁，过一段时间放弃它。因此被lock住的线程是线程安全的。

ReentrantLock分为公平模式和非公平模式,非公平模式其实就是在lock的时候多了个操作，就是尝试下能否直接获取，拿不到就乖乖排队。
```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
    ......
}
/**
* Sync object for fair locks
*/
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }
    ......
}
```

**一定要使用try和finally的组合，能确保锁被释放，以防引起阻塞。**

## ReentrantReadWriteLock
这个lock是以read和write成对的锁。
```java
ReadWriteLock lock = new ReentrantReadWriteLock();

lock.readLock().lock();
//add code to read
lock.readLock().unlock();

lock.writeLock().lock();
//add code to write
lock.writeLock().unlock();
```
该对象在实例化的时候可以传个boolean值表示选择公平模式还是非公平模式。
```java
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```
* 公平模式：先到先得，晚来的排队，不管是读锁还是写锁都没有特权。
* 非公平模式：不阻塞write锁。write锁优先级高
默认采用的是非公平模式。

write锁是独占模式，互相干扰，而read锁则是共享模式，read锁之间互不干扰。

具体原理需要先去理解下AQS的框架才行。因为ReentrantLock中几乎所有的方法都是调用被sync对象封装起来的方法，而这个sync对象是AbstractQueuedSynchronizer（AQS）的子类。也就是说大体逻辑都在AQS中得以实现，ReentrantLock和ReentrantReadWriteLock进行了不同的定制而已。

## AQS
### 功能

既然是sychronizer，那必须有一个flag来代表lock或者unlock。在AQS中使用state来维护。
```java
/**
 * The synchronization state.
 */
private volatile int state;
```
有了state，必须要有state对应的get/set方法。在AQS中是_acquire_和_release_。不过被包装在了子类中名称不太一样，比如Lock.lock,Semaphore.acquire,CountDownLatch.await.
```java
//ReentrantLock.java
final void lock() {
    acquire(1);
 }

//AbstractQueuedSynchronizer.java 
public final void acquire(int arg) {
    //tryAcquire的实现在ReentrantLock中
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
每个synchronizer必须支持以下三个特性：
*  阻塞的synchronization和非阻塞的synchronization
*  超时选项
*  可以通过interruption来打断

将state分为两种情况：
* exclusive mode：独占模式，在同一个时间只有一个线程可以访问(acquire和release)
* share mode：共享模式，有多个线程可以同时访问(acquireShared和releaseShared)

### 设计思想
同步的问题主要围绕对锁的逻辑处理，也就是acquire和release。

获取锁的操作：

_while (state不允许acquire) {_

_将该thread排队_

_}_

_如果在队列中就从队列中清除_

释放锁的操作：

_更新state_

_if(允许其他被阻塞的线程acquire) {_

_释放在队列中的一个或多个线程_

_}_

因此它必须有以下三个模块
* 原子性操作state
* 阻塞和释放线程
* 维护队列

### State
在AQS中的state是一个32位int类型的变量，0表示未被持有，1表示已经被持有。并且扩展出了get/set和compareAndSet,而它们之所以能完成原子性操作主要是基于volatile关键字。

compareAndSet这个方法会传入两个参数，expect和update,如果current state和expect相等，则会直接更新update。

而AQS的实现类必须实现tryAcquire和tryRelease方法，返回为true则表示持有锁成功。
```java
//AbstractQueuedSynchronizer.java 
//默认直接抛出异常，需要子类重写
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}

protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```
### Block
在AQS中使用LockSupport的park和unpark方法来真正实现lock功能。park方法和unpark作为成对出现，并且park方法可以传入timeout的参数，用来实现timeout的同步锁。
```java
 LockSupport.parkNanos(this, nanosTimeout);
```
这里LockSupport只是作为工具类，具体的逻辑还是要看queue中是怎么处理的。

### Queue
通过源码查看到内部维护了一个Node的内部类，并且该Node既有prev又有next的成员变量，因此这是一个双向的链表。以acquire为例
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

......

private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```
- tryAcquire->先尝试获取状态，由子类自己实现 
- addWaiter->生成一个节点，并将节点放入队列中
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
- 使用自旋锁，并且如果前面一个是头节点了则不断去尝试获取状态。也就是说下一个就到这个线程，那么他就会一直去尝试获取，确保能最快获取到状态
- shouldParkAfterFailedAcquire(p, node)->如果在该线程还有其他线程在等待，那么就判断是否需要挂起操作
```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        return true;
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
           compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
- 只有当节点的线程状态为SIGNAL的时候，才需要进行挂起操作
- ws>0表示前一个节点是canceled状态，因此就持续往前找，直到找个一个有效的节点，并将新生成的节点放在其后
- 如果不是canceled状态，那么需要将其切为SIGNAL状态
- 如果需要挂起那么就使用LockSupport.park将其挂起，等待unpark
结合release来看
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
       return true;
    }
    return false;
}
```
- tryRelease->同样的想用子类的去释放状态
- 如果成功释放了，那么唤醒队列中的头节点
```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
- LockSupport.unpark(s.thread) -> 唤醒头节点的后续节点
- 由于已经unpark了所以acquireQueued会继续执行，当、一旦头节点释放资源，后续节点则可以tryAcquire

原则上来说该队列是FIFO原则，先来先到。

比如在银行拿号，如果柜台有人了，那么你得排队
- 前面就轮到你了，那么你尝试排个队，如果失败，那么你认定前面还在处理业务，就标记为SIGNAl，过了一会儿又看了下，o！是SIGNAL，那么我还是先休息下，也就是park操作。
- 后面又来一个人，看到你在排队但不是SIGNAL，那么把你置为SIGNAL，然后再试一次，失败之后发现是SIGNAL，那么他就直接休息（park）了。
- 如果看到前面的人不排了，也就是cancel状态，那么就一直往前找，直到找到一个确实在排队的人，排在他后面。如果前面一个人状态不太对但是也确实在排，那会把他标记成SIGNAL。


具体还有很多细节问题没有深入，暂时只是知道一个大概。毕竟AQS是java锁机制的一个基础，还是需要多看几遍才行。