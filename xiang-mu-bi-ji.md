# 项目基本信息

参考：
1. JDK 1.8 source code
2. 查看native方法对应的源码（从OpenJDK里看）
http://download.java.net/openjdk/jdk8/
3. 《Java并发编程的艺术》，方腾飞，魏鹏，程晓明 著
4. 《解密Java虚拟机》，封亚飞 著
5. 《Java多线程编程核心技术》，高洪岩 著
6. 《Effective Java》，Joshua Bloch 著
7. 《Java核心技术 卷I》，Cay S. Horstmann，Gary Cornell 著
8. 《现代操作系统》，Andrew S. Tanenbaum，Herbert Bos 著
9. 《深入理解计算机系统》，Randal E. Bryant 著


# 要修改的问题

1. 多线程应该分成两个大问题来讲
   1. 线程池
   2. 同步控制



1. 前沿里的5个层次需要修改，因为线程池和同步控制是同一个层的
2. 同步控制
   1. Synchronized和Lock是同一个级别，一个是隐式的同步框架，一个是显示的同步框架
   2. 而Lock基于AQS
   3. 同步控制的一种：等待通知模型
   3. 画出等待通知的模型图
   4. Lock需要代码分析
   5. 写这样一种Lock，只有一个厕所，有两个坑，里面有女生时，男生不能进，反之也一样
3. ~~需要画一张大图，把层次及每个层的内容都表达出来~~
4. 要介绍native方法对应的源码
5. 如果要合作完成这个文档，需要完善协作制度，社群运营



