# 线程池 Executor框架

Executor框架三大模块

1. 任务（对任务的抽象）
2. 任务的管理（线程池）
3. 任务的返回结果

| 模块 | 内容 |
| :--- | :--- |
| 任务 | Runnable接口，Callable接口 |
| 任务的管理 | ThreadPoolExecutor，ScheduledThreadPoolExecutor |
| 任务的返回结果 | Future接口 |

### 任务的管理

两种任务管理器：

1. ThreadPoolExecutor
2. ScheduledThreadPoolExecutor

管理模式：

1. 如果线程数少于corePool，则创建线程
2. 线程数大于等于corePool，将任务加入工作队列中
3. 如果工作队列已满，且线程数少于maximumPool，则创建线程
4. 若线程数大于等于maximumPool，则执行拒绝新任务的策略

通过工厂类Executors创建四大类线程池

1. SingleThreadExecutor
2. FixedThreadPool
3. CachedThreadPool
4. ScheduledThreadPoolExecutor

线程池核心参数：

* corePool：核心线程池大小
* maximumPool：最大线程池的大小
* workQueue：暂时保存任务的工作队列
* keepAliveTime：多余的空闲线程等待新任务的最长时间
* handler：RejectedExecutionHandler，饱和时如何拒绝新的认为

#### ThreadPoolExecutor

| 线程池 | corePool | maximumPool | workQueue | keepAliveTime |
| :--- | :--- | :--- | :--- | :--- |
| FixedThreadPool | n | n | LinkedBlockingQueue | 0 |
| SingleThreadExecutor | 1 | 1 | LinkedBlockingQueue | 0 |
| CachedThreadPool | 0 | Integer.MAX\_VALUE | SynchronousQueue | 60s |

LinkedBlockingQueue：无界队列（大小为Integer.MAX\_VALUE）（因此maximumPool参数就无意义了）

SynchronousQueue：无容量，提交的任务必须等一个线程来处理

因为CachedThreadPool的maximumPool是无界的，因此可能出现创建过多的线程，导致耗尽CPU和内存资源

#### ScheduledThreadPoolExecutor

| 线程池 | corePool | maximumPool | workQueue | keepAliveTime |
| :--- | :--- | :--- | :--- | :--- |
| ScheduledThreadPoolExecutor | n | Integer.MAX\_VALUE | DelayedWorkQueue | 0 |

Scheduled任务有三个属性：

* time // 具体的执行时间
* sequenceNumber // 该任务的序号
* period // 任务的间隔周期

DelayedWorkQueue：

* 封装了一个PriorityQueue
* 排序优先级：time小，sequenceNumber小

### 任务的返回结果



