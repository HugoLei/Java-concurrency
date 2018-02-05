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

通过工厂类Executors创建三大类线程池

1. SingleThreadExecutor
2. FixedThreadPool
3. CachedThreadPool

线程池核心参数：

* corePool：核心线程池大小
* maximumPool：最大线程池的大小
* workQueue：暂时保存任务的工作队列
* keepAliveTime：多余的空闲线程等待新任务的最长时间
* handler：RejectedExecutionHandler，饱和时如何拒绝新的认为

|  |  |
| :--- | :--- |
|  |  |



