# main 方法与多线程
一个简单的 main 函数运行时，涉及到了多个线程
1. main 线程
2. Reference Handler 线程
3. Finalizer 线程
4. Signal Dispatcher 线程

# 线程优先级
* 最低是1
* 默认是5
* 最高是10
* 默认继承父类优先级（而不是默认值5，父线程优先级低，自然的它的子线程优先级也应该低）



```
setPriority(int)方法修改优先级并不一定生效，
跟 JVM 版本和操作系统都有关系。
```



> 当然，对于 OS来说，Java 线程优先级肯定不会比鼠标键盘的优先级高。

#### 如何验证 Java 线程的优先级不生效呢？
* 设置线程优先级，启动多个线程
* 改变 Volatile start变量，让多个线程同时开始执行计数
* 改变 Volatile end 变量，让多个线程同时停止计数
* 对比不同优先级线程，计数结果
* 发现不同优先级，计数结果差不多，说明优先级没生效

# 线程状态
Java的 Runnable 状态：包含 OS 的运行和就绪两种状态
![](/assets/线程状态.JPG)
![](/assets/状态变化.JPG)
* Thread.sleep(int) 进入TIMED_WAITING

# Daemon 线程（finally块并不一定执行）
1. Daemon 属性在线程启动之前设置
2. 当 Java 虚拟机中不存在非 Daemon 线程时，虚拟机会退出，然后所有 Daemon 线程立即终止。


```
虚拟机退出时，Daemon线程立即终止，其finally块中的内容并不一定会执行，因此不能依靠 finally 块中的内容来确保执行关闭或清理资源的逻辑
```

# 中断（相当于是发了个信号）
中断相当于是线程的一个标识位属性。
#### 触发中断
其他线程调用该线程的 interrupt()方法，触发中断（相当于把标志位置为中断）
#### 响应中断
该线程通过isInterrupted()检查自身是否被中断
#### 复位
Thread.interrupted()对当前线程的中断标识进行复位
#### InterruptedException
Java API 中声名抛出InterruptedException异常的方法
这些方法在抛出InterruptedException之前，JVM 会将线程的中断标识清除，然后抛出异常。
> 例如 Thread.sleep(int)会抛出InterruptedException

# suspend() resume() stop()的缺陷
这些方法过期，不推荐使用。
因为：
suspend()暂停时，线程不会释放资源
stop()终止线程时，也不保证线程资源正常释放

# 如何安全地终止线程
1. 通过中断标识
2. Volatile boolean共享变量

> 好处：线程有机会来清理资源

