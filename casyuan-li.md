#                         CAS（Compare and Swap）

中文含义是：比较并交换

# CAS操作

CAS操作的意思是，先比较内存中的值与预期的旧值是否相同，若相同就更新成新的值。

该操作通常包含三个参数（内存地址，预期的旧值，新值）。

# AtomicInteger中CAS的应用示例一

> compareAndSet\(\)

    `/**`

    `* Atomically sets the value to the given updated value`

     `* if the current value {@code ==} the expected value.`

    `* @param expect the expected value`

    `* @param update the new value`

                     `* @return {@code true} if successful. False return indicates that`

`* the actual value was not equal to the expected value.`

`*/`

`public final boolean compareAndSet(int expect, int update) {`

`return unsafe.compareAndSwapInt(this, valueOffset, expect, update);`

`}`

其中`private static final Unsafe unsafe = Unsafe.getUnsafe();`

AtomicInteger的`compareAndSet()`使用了`unsafe`的`compareAndSwapInt()`方法，如果内存中的值与预期的旧值`expect`相同，则修改内存中的值为`update`，并返回true，否则返回false。

# AtomicInteger中CAS的应用示例二

> getAndIncrement\(\)

`/**`

`* Atomically increments by one the current value.`

`* @return the previous value`

`*/`

`public final int getAndIncrement() {`

`return unsafe.getAndAddInt(this, valueOffset, 1);`

`}`

# 初探Unsafe

> `unsafe.getAndAddInt()`

该方法先获取内存中对应的值，再执行CAS操作，如果操作成功，会返回更新前的旧值。

**同时为了保证操作一定能成功，需要将CAS操作放在循环里。**

    `public final int getAndAddInt(Object arg0, long arg1, int arg3) {`

        `int arg4;`

        `do {`

            `arg4 = this.getIntVolatile(arg0, arg1);`

        `} while (!this.compareAndSwapInt(arg0, arg1, arg4, arg4 + arg3));`

        `return arg4;`

    `}`

> 底层调用native方法

    `public native int getIntVolatile(Object arg0, long arg1);`

    `public final native boolean compareAndSwapInt(Object arg0, long arg1, int arg3, int arg4);`

> 关于参数

以方法`getIntVolatile`为例，`Object arg0`表示的是实例对象的地址，`long arg1`表示的是在该对象内的offset。

这个方法就是找到实例对象在内存中的起始地址，并通过`offset`定位到要读取的值，同时方法名称`getIntVolatile`表明是要读取一个4byte 的int。

关于offset，在AtomicInteger里有如下代码

    `private volatile int value;`

    `private static final long valueOffset;`

    `static {`

        `try {`

            `valueOffset = unsafe.objectFieldOffset`

           `(AtomicInteger.class.getDeclaredField("value"));`

        `} catch (Exception ex) { throw new Error(ex); }`

    `}`

# 为什么使用CAS

现代CPU都能保证CAS操作是原子的，因此可以用CAS操作来保证线程安全。

这样可以减少开发人员对并发的控制操作。

