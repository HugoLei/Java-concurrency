# CAS（Compare and Swap）

中文含义是：比较并交换

# CAS操作

CAS操作的意思是，先比较内存中的值与预期的旧值是否相同，若相同就更新成新的值。

该操作通常包含三个参数（内存地址，预期的旧值，新值）。

# AtomicInteger中CAS的应用一

> compareAndSet\(\)

`/**`

`* Atomically sets the value to the given updated value`

`* if the current value {@code ==} the expected value.`

`*`

`* @param expect the expected value`

`* @param update the new value`

`* @return {@code true} if successful. False return indicates that`

`* the actual value was not equal to the expected value.`

`*/`

`public final boolean compareAndSet(int expect, int update) {`

`return unsafe.compareAndSwapInt(this, valueOffset, expect, update);`

`}`

其中private static final Unsafe unsafe = Unsafe.getUnsafe\(\);

AtomicInteger的`compareAndSet()`使用了`unsafe`的`compareAndSwapInt()`方法，如果内存中的值与预期的旧值`expect`相同，则修改内存中的值为`update`，并返回true，否则返回false。

# AtomicInteger中CAS的应用二

> getAndIncrement\(\)

`/**`

`     * Atomically increments by one the current value.`

`     *`

`     * @return the previous value`

`     */`

`    public final int getAndIncrement() {`

`        return unsafe.getAndAddInt(this, valueOffset, 1);`

`    }`

> unsafe.getAndAddInt\(\)

该方法先获取内存中对应的值，再执行CAS操作，如果操作成功，会返回更新前的旧值。

同时为了保证操作一定能成功，需要将CAS操作放在循环里。

`public final int getAndAddInt(Object arg0, long arg1, int arg3) {`

`		int arg4;`

`		do {`

`			arg4 = this.getIntVolatile(arg0, arg1);`

`		} while (!this.compareAndSwapInt(arg0, arg1, arg4, arg4 + arg3));`

`		return arg4;`

`	}`

