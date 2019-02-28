最核心：CAS 要解决的问题是——保证原子性

---

# CAS（Compare and Swap）

中文含义是：比较并交换

# CAS的目的：系统指令保证其操作是原子的

对于`i++`这种`读改写`操作，系统是无法保证其原子性的。
但是对于 CAS，其读比较写操作，系统通过指令来保证这一过程是原子的。

CAS操作的意思是，先比较内存中的值与预期的旧值是否相同，若相同就更新成新的值。

该操作通常包含三个参数（内存地址，预期的旧值，新值）。

## 对比`i++`和 CAS
#### 非原子操作i++
简单来说，i++语句共包含了3步操作：
1. 读取i
2. i + 1
3. 写回 i

#### i++与缓存一致性协议
假设线程 A 和 B 同时访问 i。
1. A 和 B 都将 i 所在的内存行读入到对应的 CPU 高速缓存
2. A 计算了 i + 1，得到值 m
3. B 计算了 i + 1，得到值 m
4. A 更新自己 CPU 高速缓存中的缓存行里i 的值为 m
5. 根据缓存一致性协议，B 的缓存行将失效，B 将重新读取i 所在的缓存行
6. B 将计算得到的值 m 写到自己的高速缓存行（我们期望的是 m+1）



```
因为 i++不是原子操作，所以即使有缓存一致性协议，也无法保证 i++的结果符合我们预期。
```

#### CAS 与缓存一致性协议
假设线程 A 和 B 同时访问 i。
1. A 和 B 都将 i 所在的内存行读入到对应的 CPU 高速缓存
2. A CAS（i, m）
3. A 更新自己 CPU 高速缓存中的缓存行里i 的值为 m
4. 根据缓存一致性协议，B 的缓存行将失效，B 将重新读取i 所在的缓存行
5. B CAS（i, m）失败，因为此时 i 的值已经变成 m 了

```
因为CAS 是原子的，配合上缓存一致性协议，就能保证结果符合我们的预期。
```


# AtomicInteger中CAS的应用示例一

> compareAndSet\(\)

```
/**
* Atomically sets the value to the given updated value
* if the current value {@code ==} the expected value.
* @param expect the expected value
* @param update the new value
* @return {@code true} if successful. False return indicates that
* the actual value was not equal to the expected value.
*/
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

其中`private static final Unsafe unsafe = Unsafe.getUnsafe();`

AtomicInteger的`compareAndSet()`使用了`unsafe`的`compareAndSwapInt()`方法，如果内存中的值与预期的旧值`expect`相同，则修改内存中的值为`update`，并返回true，否则返回false。

# AtomicInteger中CAS的应用示例二

> getAndIncrement\(\)

```
/
**
* Atomically increments by one the current value.
* @return the previous value
*/
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

# 初探Unsafe

> `unsafe.getAndAddInt()`

该方法先获取内存中对应的值，再执行CAS操作，如果操作成功，会返回更新前的旧值。

## CAS & volatile

**同时为了保证操作一定能成功，需要将CAS操作放在循环里，并且需要volatile特性配合每次读到的都是最新值。**

```
public final int getAndAddInt(Object arg0, long arg1, int arg3) {
    int arg4;
    do {
        arg4 = this.getIntVolatile(arg0, arg1);
    } while (!this.compareAndSwapInt(arg0, arg1, arg4, arg4 + arg3));
    return arg4;
}
```

> 底层调用native方法

```
public native int getIntVolatile(Object arg0, long arg1);
public final native boolean compareAndSwapInt(Object arg0, long arg1, int arg3, int arg4);
```

> 关于参数

以方法`getIntVolatile`为例，`Object arg0`表示的是实例对象的地址，`long arg1`表示的是在该对象内的offset。

这个方法就是找到实例对象在内存中的起始地址，并通过`offset`定位到要读取的值，同时方法名称`getIntVolatile`表明是要读取一个4byte 的int。

关于offset，在AtomicInteger里有如下代码

```
private volatile int value;
private static final long valueOffset;
static {
    try {
        // valueOffset记录的就是value字段在AtomicInteger实例对象中的offset
        valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

# 为什么使用CAS

现代CPU都能保证CAS操作是原子的，因此可以用CAS操作来保证线程安全。

这样可以减少开发人员对并发的控制操作。

