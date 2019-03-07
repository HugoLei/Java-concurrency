# main 方法与多线程
一个简单的 main 函数运行时，涉及到了多个线程
1. main 线程
2. Reference Handler 线程
3. Finalizer 线程
4. Signal Dispatcher 线程

# 线程优先级
* 最低是1
* 默认是5
* 最高是10
* 默认继承父类优先级（而不是默认值5，父线程优先级低，自然的它的子线程优先级也应该低）

setPriority(int)方法修改优先级并不一定生效，跟 JVM 版本和操作系统都有关系。

> 当然，对于 OS来说，Java 线程优先级肯定不会比鼠标键盘的优先级高。