Activity发生了内存泄漏，
从引用路径来看，是被匿名内部类的实例mHandler持有引用了，
而Handler的引用是被Message持有了，Message引用是被MessageQueue持有了...

结合我们所学的Handler知识和这次引用路径分析，这次内存泄漏完整的引用链应该是：

主线程 —> threadlocal —> Looper —> MessageQueue —> Message —> Handler —> Activity

