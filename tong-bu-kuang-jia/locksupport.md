# LockSupport

用来阻塞或唤醒线程

> ~~实现原理是什么？~~

提供了下述public static方法

* 用于阻塞线程的方法
  * void park\(\)
  * void park\(Object blocker\)
  * void parkNanos\(long nanos\)
  * void parkNanos\(Object blocker, long nanos\)
  * void parkUntil\(long deadline\)
  * void parkUntil\(Object blocker, long deadline\)
* 用于唤醒线程的方法
  * void unpark\(Thread thread\)

blocker是什么 HotSpot源码

