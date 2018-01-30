# Synchronized and Lock

Lock接口提供了锁功能。

|  | 使用便捷性 | 灵活性 | 可中断的锁 | 超时获取锁 | 互斥/共享 | 阻塞 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Synchronized | 隐式获取/释放锁，代码简洁 | 固化，自动执行 | No | No | 只互斥 | 阻塞 |
| Lock | 显示获取/释放锁，注意编码技巧 | 灵活，手动控制 | Yes | Yes | 可互斥，可共享 | 可非阻塞 |

可中断的锁：在获取锁的过程中或在同步队列中，可响应中断

超时获取锁：如果线程在超时时间内没有获取到锁，则返回

# 如何实现一个Lock

* 需要一个队列来维护等待线程
* 需要同步访问锁的状态

是不是看着很麻烦？

所以Java并发包提供了队列同步器AbstractQueuedSynchronized来支持锁的开发。

# 同步器AbstractQueuedSynchronized

提供了

* FIFO队列来维护等待线程
* 状态访问/修改方法

### Lock + AbstractQueuedSynchronized

* Lock对锁的使用者提供公共接口
* AbstractQueuedSynchronized对锁的开发者提供了锁的语义实现

### AbstractQueuedSynchronized状态访问与更新

> 状态代表的是：共享的资源

* getState\(\)
* setState\(int newState\)
* compareAndSetState\(int expect, int update\)

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

# 同步队列

一个FIFO双向队列。

同步器拥有同步队列的头节点和尾节点，head和tail，这样就可以完成唤醒头部节点，将新的等待线程加入尾部。

* 首节点：首节点是获取锁成功的节点
* 唤醒：首节点释放锁时，唤醒后继节点，后继节点获取锁成功后，将自己设置为首节点
* 更新tail：防止并发修改tail，用CAS更新tail，compareAndSetTail
* 更新head：只有一个节点设置自己为首节点，所以head不需要同步

### 为什么是双向队列？

1. 因为首节点是成功获取锁的节点，由它负责唤醒后继节点
2. 若更新首节点的操作先于后继节点获取到锁，则可能出现新的首节点不是成功获取到锁的节点
3. 这样的话等待队列就无法被唤醒了
4. 所以只有当后继节点成功获取到锁后，才能更新首节点
5. 所以更新首节点的操作，只能由成功获取锁的后继节点完成
6. 所以后继节点必须能找到首节点
7. 所以是双向的

> 同步队列图片，《Java并发编程的艺术》

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

# TwinsLock

在同一时刻，只允许至多两个线程同时访问，超过两个线程的访问将被阻塞。

### 独占 or 共享？

同一时刻支持多个线程访问，显然是共享的。

* 使用模板方法acquireShared\(int arg\)等share相关方法
* 重写tryAcquireShared\(int arg\)方法
* state变化：合法值为0，1，2，初始为2，当一个线程获取，status减1，当线程释放，status+1

# 重入锁

支持一个线程对资源的重复加锁

支持获取锁时的公平/非公平（为什么可重入要和公平非公平捆绑？）

两个核心问题：

1. 可再次获取锁（锁要记录正在拥有锁的线程，才能保证该线程可再次获取锁）
2. 最终释放（要记录重入次数，释放时次数-1，次数为0时表示释放完成）

用state记录重入次数，因此本锁不支持共享（因为共享和重入次数都需要用state来保证）

### 公平 or 非公平

公平：获取锁的顺序，严格按照请求的时间先后顺序，即FIFO（保证公平的核心：判断是否有前驱节点，有则表示在自己前面还有其他线程等待）

非公平：只要自旋阻塞过程中CAS获取锁成功，就获取了优先执行权，不按请求时间先后顺序来

# 读写锁

> 集共享与排他于一体，读是共享，写是排他
>
> 但读与写又不是毫无关联，写时不能有其他写或读

1. 如何仅通过一个同步状态state来同时达到共享与排他？
2. 可以用两个状态来实现读写锁吗？

### 读写锁同步状态的设计

在一个同步状态上（整型）维护多个读线程和一个写线程

在一个整型上维护多种类型的状态，必然需要“按位切割使用”

高16位：读，低16位：写

同步状态值为S，写状态=S & 0x0000FFFF，读状态=S&gt;&gt;&gt;16

### 写锁的获取与释放

获取：

1. 拿到同步状态 + 写状态
2. 状态 ！= 0，说明有读或者写
   1. 如果有读（写状态==0），失败
   2. 如果有写，但不是当前线程在写，失败
   3. 重入
3. 状态==0，说明无锁
   1. CAS获取锁

释放：

1. 检查是否是当前线程拥有锁
2. 写锁释放一次（重入对应的释放）

### 读锁的获取与释放

获取：

1. 有写锁 && 不是当前线程在写，失败
2. 只要读状态&lt;MAX，就可以CAS获取读锁（因此也支持可重入）

### 锁降级

> 把持住写锁，再获取读锁，然后释放写锁

为什么要锁降级？

# LockSupport工具

用来阻塞或唤醒线程

> ~~实现原理是什么？~~

# Condition接口

监视器

每个Java对象都关联一个监视器，都拥有一组监视器方法（定义在Object中）

等待/通知模型：wait\(\) notify\(\) + synchromized 

