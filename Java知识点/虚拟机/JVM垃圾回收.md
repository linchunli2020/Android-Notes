
***分代收集算法***

对于一个大型的系统，当创建的对象和方法变量比较多时，堆内存中的对象也会比较多，如果逐一分析对象是否该回收，那么势必造成效率低下。
分代收集算法是基于这样一个事实：不同的对象的生命周期(存活情况)是不一样的，而不同生命周期的对象位于堆中不同的区域，因此对堆内存不同区域采用不同的策略进行回收可以提高 JVM 的执行效率。
当代商用虚拟机使用的都是分代收集算法：新生代对象存活率低，就采用复制算法；老年代存活率高，就用标记清除算法或者标记整理算法。

***Java堆内存一般可以分为新生代、老年代和永久代三个模块，如下图所示：***
  
<img width="596" alt="image" src="https://user-images.githubusercontent.com/67937122/161372346-be734cd1-5e2e-45f8-937a-d8cb3e8a199d.png">

***新生代（Young Generation）***

新生代的目标就是尽可能快速的收集掉那些生命周期短的对象，一般情况下，所有新生成的对象首先都是放在新生代的。
新生代内存按照 8:1:1 的比例分为一个eden区和两个survivor(survivor0，survivor1)区，大部分对象在Eden区中生成。
在进行垃圾回收时，先将eden区存活对象复制到survivor0区，然后清空eden区，当这个survivor0区也满了时，则将eden区和survivor0区存活对象复制到survivor1区，
然后清空eden和这个survivor0区，此时survivor0区是空的，然后交换survivor0区和survivor1区的角色（即下次垃圾回收时会扫描Eden区和survivor1区），即保持survivor0区为空，如此往复。
特别地，当survivor1区也不足以存放eden区和survivor0区的存活对象时，就将存活对象直接存放到老年代。
如果老年代也满了，就会触发一次FullGC，也就是新生代、老年代都进行回收。注意，新生代发生的GC也叫做MinorGC，MinorGC发生频率比较高，不一定等 Eden区满了才触发。

***老年代（Old Generation）***

老年代存放的都是一些生命周期较长的对象，就像上面所叙述的那样，在新生代中经历了N次垃圾回收后仍然存活的对象就会被放到老年代中。
此外，老年代的内存也比新生代大很多(大概比例是1:2)，当老年代满时会触发Major GC(Full GC)，老年代对象存活时间比较长，因此FullGC发生的频率比较低。

***永久代（Permanent Generation）***
永久代主要用于存放静态文件，如Java类、方法等。
永久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class，例如使用反射、动态代理、CGLib等bytecode框架时，
在这种时候需要设置一个比较大的永久代空间来存放这些运行过程中新增的类。


自己写的笔记：

<img width="547" alt="image" src="https://user-images.githubusercontent.com/67937122/161372504-abba0283-ac16-49f7-992d-99eea18a7b7c.png">



参考：https://segmentfault.com/a/1190000023637649

