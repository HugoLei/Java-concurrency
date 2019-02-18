### getName()的区别


```
首先理清一个逻辑：首先我们写了一个任务（可以运行run()），然后把这个任务放在一个 Thread 上去执行。
任务是任务，Thread 是 Thread
```


#### Thread.currentThread().getName()
拿到的是 Thread 的名字

#### this.getName()
拿到的是任务的名字