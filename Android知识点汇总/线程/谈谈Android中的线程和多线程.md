谈谈Android中的线程和多线程？

（一）线程

Android中的线程

在Android中，当应用启动的时候，系统会给应用分配一个进程（大部分应用都是单进程的，不过也可以用过设置来使不用组件运行在不同的进程中），在创建进程的同时会创建一个线程，应用的大部分操作都会在这个线程中运行，所以称为主线程，同时所有的UI控件相关的操作也要求在这个线程中操作，所以也称为UI线程.

UI线程和工作线程

因为所有的UI控件的操作都在UI线程中执行, 如果在UI线程中执行耗时操作, 例如网络请求等, 就会阻塞UI线程, 导致系统报ANR(Application Not Response)错误. 因此对于耗时操作需要创建工作线程来执行而不能直接在UI线程中执行. 这样就需要在应用中使用多线程, 但是Android提供的UI工具包并不是线程安全的, 也就是说不能直接在工作线程中访问UI控件, 否则会导致不能预测的问题, 因此需要额外的机制来进行线程交互, 主要是让其他线程可以访问UI线程.

线程交互 - Handler机制

在Android当中, 工作线程主要通过Handler机制来访问UI线程. 当然还有一些封装好的类例如AsyncTask可以使用, 但是本质仍是使用Handler.
Handler机制主要由4部分组成, Looper, 消息队列, 消息类和Handler组成, 其中Looper和消息队列是和线程绑定的, 每个线程只会有一个Looper和一个消息队列, 当Looper启动时, 它会无限循环尝试从消息队列中获取消息实例, 如果没有消息则会阻塞等待. 当Handler发送消息时会把消息实例放入消息队列中, Looper从中取得消息实例然后就会调用Handler的相关方法, 因为Looper是线程绑定的, 如果绑定的是UI线程, 那么此时Handler的方法就会在UI线程中得到执行, 线程间就是这样进行交互的。

（二）线程池相关

线程池的作用

1.线程的创建和销毁的开销是巨大的，而通过线程池的使用大大减少了这些不必要的开销，减少了内存的开销，提升了线程执行的速度。

2.控制线程池的并发数可以有效的避免大量的线程池争夺CPU资源而造成堵塞。

注意：并发与并行的区别：

并发：在某个时间段内，多个程序都处在执行和执行完毕之间；但在一个时间点上只有一个程序在运行。

并行：在某个时间段里，每个程序按照自己独立异步的速度执行，程序之间互不干扰。

3.线程池可以对线程进行管理，可以提供定时、定期、单线程、并发数控制等功能。比如通过ScheduledThreadPool线程池来执行S秒后，每隔N秒执行一次的任务。

线程池的种类：

1.ThreadPoolExecutor

2.FixedThreadPool：有固定数量线程的线程池，其corePoolSize=maximumPoolSize，且keepAliveTime为0，适合线程稳定的场所。

3.SingleThreadPoo：固定数量线程的线程池，且数量为一，从数学的角度来看SingleThreadPool应该属于FixedThreadPool的子集。其corePoolSize=maximumPoolSize=1,且keepAliveTime为                  0，适合线程同步操作的场所。

4.CachedThreadPool：储存的线程池，容量很大，他的corePoolSize=0，maximumPoolSize=Integer.MAX_VALUE(2^32-1一个很大的数字)。

5.ScheduledThreadPool:计划的线程池，是一个具有定时定期执行任务功能的线程池。

ThreadPoolExecutor的参数详解：
public ThreadPoolExecutor(int corePoolSize,
                           int maximumPoolSize,
                           long keepAliveTime,
                           TimeUnit unit,
                           BlockingQueue<Runnable>
                           workQueue,
                           ThreadFactory
                           threadFactory,
                           RejectedExecutionHandler
                           handler)
   
这里是7个参数(我们在开发中用的更多的是5个参数的构造方法)
   
corePoolSize：线程池中核心线程的数量
   
maximumPoolSize：线程池中最大线程数量
   
keepAliveTime：非核心线程的超时时长，当系统中非核心线程闲置时间超过keepAliveTime之后，则会被回收。如果ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，则该参数也表              示核心线程的超时时长
   
unit：第三个参数的单位，有纳秒、微秒、毫秒、秒、分、时、天等
   
workQueue：线程池中的任务队列，该队列主要用来存储已经被提交但是尚未执行的任务。存储在这里的任务是由ThreadPoolExecutor的execute方法提交来的。
   
threadFactory：线程工厂，为线程池提供创建新线程的功能，这个我们一般使用默认即可
   
handler：拒绝策略，当线程无法执行新任务时（一般是由于线程池中的线程数量已经达到最大数或者线程池关闭导致的），默认情况下，当线程池无法处理新线程时，会抛RejectedExecutionException。

这7个参数中，平常最多用到的是corePoolSize、maximumPoolSize、keepAliveTime、unit、workQueue。
   
maximumPoolSize(最大线程数) = corePoolSize(核心线程数) +noCorePoolSize(非核心线程数)；
   
（1）当currentSize<corePoolSize时，没什么好说的，直接启动一个核心线程并执行任务。
                                                      
（2）当currentSize>=corePoolSize、并且workQueue未满时，添加进来的任务会被安排到workQueue中等待执行。
   
（3）当workQueue已满，但是currentSize<maximumPoolSize时，会立即开启一个非核心线程来执行任务。
                                                                  
（4）当currentSize>=corePoolSize、workQueue已满、并且currentSize>maximumPoolSize时，调用handler默认抛出RejectExecutionExpection异常。

线程池的实现原理：
   
提交一个任务到线程池中，线程池的处理流程如下：
   
1.判断线程池里的核心线程是否都在执行任务，如果不是（核心线程空闲或者还有核心线程没有被创建）则创建一个新的工作线程来执行任务。如果核心线程都在执行任务，则进入下个流程。
   
2.线程池判断工作队列是否已满，如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。
   
3.判断线程池里的线程是否都处于工作状态，如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。
   
   
   
   
   
   
   
   
   
   
   
   
