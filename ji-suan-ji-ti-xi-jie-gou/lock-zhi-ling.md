最核心：lock 前缀指令的目的——保证原子性

---
# 原子性操作的理解
> 对原子操作的理解，一定是基于多线程环境。
如果是单线程环境，没人打断你的操作，效果上便是原子的

举例：线程 A 在操作共享变量的过程中，不允许插入其他线程对共享变量的操作。

举例：线程 A 对共享变量 i 执行 i++操作，因为在执行该操作（多条指令）的过程中，允许其他线程同时操作 i，所以 i++操作不是原子的。


# 原子操作举例
1. 处理器保证从内存中读写一个 byte 是原子的（正在处理时，其他处理器不能访问这个字节的地址）
2. 最新的处理器，可以保证单处理器对同一个缓存行里进行16/32/64位的操作是原子的

# lock前缀指令的两种实现方式
> 来自intel手册

# 如何保证操作是原子的？
> 使用锁，禁止其他线程对共享变量操作

#### 总线锁（锁定总线）
早期 CPU，总是采用锁总线的方式。
某个核心遇到 Lock 指令，就触发总线仲裁器，让其独占总线。
其余的 CPU 核心不能再通过总线与内存通讯。
该核心获取总线锁后，执行操作，最后释放锁。
从而达到“原子性”。

#### 总线锁的问题
总线锁导致其他 CPU 无法工作，代价大。
因为锁定期间，其他 CPU 也不能访问其他内存地址的数据。

#### 缓存锁（优化总线锁）
>  前提是在这样一个特定的场景下：要操作的内存区域正好在当前 Cache 的缓存行里，并且该缓存行处于锁定状态（MESI 协议中的 M 态或 E 态）。

此时，写操作可以直接写回 memory，而不需要发起总线锁。
本质是它是让 MESI 缓存一致性协议来保证了操作的原子性。


# 原子操作示例
#### CAS 操作
对应的指令是 cmpxchg，该指令前加 Lock 来保证原子性

