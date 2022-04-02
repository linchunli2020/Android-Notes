***一、Service和IntentServic定义：***

Service既不是线程，也不是进程，而是依附于主线程的一个组件。Service用于在后台处理一些逻辑业务，不需要和用户进行交互。

IntentService继承于Service，用于处理异步请求，调用startService（intent）方法后，将请求通过intent传递给intentService，intentService在onCreate方法中构建一个HandlerThread的子线程，用于处理传递过来的请求。通过HandlerThread单独开启一个线程来依次处理所有Intent请求对象所对应的任务。这样以免事务处理阻塞主线程（ＡＮＲ）。执行完所有Intent请求对象所对应的工作之后，如果没有新的Intent请求达到，则自动停止Service；否则执行下一个Intent请求所对应的任务。

IntentService在处理事务时，还是采用的Handler方式，创建一个名叫ServiceHandler的内部Handler，并把它直接绑定到HandlerThread所对应的子线程。 ServiceHandler把处理一个intent所对应的事务都封装到叫做onHandleIntent的虚函数；因此我们直接实现虚函数onHandleIntent，再在里面根据Intent的不同进行不同的事务处理就可以了。另外，IntentService默认实现了Onbind（）方法，返回值为null。

***二、使用IntentService必须实现的函数***

1、参数为空的构造函数：然后再在其中调用super("name")这种形式的构造函数。因为Service的实例化是系统来完成的，而且系统是用参数为空的构造函数来实例化Service的

      public myIntentService() {

                   super("myIntentService");

                  // 注意构造函数参数为空，这个字符串就是worker thread的名字

      }

2、实现函数onHandleIntent：在里面根据Intent的不同进行不同的事务处理。

好处：处理异步请求的时候可以减少写代码的工作量，比较轻松地实现项目的需求。

***三、IntentService与Service的区别***

Service不是独立的进程，也不是独立的线程，它是依赖于应用程序的主线程的，不建议在Service中编写耗时的逻辑和操作，否则会引起ANR。

IntentService 它创建了一个独立的工作线程来处理所有的通过onStartCommand()传递给服务的intents（把intent插入到工作队列中）。通过工作队列把intent逐个发送给onHandleIntent()。

不需要主动调用stopSelft()来结束服务。因为，在所有的intent被处理完后，系统会自动关闭服务。

默认实现的onBind()返回null。


参考：

https://www.jianshu.com/p/c0fa9923d44d
