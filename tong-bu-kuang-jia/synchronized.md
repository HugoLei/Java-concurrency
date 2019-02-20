Synchronized隐式地实现同步控制，对开发者透明。

原理说明
使用示例

Java SE 1.6对 synchronized 进行了多种优化。
1. 自旋
2. 锁粗化
3. 锁消除
4. 偏向锁 - 轻量级锁 - 重量级锁

> 下面根据锁的四大问题分别进行说明。[《关于锁》](/guan-yu-suo.md)

# synchronized 锁的是一个对象

| 锁的形式|锁定的对象|
|:--|:--|
|普通同步方法|当前实例对象|
|静态同步方法|Class 对象|
|同步代码块|synchronized(参数对象)|

# 锁存在哪里（对象头的 Mark word 里）
## Java 对象头
在 JVM 中每个对象都有个“头”
数组对象：JVM 用3个字宽 word 来存对象头
非数组对象：JVM 用2个字宽 word 存对象头

## 32位系统对象头说明（字宽 word 是32bit）
|长度|内容|说明|
|:--|:--|:--|
|1字宽|Mark Word|hashcode 或锁信息等|
|1字宽|Class Metadata Address|到对象类型数据的指针|
|1字宽|Array Length|数组长度（如果是数组的话，否则没有这个字宽）|

## Mark word 示例
![](/assets/Mark word.png)

# “谁拥有锁”这个信息存在哪里？
1. 偏向锁：ThreadId 放在 Mark word 里
2. 轻量级锁：Mark word 指向线程的 Lock record，在 Lock record 里记录 owner 为拥有锁的线程
3. 重量级锁：放在 monitor 中，直接对标操作系统的 mutex（未确认）

# 偏向锁 - 轻量级锁 - 重量级锁

## Synchronized 实现互斥
直接使用上述三种形式之一

## Synchronized 实现等待/通知模型
1. 上述三种形式之一 
2. object.wait() object.notify() object.notifyAll()
1 和 2 都是隐式实现


### 加了偏向锁之后，对象的 hashcode 存在哪？

## 偏向锁？锁的升级与对比
Java SE 1.6中锁有4种状态，级别从低到高依次是：无锁状态，偏向锁，轻量级锁，重量级锁

### 为什么要有多种状态的锁？
为了减少获取锁和释放锁带来的性能损耗


# 隐式锁 vs 显示锁

| | 使用便捷性 | 灵活性 | 可中断的锁 | 超时获取锁 | 互斥/共享 | 阻塞 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Synchronized | 隐式获取/释放锁，代码简洁 | 固化，自动执行 | No | No | 只互斥 | 阻塞 |
| Lock | 显示获取/释放锁，注意编码技巧 | 灵活，手动控制 | Yes | Yes | 可互斥，可共享 | 可非阻塞 |

可中断的锁：在获取锁的过程中或在同步队列中，可响应中断

超时获取锁：如果线程在超时时间内没有获取到锁，则返回