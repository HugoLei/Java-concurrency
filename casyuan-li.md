# CAS（Compare and Swap）

中文含义是：比较并交换

# CAS操作

CAS操作的意思是，先比较内存中的值与预期的旧值是否相同，若相同就更新成新的值。

该操作通常包含三个参数（内存地址，预期的旧值，新值）。

AtomicInteger中CAS的应用

`/**`

`     * Atomically sets the value to the given updated value`

`     * if the current value {@code ==} the expected value.`

`     *`

`     * @param expect the expected value`

`     * @param update the new value`

`     * @return {@code true} if successful. False return indicates that`

`     * the actual value was not equal to the expected value.`

`     */`

`    public final boolean compareAndSet(int expect, int update) {`

`        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);`

`    }`

其中private static final Unsafe unsafe = Unsafe.getUnsafe\(\);

AtomicInteger的`compareAndSet()`使用了`unsafe`的`compareAndSwapInt()`方法，如果内存中的值与预期的旧值`expect`相同，则修改内存中的值为`update`，并返回true，否则返回false。

