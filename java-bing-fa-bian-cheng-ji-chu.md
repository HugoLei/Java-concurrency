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
* 默认继承父类优先级（而不是默认值5）