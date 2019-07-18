# Fork & Join框架

### 背景

Java7 提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

#### 跟Executors FurtureTask的关系？

Fork：把大任务切分为若干子任务

Join：合并子任务的执行结果

例如：计算1+2+...+10000，可以分割成10个子任务，每个子任务分别对1000个数进行求和，最终汇总这10个子任务的结果得到最后的结果。

### 工作窃取算法

如果将所有任务放在同一个队列中，则多个线程会竞争一个队列。

如果将所有任务放在不同的队列中，每个队列对应一个线程，这样线程间就没有竞争。

窃取

* 有的线程处理得快，队列很快就空了，则该线程可以窃取别的队列中的任务去执行
* 为了减少队列竞争，使用双端队列，两个线程分别从头和尾获取任务

优缺点：

* 优点：减少线程间的竞争，提高并行计算的能力
* 缺点：消耗更多资源，如多个双端队列；竞争并没有被杜绝，在窃取的时候如果只剩一个任务了，还是会发生竞争

### 线程数量

默认情况下，Fork Join框架能使用的线程的数量是`Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors())`，所以通常也就是系统的逻辑核数（我的MBP 是逻辑4核）

```
public ForkJoinPool() {
        this(Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors()),
             defaultForkJoinWorkerThreadFactory, null, false);
    }
```

### 示例：求和

```
@Override
    protected Integer compute() {
        int sum = 0;
        boolean canCompute = (end - start) <= threshold;
        System.out.println("Current thread:" + Thread.currentThread().getName() + "compute: " + start + "--" + end);
        if (canCompute) {
            for (int i=start +1; i<= end; ++i) {
                sum += i;
            }
        } else {
            int middle = (start + end) / 2;
            TestForkJoin leftForkJoin = new TestForkJoin(start, middle);
            TestForkJoin rightForkJoin = new TestForkJoin(middle, end);
            invokeAll(leftForkJoin, rightForkJoin); // 1
//            leftForkJoin.fork(); // 2
//            rightForkJoin.fork(); // 2
            int left = leftForkJoin.join();
            int right = rightForkJoin.join();
            sum = left + right;
        }
        return sum;
    }
```

运营环境：MacBook Pro，1 CPU，2物理内核，4逻辑内核

```
# 分别调用fork()
# 整个计算过程只用到了3个线程
# ForkJoinPool-1-worker-1线程一直处在阻塞中，并未实际参与计算
Current thread:main
Current thread:ForkJoinPool-1-worker-1compute: 0--600
Current thread:ForkJoinPool-1-worker-3compute: 300--600
Current thread:ForkJoinPool-1-worker-3compute: 300--450
Current thread:ForkJoinPool-1-worker-3compute: 300--375
Current thread:ForkJoinPool-1-worker-3compute: 375--450
Current thread:ForkJoinPool-1-worker-3compute: 450--600
Current thread:ForkJoinPool-1-worker-3compute: 450--525
Current thread:ForkJoinPool-1-worker-3compute: 525--600
Current thread:ForkJoinPool-1-worker-2compute: 0--300
Current thread:ForkJoinPool-1-worker-2compute: 0--150
Current thread:ForkJoinPool-1-worker-2compute: 0--75
Current thread:ForkJoinPool-1-worker-2compute: 75--150
Current thread:ForkJoinPool-1-worker-2compute: 150--300
Current thread:ForkJoinPool-1-worker-2compute: 150--225
Current thread:ForkJoinPool-1-worker-2compute: 225--300
cost:7076919ns

# ForkJoinTask# invokeAll()
# 整个计算过程只用到了4个线程
# ForkJoinPool-1-worker-1也执行了部分计算任务，compute: 0--75
# 因此此种方式的性能更好，cost:5002036 < cost:7076919
Current thread:main
Current thread:ForkJoinPool-1-worker-1compute: 0--600
Current thread:ForkJoinPool-1-worker-1compute: 0--300
Current thread:ForkJoinPool-1-worker-1compute: 0--150
Current thread:ForkJoinPool-1-worker-1compute: 0--75
Current thread:ForkJoinPool-1-worker-2compute: 300--600
Current thread:ForkJoinPool-1-worker-1compute: 75--150
Current thread:ForkJoinPool-1-worker-3compute: 150--300
Current thread:ForkJoinPool-1-worker-3compute: 150--225
Current thread:ForkJoinPool-1-worker-0compute: 450--600
Current thread:ForkJoinPool-1-worker-0compute: 450--525
Current thread:ForkJoinPool-1-worker-0compute: 525--600
Current thread:ForkJoinPool-1-worker-2compute: 300--450
Current thread:ForkJoinPool-1-worker-2compute: 300--375
Current thread:ForkJoinPool-1-worker-2compute: 375--450
Current thread:ForkJoinPool-1-worker-1compute: 225--300
cost:5002036ns
```



