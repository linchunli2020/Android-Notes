***知识点：***

***Android中如何处理耗时操作？***
--------------------------------------------------------------------------------
1）通过View.post(Runnable)方法；

      view.post(new Runnable() { //使用View.post(Runnable)进行组 件设置 
        @Override public void run() { 
          //在这里进行UI操作，将结果显示在界面上 
          btn.setText("计数:"+sum) 
      )};

2）通过runOnUIThread(Runnable)方法。

      runOnUIThread（
        new Runnable(){ 
          @Override public void run() { 
            //在这里进行UI操作，将结果显示在界面上 
            btn.setText("计数:"+sum) 
       )};
       
       
***Android中执行耗时操作和返回主线程最好的实现方式?***
--------------------------------------------------------------------------------

  即为Android中实现异步任务的方式：
  （1）Handler类，在android中负责发送和处理消息，通过它可以实现其他支线线程与主线程之间的消息通讯。
  （2）AsyncTask类，Android从1.5版本之后引入，使用它就可以非常灵活方便地从子线程切换到UI线程。


***两种异步更新UI方法的对比：***

    ①Handler类：
              优点：结构清晰，功能定义明确，对于后台多个任务时，简单清晰；
              缺点：在单个后台异步处理时，显得代码过多，结构过于复杂（相对性）；
    ②AsyncTask类：
              优点：简单快捷过程可控的轻量级异步类；
              缺点：在使用多个异步操作和并需要UI变更时，就变得复杂起来；
              总结：二者各有优劣，有各自对应的开发场景，开发者都需要掌握。
              

***Android 的消息机制  Handler***
--------------------------------------------------------------------------------

Android的消息机制主要是指 Handler 的运行机制，Handler的运行机制需要底层的 MessageQueen 和 Looper 的支撑。

***主线程（UI 线程）***
    定义：当程序第一次启动时，Android 会同时启动一条主线程（Main Thread）
    作用：主线程主要负责处理与 UI 相关的事件

***Message（消息）***
    定义：Handler 接收和处理的消息对象（Bean 对象）
    作用：通信时相关信息的存放和传递

***MessageQueue（消息队列）***
    定义：是指消息队列，它的内部存储了一组信息，以队列的形式对外提供插入和删除的工作。虽然叫做消息队列，但是它的内部存储结构不是真正的队列，而是采用单链表的数据结构来存储消息队列。
    作用：用来存放通过 Handler 发过来的 Message，按照先进先出执行

***Handler（处理者）***
    定义：Message 的主要处理者,创建的时候会采用当前线程的 Looper 来构造消息循环系统，那么 Handler 内部通过 ThreadLocal 获取到当前线程的 Looper。
    作用：
        负责发送 Message 到消息队列
        处理 Looper 分派过来的 Message

***Looper（循环器）***
    定义：扮演 MessageQueue 和 Handler 之间桥梁的角色。
         Looper是指消息循环，每个线程只能有一个Looper，由于MessageQueen只是一个消息的存储单元，它不能去处理消息，而 Looper 填补了这个功能，Looper会以无限循环的形式去
         查找是否有新消息，如果有的话就处理消息，否则就一直等待着。
    作用：主要作用是将 一个任务切换 到 指定的线程 中去执行。
        消息循环：循环取出 Message Queue 的 Message 
        消息派发：将取出的 Message 交付给相应的 Handler

***ThreadLocal***
    定义：线程内部的数据存储
    作用：负责存储和获取本线程的 Looper，可以在不同的线程中互不干扰地存储并提供数据，通过 ThreadLocal 可以获取每个线程的 Looper。


Android中规定访问 UI 只能在主线程中进行，如果在子线程中访问UI，那么程序会抛出异常，这是因为 ViewRootImpl 的 checkThread 方法会进行验证。

Android 不建议在主线程中进行耗时的操作否则会导致程序无法响应即 ANR。

考虑这种情况，我们可以在子线程处理耗时操作，通过 Handler 切换线程在主线程中访问 UI。


***Message、Handler、MessageQueen、Looper 的之间的关系？***
--------------------------------------------------------------------------------
    首先，是这个 MessageQueen，MessageQueen 是一个消息队列，它可以存储 Handler 发送过来的消息，其内部提供了进队和出队的方法来管理这个消息
    队列，其出队和进队的原理是采用单链表的数据结构进行插入和删除的，即 enqueueMessage() 方法和 next()方法。
    这里提到的 Message，其实就是一个Bean 对象，里面的属性用来记录 Message 的各种信息。
    然后，是这个 Looper，Looper 是一个循环器，它可以循环的取出 MessageQueen 中的 Message，其内部提供了 Looper 的初始化和循环出去Message 的方法，即 prepare() 方法和 loop() 方法。在 prepare()方法中，Looper会关联一个 MessageQueen，而且将 Looper 存进一个 ThreadLocal 中，在loop()方法中，通过 ThreadLocal 取出 Looper，使用MessageQueen 的next() 方法取出 Message 后，判断 Message 是否为空，如果是则 Looper 阻塞，如果不是，则通过 dispatchMessage() 方法分发该 Message 到 Handler 中，而 Handler 执行 handlerMessage() 方法，由于 handlerMessage() 方法是个空方法，这也是为什么需要在 Handler 中重写 handlerMessage()方法的原因。
    这里要注意的是Looper 只能在一个线程中只能存在一个。
    这里提到的ThreadLocal，其实就是一个对象，用来在不同线程中存放对应线程的 Looper。
    最后，是这个Handler，Handler 是Looper 和MessageQueen 的桥梁，Handler 内部提供了发送 Message 的一系列方法，最终会通过 MessageQueen 的
    enqueueMessage()方法将 Message 存进 MessageQueen 中。我们平时可以直接在主线程中使用 Handler，那是因为在应用程序启动时，在入口的 main 方
    法中已经默认为我们创建好了 Looper。

    线程的转换由 Looper 完成，handleMessage() 所在线程由 Looper.loop() 调用者所在线程决定


***Handler.postDelayed()****
--------------------------------------------------------------------------------
问：如果Message会阻塞MessageQueue的话，那么先postDelay10秒一个Runnable A，消息队列会一直阻塞，然后我再post一个Runnable B， B岂不是会等A执行完了再执行？
正常使用时显然不是这样的，整个调用流程如下所示：

    1. postDelay()一个10秒钟的Runnable A、消息进队，MessageQueue调用nativePollOnce()阻塞，Looper阻塞；
    2. 紧接着post()一个Runnable B、消息进队，判断现在A时间还没到、正在阻塞，把B插入消息队列的头部（A的前面），然后调用nativeWake()方法唤醒线程；
    3. MessageQueue.next()方法被唤醒后，重新开始读取消息链表，第一个消息B无延时，直接返回给Looper； 4. Looper处理完这个消息再次调用next()方法，MessageQueue继续
       读取消息链表，第二个消息A还没到时间，计算一下剩余时间（假如还剩9秒）继续调用nativePollOnce()阻塞；
    5. 直到阻塞时间到或者下一次有Message进队；
    这样，基本上就能保证Handler.postDelayed()发布的消息能在相对精
    确的时间被传递给Looper进行处理而又不会阻塞队列了。
    
    
    另外，这里在阅读原文的基础上添加一点思考内容：
    MessageQueue会根据post delay的时间排序放入到链表中，链表头的时间小，尾部时间最大。
    因此能保证时间Delay最长的不会block住时间短的。当每次post message的时候会进入到MessageQueue的next()方法，会根据其delay时间和链表头的比较，
    如果更短则，放入链表头，并且看时间是否有delay，如果有，则block，等待时间到来唤醒执行，否则将唤醒立即执行。
    所以handler.postDelay并不是先等待一定的时间再放入到MessageQueue中，而是直接进入MessageQueue，以MessageQueue的时间顺序排列和唤醒的方式结合实现的。
    使用后者的方式，我认为是集中式的统一管理了所有message，而如果像前者的话，有多少个delaymessage，则需要起多少个定时器。前者由于有了排序，
    而且保存的每个message的执行时间，因此只需一个定时器按顺序next即可。
    
***Handler postDelayed（） 的原理***

    1、消息是通过MessageQueen中的enqueueMessage()方法加入消息队列中的，并且它在放入中就进行好排序，链表头的延迟时间小，尾部延迟时间最大；
    2、Looper.loop()通过MessageQueue中的next()去取消息；
    3、next()中如果当前链表头部消息是延迟消息，则根据延迟时间进行消息队列会阻塞，不返回给Looper message，知道时间到了，返回给message；
    4、如果在阻塞中有新的消息插入到链表头部则唤醒线程；
    5、Looper将新消息交给回调给handler中的handleMessage后，继续调用MessageQueen的next()方法，如果刚刚的延迟消息还是时间未到，则计算时间继续阻塞；
    总结：
    handler.postDelay() 的实现 是通过 MessageQueue 中执行时间顺序排列，消息队列阻塞，和唤醒的方式结合实现的。
    如果真的是通过延迟将消息放入到 MessageQueen 中，那放入多个延迟消息就要维护多个定时器；

参考：https://www.jianshu.com/p/44b322dfc040

***Handler 机制***
--------------------------------------------------------------------------------
Android 中主线程也叫 UI 线程，那么从名字上我们也知道主线程主要是用来创建、更新 UI 的，而其他耗时操作，比如网络访问，或者文件处理，多媒体
处理等都需要在子线程中操作，之所以在子线程中操作是为了保证 UI 的流畅程度，手机显示的刷新频率是 60Hz，也就是一秒钟刷新 60 次，每 16.67 毫秒刷
新一次，为了不丢帧，那么主线程处理代码最好不要超过 16 毫秒。当子线程处理完数据后，为了防止 UI 处理逻辑的混乱，Android 只允许主线程修改 UI，那
么这时候就需要 Handler 来充当子线程和主线程之间的桥梁了。

我们通常将 Handler声明在Activity中，然后覆写Handler中的handleMessage 方 法 , 当子线程调用handler.sendMessage()方法后handleMessage方法就会在主线程中执行。
这里面除了 Handler、Message 外还有隐藏的 Looper 和 MessageQueue对象。
在主线程中 Android 默认已经调用了 Looper.prepare()方法，调用该方法的目的是在 Looper 中创建MessageQueue 成员变量并把Looper 对象绑定到当前线程中。
当调用 Handler 的 sendMessage（对象）方法的时候就将 Message 对象添加到了 Looper 创建的 MessageQueue 队列中，同时给 Message 指定了 target 对象，
其实这个 target 对象就是 Handler 对象。主线程默认执行了 Looper.looper（）方法，该方法从 Looper 的成员变量 MessageQueue 中取出 Message，
然后调用 Message 的 target 对象的 handleMessage()方法。这样就完成了整个消息机制。


***Handler 原理***
--------------------------------------------------------------------------------
Handler，Message，looper 和 MessageQueue 构成了安卓的消息机制，handler 创建后可以通过 sendMessage 将消息加入消息队列，然后 looper 不断的将消息从
MessageQueue 中取出来，回调到 Hander 的 handleMessage 方法，从而实现线程的通信。
从两种情况来说
第一在 UI 线程创建 Handler,此时我们不需要手动开启 looper，因为在应用启动时，在 ActivityThread 的 main 方法中就创建了一个当前主线程的looper，并开启了消息队列，
消息队列是一个无限循环；

**为什么无限循环不会ANR?**
因为可以说，应用的整个生命周期就是运行在这个消息循环中的，安卓是由事件驱动的，Looper.loop 不断的接收处理事件，每一个点击触摸或者 Activity 每一个生命
周期都是在 Looper.loop 的控制之下的，looper.loop 一旦结束，应用程序的生命周期也就结束了。

**我们可以想想什么情况下会发生 ANR？**

    第一，事件没有得到处理
    第二，事件正在处理，但是没有及时完成，而对事件进行处理的就是 looper，所以只能说事件的处理如果阻塞会导致 ANR，而不能说 looper 的无限循环会 ANR;
    另一种情况就是在子线程创建 Handler,此时由于这个线程中没有默认开启的消息队列，所以我们需要手动调用 looper.prepare(),并通过 looper.loop 开启消息；
    主线程 Looper 从消息队列读取消息，当读完所有消息时，主线程阻塞。
    
子线程往消息队列发送消息，并且往管道文件写数据，主线程即被唤醒，从管道文件读取数据，主线程被唤醒只是为了读取消息，当消息读取完毕，再次睡眠。
因此 loop 的循环并不会对 CPU 性能有过多的消耗。


***为什么不允许在子线程中访问UI呢***
--------------------------------------------------------------------------------

这是因为Android的 UI 控件线程不是安全的，如果在多线程情况下并发访问可能会导致UI控件处于不可预期的状态。

加上锁机制缺点有两个：加上锁机制会让UI访问逻辑变得复杂，锁机制会降低UI访问效率，会阻塞某些线程的执行。
每一个线程只能有一个Looper，如果当前线程没有 Looper 系统会报错，解决办法是为当前线程创建 Looper 或者在一个有Looper的线程中创建Handler。

Handler的post方法将一个Runnable投递到Handler内部的Looper中去处理，也可以通过Handler的 send 方法发送一个消息，这个消息同样会在Looper中去处理。
其实post方法最终也是通过send方法来完成的。

当Handler的 send 方法被调用时，它会调用 MessageQueen 的 enqueueMessage 方法将这个消息放入消息队列中，然后Looper发现有新消息到来时，就会处理这个消息，
最终消息中的Runnable或者Handler的handleMessage方法就会被调用。


***Handler为什么会导致内存溢出***
--------------------------------------------------------------------------------
解决这个问题的思路就是：使用静态内部类并继承handler
因为静态的内部类不会持有外部类的引用，所以不会导致外部类实例的内存泄漏。当你需要在静态内部类中调用外部的activity时，可以使用弱引用（WeakReference）来处理。

参考：https://blog.csdn.net/javazejian/article/details/50839443


