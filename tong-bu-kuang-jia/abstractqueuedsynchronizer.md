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

> 状态代表的是：公共资源

```
volatile int state;
```

* getState\(\)
* setState\(int newState\)
* compareAndSetState\(int expect, int update\) // CAS原子操作

### 线程FIFO队列

> 由Node组织起来的双向链表

##### Node模型

![](/assets/node.png)

##### 队列模型

队列只包含首节点head和尾节点tail

@图 见书

```
volatile Node head;
volatile Node tail;

// 入队
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

### 

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



