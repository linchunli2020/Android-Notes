
总结：

（1）点击桌面应用图标，Launcher进程会把启动Activity的请求以Binder的方式发送给AMS

（2）AMS接受到请求后，会交给ActivityStarter处理Intent和Flag相关信息，然后交给ActivityStack处理Activity进栈相关流程，并已socket方式请求zygote进程fork新进程

（3）zygote进程接收到创建新进程的请求后，fork出新进程

（4）在新进程就是应用进程，在应用进程中启动ActivityThread对象，

（5）应用进程绑定到AMS

（6）AMS发送启动Activity的请求

（7）ActivityThread的Handler处理启动Activity的请求

具体分析如下链接：

链接：https://juejin.cn/post/6844903959589552142
