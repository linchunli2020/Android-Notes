
（1）点击桌面应用图标，Launcher进程会把启动Activity的请求以Binder的方式发送给AMS
![image](https://user-images.githubusercontent.com/67937122/161364135-c4fa719a-8cd8-4c0d-8961-9099b19eb6cf.png)


（2）AMS接受到请求后，会交给ActivityStarter处理Intent和Flag相关信息，然后交给ActivityStack处理Activity进栈相关流程，并已socket方式请求zygote进程fork新进程

![image](https://user-images.githubusercontent.com/67937122/161364164-5ee821ae-f2b3-4304-a7eb-3ae4e7a29013.png)


（3）zygote进程接收到创建新进程的请求后，fork出新进程
![image](https://user-images.githubusercontent.com/67937122/161364199-660c1c41-96ad-4163-a584-908f42d2fd1f.png)



（4）在新进程中，创建ActivityThread对象，新进程就是应用的主线程，在主线程中开启Looper消息循环，开始创建Activity
![image](https://user-images.githubusercontent.com/67937122/161364276-4a254512-b45f-4523-9e18-d64e25e605e0.png)


（5）ActivityThread通过ClassLoader去加载Activity，创建Activity实例，并回调Activity的oncreate方法，完成Activity的启动。


1. Launcher进程请求AMS
2. AMS发送创建应用进程请求
3. Zygote进程接受请求并孵化应用进程
4. 应用进程启动ActivityThread
5. 应用进程绑定到AMS
6. AMS发送启动Activity的请求
7. ActivityThread的Handler处理启动Activity的请求


链接：https://juejin.cn/post/6844903959589552142
