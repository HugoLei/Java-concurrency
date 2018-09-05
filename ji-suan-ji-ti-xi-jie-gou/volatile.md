
举个例子：
```
//线程1
boolean stop = false;
while(!stop){
    doSomething();
}
 
//线程2
stop = true;
```
# 非 volatile 时可能发生的情况
当线程2更改了stop变量的值之后，但是还没来得及写入主存当中，线程2转去做其他事情了，那么线程1由于不知道线程2对stop变量的更改，因此还会一直循环下去。
# volatile 语义
1. 使用volatile关键字会强制将修改的值立即写入主存；
2. 使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量stop的缓存行无效；
3. 由于线程1的工作内存中缓存变量stop的缓存行无效，所以线程1再次读取变量stop的值时会去主存读取。

# Volatile 原理
> Volatile 涉及两个层次，一层是 Java 多线程内存模型，一层是多 CPU 高速缓存一致性

## Java多线程内存模型
参考[Java多线程内存模型](/jvm/java-nei-cun-mo-xing.md)
可知共享变量实际是被拷贝到了线程的本地内存中。
当线程 A 修改 Volatile 变量时，会将修改后的值写回到主存。
当线程 B 读取 Volatile 变量时，
