# 同步框架AbstractQueuedSynchronizer

## 同步框架AbstractQueuedSynchronized（简称AQS）

提供了一个通用的多线程同步管理的基础框架，在此框架之上可以构建各种形式的Lock。

包括：

* 一个volatile同步状态 + 一个FIFO双向队列（暂存阻塞状态的线程的队列）
* 对同步状态的同步操作
* 对FIFO队列的同步操作

#### Lock + AbstractQueuedSynchronized

* Lock对锁的使用者提供公共接口
* AbstractQueuedSynchronized对锁的开发者提供了锁的语义实现

## AQS模型：同步状态+线程FIFO队列

### 同步状态

> 同步状态代表的是：公共资源 获取到同步状态，则表示拥有了对公共资源的访问权限

```text
volatile int state; // 同步状态
```

#### 与同步状态相关的操作

* getState\(\)
* setState\(int newState\)
* compareAndSetState\(int expect, int update\) // CAS原子操作

### 线程FIFO队列

> 由Node组织起来的双向链表

#### Node模型

![](../.gitbook/assets/node.png)

#### 队列模型

链表，队列只包含首节点head和尾节点tail

包含tail，方便入队

@图一，初始队列

![](../.gitbook/assets/aqs_queue_init.png)

@图二，插入第一个节点

![](../.gitbook/assets/aqs_queue_first_.png)

在插入第一个节点时，其实队列的首节点是个空的node。这样做，主要是为了队列中阻塞状态的线程的行为是一致的（参见下一节：[队列中的线程在做什么？](abstractqueuedsynchronizer.md#dead-loop)）。

```text
volatile Node head;
volatile Node tail;

// 入队操作
/**
     * Inserts node into queue, initializing if necessary. See picture above.
     * @param node the node to insert
     * @return node's predecessor
     */
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node())) // CAS设置head
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) { // CAS设置tail
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

#### 队列中的线程在做什么？

队列中的线程会处于阻塞（死循环）中，并且只有队列头部的那个线程有机会获取到同步状态（独占式获取同步状态）

```text
for (;;) {
    final Node p = node.predecessor(); 
    // 第一个真实节点的前节点是个空的node，这样所有真实节点的处理逻辑都是一样的
    // 都是判断前节点是否首节点
    if (p == head && tryAcquire(arg)) { // 队列头部的线程有机会获取到同步状态，后续线程一直处在阻塞中
        setHead(node); // 只有一个节点设置自己为首节点，所以head不需要同步
        p.next = null; // help GC
        failed = false;
        return interrupted;
    }
    // 此处忽略部分源码
}
```

```text
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

![](../.gitbook/assets/aqs_queue_twonode.png)

![](../.gitbook/assets/aqs_queue_sethead.png)

@图三 出队操作 setHead操作，为了维护FIFO

#### 这个队列模型的特点

1. 如果队列不为空，则队列有两种状
   1. 所有真实节点都阻塞
   2. 首真实节点获取到同步状态并且执行，而其他真实节点都阻塞
2. 为什么在获取同步状态时，需要判断前节点是否首节点？
   1. 可以保证这个链式数据结构是个FIFO的Queue
3. 为什么是双向的？
   1. 参考2，因其需要找前节点

### AQS同步模式

AQS同步框架支持两大类的同步：独占式+共享式

在每种类型的同步中又细分为以下三种模式

* 普通模式
* 响应中断模式
* 超时并响应中断模式三类

下面将详细介绍各类同步模式。

#### AQS三大类模板方法

* 独占式获取与释放同步状态
  * acquire\(int arg\)
  * acquireInterruptibly\(int arg\) // 响应中断
  * tryAcquireNanos\(int arg, long nanos\)
  * release\(int arg\)
* 共享式获取与释放同步状态
  * acquireShared\(int arg\)
  * acquireSharedInterruptibly\(int arg\) // 响应中断
  * tryAcquireSharedNanos\(int arg, long nanos\)
  * releaseShared\(int arg\)
* 查询同步队列中的等待线程情况
  * Collection&lt;Thread&gt; getQueuedThreads\(\)

#### AQS可重写的方法

* protected boolean tryAcquire\(int arg\) 独占式获取锁
* protected boolean tryRelease\(int arg\) 独占式释放锁
* protected int tryAcquireShared\(int arg\) 共享式获取锁，&gt;=0表示获取成功
* protected boolean tryReleaseShared\(int arg\) 共享式释放锁
* protected boolean isHeldExclusively\(\) 是否被当前线程独占

> ~~画一个线程从lock（）到release（）的函数调用时序图~~

#### 独占式·同步状态获取acquire\(int arg\)

> 互斥访问
>
> 同步状态只有两个合法状态

调用AQS.acquire\(int arg\)模板方法，然后在子类中重写protected boolean tryAcquire\(int arg\)，注意此方法只会返回true/false。

```text
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

// tryAcquire方法
protected boolean tryAcquire(int arg)
```

独占模式下，阻塞中的线程在做什么？参见[队列中的线程在做什么？](abstractqueuedsynchronizer.md#dead-loop)

#### 共享式·同步状态获取与释放

> 同时支持若干线程访问 同步状态有多个合法状态

调用AQS.acquireShared\(int arg\)模板方法，然后在子类中重写protected int tryAcquireShared\(int arg\)，注意此方法只会返回int（与独占式访问返回boolean不同）。

```text
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

```text
// 共享模式下，阻塞中的线程在做什么
for (;;) {
    final Node p = node.predecessor();
    if (p == head) {
        int r = tryAcquireShared(arg);
        // 与独占式的区别，此处不再判断true/false，而是判断一个int>=0
        // 在共享模式下，同步状态有多个合法值，所以不再用true/false，改用int
        if (r >= 0) { 
            setHeadAndPropagate(node, r);
            p.next = null; // help GC
            if (interrupted)
                selfInterrupt();
            failed = false;
            return;
        }
    }
    if (shouldParkAfterFailedAcquire(p, node) &&
        parkAndCheckInterrupt())
        interrupted = true;
}
```

#### 关于响应中断

> 以响应中断·独占式·同步状态获取acquireInterruptibly\(int arg\)为例 响应中断是如何实现的？方法入口+阻塞中 其他同步模式中响应中断的方式与此类似

```text
// 方法入口处响应中断
public final void acquireInterruptibly(int arg) throws InterruptedException {
    if (Thread.interrupted()) // 此处响应中断
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}

// 阻塞中响应中断
private void doAcquireInterruptibly(int arg) throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException(); // 此处响应中断
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### 超时·响应中断·独占式·同步状态获取tryAcquireNanos\(arg, timeout\)

> 独占式、共享式，超时访问都响应中断，中断方式见[关于响应中断](abstractqueuedsynchronizer.md#interrupt) 这里关注超时是如何实现

```text
// 以独占式超时访问为例
// 超时实现：在自旋阻塞时，不断计算剩余等待时长，若此时长<=0，则表明已超时
private boolean doAcquireNanos(int arg, long nanosTimeout) throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout; // 计算出终止时间
    // 省略
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime(); // 计算剩余等待时长
            if (nanosTimeout <= 0L) // 若等待时长<=0，表明已超时
                return false;
            // 省略
        }
    } finally {
        // 省略
    }
}
```

