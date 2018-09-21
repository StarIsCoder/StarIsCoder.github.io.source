---
title: Java_Concurrent_AQS
date: 2018/09/17
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
