
# 原子操作类
以AtomicInteger为例
1. 字段Volatile value存储值
2. value字段偏移量valueOffset，在 static 块中 unsafe 获取字段的地址偏移量，用于后续 cas
3. 原子修改：循环，get Volatile value， CAS，直到成功
#### 原子更新基本类型
AtomicBoolean（实际是吧 Boolean 转成 int）
AtomicInteger
AtomicLong

只有上述三种，因为底层 unsafe 有：
casObject
casInt
casLong

> 其他类型 char，float 如何处理？
char 可以转成 int
byte可以转成 int
Float.floatToIntBits()可以把 float 转成 int
Float.intBitsToFloat()可以把 int 转成 float
Double 同理
因此可以用 AtomicInteger 实现 float 和 double 的原子操作类

#### 原子更新数组里的某个元素
AtomicIntegerArray
AtomicLongArray
AtomicReferenceArray
#### 原子更新引用类型（或引用类型里的字段）
#### 原子更新类里的字段



