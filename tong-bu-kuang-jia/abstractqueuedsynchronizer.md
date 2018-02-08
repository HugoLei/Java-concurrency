# 同步器AbstractQueuedSynchronized

提供了

* 暂存等待状态的线程的队列
* 对同步状态的访问/修改

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



