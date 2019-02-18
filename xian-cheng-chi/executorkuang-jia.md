# 线程池 Executor框架

Executor框架三大模块

1. 任务（对任务的抽象）
2. 任务的管理（线程池）
3. 任务的返回结果

@线程池模型的图

| 模块 | 内容 |
| :--- | :--- |
| 任务 | Runnable接口，Callable接口 |
| 任务的管理 | ThreadPoolExecutor，ScheduledThreadPoolExecutor |
| 任务的返回结果 | Future接口 |

### 任务的管理

两种任务管理器：

1. ThreadPoolExecutor
2. ScheduledThreadPoolExecutor


```
ScheduledThreadPoolExecutor继承自ThreadPoolExecutor
最主要的区别是：workQueue 支持 schedule
```



任务管理器核心参数：
* `corePool`：核心线程池大小
* `maximumPool`：最大线程池的大小
* `workQueue`：暂时保存任务的工作队列
* `keepAliveTime`：多余的空闲线程等待新任务的最长时间
* `handler`：RejectedExecutionHandler，饱和时如何拒绝新的任务
* `allowCoreThreadTimeOut`: default false，默认 core thread 不会 timeout


管理模式：

1. 如果`正在运行的`线程数少于corePool，则创建线程
2. `正在运行的`线程数大于等于corePool，将任务加入工作队列中
3. 如果工作队列已满，且`正在运行的`线程数少于maximumPool，则创建线程
4. 若`正在运行的`线程数大于等于maximumPool，则执行拒绝新任务的策略
5. 超过 coreSize的空闲线程，等待新任务超时后，结束线程
6. `core size 线程默认不会timeout`


### 通过工厂类Executors创建四大类线程池

1. SingleThreadExecutor
2. FixedThreadPool
3. CachedThreadPool
4. ScheduledThreadPoolExecutor

#### ThreadPoolExecutor

| 线程池 | corePool | maximumPool | BlockingQueue | keepAliveTime |
| :--- | :--- | :--- | :--- | :--- |
| FixedThreadPool | n | n | LinkedBlockingQueue | 0 |
| SingleThreadExecutor | 1 | 1 | LinkedBlockingQueue | 0 |
| CachedThreadPool | 0 | Integer.MAX\_VALUE | SynchronousQueue | 60s |

##### BlockingQueue：用来存任务（而不是线程，不要跟线程的同步队列搞混）

1. LinkedBlockingQueue：无界队列（大小为Integer.MAX\_VALUE）
1.1 因为可以一直将任务加入到队列中，所以maximumPool参数就无意义了

2. SynchronousQueue：无容量，提交的任务必须等一个线程来处理
2.1 因为CachedThreadPool的maximumPool是无界的，因此可能出现创建过多的线程，导致耗尽CPU和内存资源

##### Pool 中 Thread 数量的变化


```
默认 core Thread 不会 timeout
```


![](/assets/IMG_4997.jpg)

#### ScheduledThreadPoolExecutor

| 线程池 | corePool | maximumPool | BlockingQueue | keepAliveTime |
| :--- | :--- | :--- | :--- | :--- |
| ScheduledThreadPoolExecutor | n | Integer.MAX\_VALUE | DelayedWorkQueue | 0 |

##### schedule() 延迟执行

```
schedule(Runnable command,long delay,TimeUnit unit)
```

##### scheduleAtFixedRate() 固定周期执行

```
scheduleAtFixedRate(Runnable command,long initialDelay,long period, TimeUnit unit)
```
* initialDelay: 初始延迟
* period：周期



Scheduled任务有三个属性：

* time // 具体的执行时间
* sequenceNumber // 该任务的序号
* period // 任务的间隔周期

DelayedWorkQueue：

* 封装了一个PriorityQueue
* 排序优先级：time小，sequenceNumber小

### 任务的返回结果

如何拿到别的任务的执行结果？

线程能够返回结果的本质是：一个线程能看到另一个线程的执行结果，也即线程间的通信[/线程间的通信](/线程间的通信)

#### Future接口 和 FutureTask类

FutureTask类实现了Future接口

当前线程提交了FutureTask任务后，若想要获取任务执行的结果，需要调用FutureTask的get\(\)

如果get\(\)不成功，当前线程会被阻塞住

FutureTask任务执行完后，执行结果放在任务对象里，然后唤醒当前线程，当前线程从任务对象里获取结果

> 核心：共享变量（任务对象）



