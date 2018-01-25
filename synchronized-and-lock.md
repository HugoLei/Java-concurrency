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

# AbstractQueuedSynchronized

提供了

* FIFO队列来维护等待线程
* 状态访问/修改方法

Lock + AbstractQueuedSynchronized

* Lock对锁的使用者提供公共接口
* AbstractQueuedSynchronized对锁的开发者提供了锁的语义实现

AbstractQueuedSynchronized三大类模板方法

* 独占式获取与释放同步状态
* 共享式获取与释放同步状态
* 查询同步队列中的等待线程情况



