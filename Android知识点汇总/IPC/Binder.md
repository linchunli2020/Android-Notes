Binder 是 Android 中的一个类，它实现了 IBinder 接口。

从 IPC 角度来说，Binder 是 Android 中的一种跨进程通信方式；

从 Android Framework 角度来说，Binder 是 ServiceManager 连接各种 Manager 和相应 ManagerService 的桥梁；

从Android应用层来说，Binder 是客户端和服务端进行通信的媒介，当 bindService 的时候，服务端会返回一个包含了服务端业务调用的Binder对象，
通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。

***Binder 运行机制：***

一种基于 Client/Service 架构，客户端 Client 要调用远程进程函数时，只需要把数据写入到 Parcel，再调用所持有Binder 引用的 transact 函数，
transact 函数执行过程中会把参数，标识符等数据放入到 Client 的共享内存中，Binder 驱动从 Client 的共享内存中读取数据，根据这些数据找到远程进程中的共享内存，
把数据拷贝到远程进程中的共享内存中，并通知远程进程执行 onTransact 函数。
远程进程执行完成后将结果写入到共享内存里，Binder 驱动再将共享内存中的结果拷贝到客户端的共享内存内并唤醒客户端线程。
