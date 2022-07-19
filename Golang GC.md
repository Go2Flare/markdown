# Golang GC算法

虽然Golang的GC自打一开始，就被人所诟病，但是经过这么多年的发展，Golang的GC已经改善了非常多，变得非常优秀了。

以下是Golang GC算法的里程碑：

v1.1 STW

v1.3 Mark STW, Sweep 并行

v1.5 三色标记法

v1.8 hybrid write barrier

经典的GC算法有三种：引用计数(reference counting)、标记-清扫(mark & sweep)、复制收集(Copy and Collection)。

Golang的GC算法主要是基于标记-清扫(mark and sweep)算法，并在此基础上做了改进。因此，在此主要介绍一下标记-清扫(mark and sweep)算法，关于引用计数(reference counting)和复制收集(copy and collection)可自行百度。

标记-清扫(Mark And Sweep)算法
此算法主要有两个主要的步骤：

标记(Mark phase)

清除(Sweep phase)

第一步，找出不可达的对象，然后做上标记。

第二步，回收标记好的对象。

操作非常简单，但是有一点需要额外注意：mark and sweep算法在执行的时候，需要程序暂停！即stop the world。

也就是说，这段时间程序会卡在哪儿。故中文翻译成卡顿。

我们来看一下图解：

开始标记，程序暂停。程序和对象的此时关系是这样的：



image

然后开始标记，process找出它所有可达的对象，并做上标记。如下图所示：



image

标记完了之后，然后开始清除未标记的对象：



image

然后垃圾清除了，变成了下图这样。



image

最后，停止暂停，让程序继续跑。然后循环重复这个过程，直到process生命周期结束。

标记-清扫(Mark And Sweep)算法存在什么问题？
标记-清扫(Mark And Sweep)算法这种算法虽然非常的简单，但是还存在一些问题：

STW，stop the world；让程序暂停，程序出现卡顿。

标记需要扫描整个heap

清除数据会产生heap碎片

这里面最重要的问题就是：mark-and-sweep 算法会暂停整个程序。

Go是如何面对并这个问题的呢？