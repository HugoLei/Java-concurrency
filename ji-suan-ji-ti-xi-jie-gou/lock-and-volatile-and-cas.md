# Lock指令，Volatile，CAS 三者之间的关系

# Intel 手册对 lock 前缀指令的说明：
1. 确保对内存的读-改-写操作，原子执行。（原子性：缓存锁，总线锁）
2. 禁止重排：禁止该指令与其前后的读写指令重排。
3. 把写缓冲区的所有数据刷新到内存中。

# lock 指令 与 Volatile
上述2和3，具有内存屏障的效果，足以同时实现 Volatile 读和 Volatile 写的语义（Volatile 基于内存屏障实现）

# lock 指令 与 CAS
CAS 操作对应的指令是 lock cmpxch，是个 lock 前缀的指令

# CAS 与 Volatile
因为 CAS 是个 lock 前缀的指令，所以其本身具有 Volatile 读和 Volatile 写的语义。