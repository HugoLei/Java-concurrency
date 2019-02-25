# Java Concurrency

> 已经有那么多书讲这个东西，该领域已经研究得很透彻了，那我写这个文档的意义是什么？

关于Java并发

![](/assets/framework.png)
（此图持续更新）

本文档将从5个层面介绍Java的并发编程：

1. 线程池
2. 同步框架
3. JVM
4. 操作系统
5. 计算机体系结构

整体的思路是从应用层的线程池出发，一直下探到底层的计算机体系结构，也即CPU、高速缓存、主存等。

窃以为这种思路是实践驱动的、目标导向的，学习动力会更强一些。回想在大学的四年里，我们分别学习了高数、离散数学、概率论、编译原理、计算机体系结构、C语言、数据结构算法等课程，一路课程下来，发现学习兴趣并不是很浓厚，其中一个很重要的原因是，我们不清楚学的这些课程有什么用，不清楚课程之间有什么联系，说白了就是没有强实践支撑。

所以本文档采取了从实践出发的思路，以大家常用的线程池技术为入口，一层层探索Java的多线程到底是如何实现的。这种分层思路的另一个好处是，如果你对底层不是很感兴趣，那可以只看到JVM层；如果对硬件不感兴趣，那看到操作系统层就可以了，因此非常灵活，适用范围较广。

想法当然是好的。如果这个文档完成了，那将是对Java多线程的一次连贯的、深入浅出的介绍，更有意义的是这个五层模型，可以帮助大家以结构化的思维去考虑Java多线程，比如你可以按照这个五层模型滔滔不绝地给面试官讲2个小时。

现实是残酷的。这个文档牵扯了太多的东西进来，甚至一直深入到CPU设计这一层，因此需要庞大的知识树。仅凭一人之力犹如愚公移山，另一方面每个人的专长不一样，如果让专业的人来做擅长的事，那结果将更具有说服力。因此非常欢迎各位同好来一起完善此文档，是对个人思维、知识、写作等能力的一次锻炼，也能为同行提供非常有价值的成果。通力合作，便可“达则兼济天下”，岂不悦乎？

也非常欢迎任何对本文档感兴趣的朋友，集思广益。也许你的一个意见，可以让这个文档的思路变得更加的通顺，会提问题也是一种优秀的能力。再者，广结同好，岂不快哉？

