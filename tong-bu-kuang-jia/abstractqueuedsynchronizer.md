# 同步器AbstractQueuedSynchronized

提供了一个通用的多线程同步管理的基础框架，在此框架之上可以构建各种形式的Lock。

包括：

* 一个volatile同步状态 + 一个FIFO双向队列（暂存阻塞状态的线程的队列）
* 对同步状态的同步操作
* 对FIFO队列的同步操作

### Lock + AbstractQueuedSynchronized

* Lock对锁的使用者提供公共接口
* AbstractQueuedSynchronized对锁的开发者提供了锁的语义实现

### 同步状态

> 同步状态代表的是：公共资源
>
> 获取到同步状态，则表示拥有了对公共资源的访问权限

```
volatile int state; // 同步状态
```

##### 与同步状态相关的操作

* getState\(\)
* setState\(int newState\)
* compareAndSetState\(int expect, int update\) // CAS原子操作

### 线程FIFO队列

> 由Node组织起来的双向链表

##### Node模型

![](/assets/node.png)

##### 队列模型

链表，队列只包含首节点head和尾节点tail

包含tail，方便入队

@图一，初始队列

![](/assets/AQS_queue_init.png)

@图二，插入第一个节点

![](/assets/AQS_queue_first_.png)

在插入第一个节点时，其实队列的首节点是个空的node。这样做，主要是为了队列中阻塞状态的线程的行为是一致的（参见下一节：队列中的线程在做什么？）。

```
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

##### 

##### 队列中的线程在做什么？

队列中的线程会处于阻塞（死循环）中，并且只有队列头部的那个线程有机会获取到同步状态

```
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

```
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

![](/assets/AQS_queue_sethead.png)

@图三 出队操作 setHead操作，为了维护FIFO

### 这个队列模型的特点

1. 如果队列不为空，则队列有两种状
   1. 所有节点都阻塞
   2. 首节点获取到同步状态并且执行，而其他节点都阻塞
2. 为什么在获取同步状态时，需要判断前节点是否首节点？
   1. 可以这个链式数据结构是个FIFO的Queue
3. 为什么是双向的？
   1. 参考2，因其需要找前节点

### AbstractQueuedSynchronized三大类模板方法

* 独占式获取与释放同步状态
  * acquire\(int arg\)
  * acquireInterruptibly\(int arg\)
  * tryAcquireNanos\(int arg, long nanos\)
  * release\(int arg\)
* 共享式获取与释放同步状态
  * acquireShared\(int arg\)
  * acquireSharedInterruptibly\(int arg\)
  * tryAcquireSharedNanos\(int arg, long nanos\)
  * releaseShared\(int arg\)
* 查询同步队列中的等待线程情况
  * Collection&lt;Thread&gt; getQueuedThreads\(\)

### AbstractQueuedSynchronized可重写的方法

* protected boolean tryAcquire\(int arg\) 独占式获取锁
* protected boolean tryRelease\(int arg\) 独占式释放锁
* protected int tryAcquireShared\(int arg\) 共享式获取锁，&gt;=0表示获取成功
* protected boolean tryReleaseShared\(int arg\) 共享式释放锁
* protected boolean isHeldExclusively\(\) 是否被当前线程独占

> ~~画一个线程从lock（）到release（）的函数调用时序图~~





### 独占式·同步状态获取与释放

> 互斥访问
>
> 同步状态只有true false

1. 获取成功，正常执行
2. 获取失败，自旋CAS加入tail后面，进入同步队列
3. 线程进入自旋阻塞（自旋阻塞其实就是死循环）
4. 在自旋阻塞过程中尝试获取锁（判断前节点是否是head，是head并且获取到锁，把自身设置为head）
5. 释放锁，唤醒首节点的后继节点（通过LockSupport）

> 图片

### 共享式·同步状态获取与释放

> 同时支持若干线程访问
>
> 同步状态代表可同时访问的线程的数量

与独占式的区别：

独占式中获取锁的方法，返回值是true false（互斥的）

共享式中获取锁的方法，返回值&gt;=0表示获取到锁

### 超时·可中断·独占式·同步状态获取与释放

1. 在自旋阻塞过程中，若获取锁成功，返回
2. 若失败，判断超时时间（nanosTimeout  &lt; 0），超时则返回
3. 更新超时时间（nanosTimeout -= now - lastTime，超时时间不断缩小）
4. 如果线程被interrupt，抛出中断异常

> 图片



