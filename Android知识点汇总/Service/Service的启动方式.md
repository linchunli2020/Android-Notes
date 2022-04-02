
***Service 启动方式有两种，一种是通过startService()方式进行启动，另一种是通过bindService()方式进行启动。不同的启动方式他们的生命周期是不一样。***

***（1）startService***

通过startService()这种方式启动的service，生命周期是这样：调用startService() --> onCreate()--> onStartConmon()--> onDestroy()。

这种方式启动的话，需要注意一下几个问题，

      第一：当我们通过startService被调用以后，多次在调用startService(),onCreate()方法也只会被调用一次，而onStartConmon()会被多次调用当我们调用stopService()的时候，
           onDestroy()就会被调用，从而销毁服务。

      第二：当我们通过startService启动时候，通过intent传值，在onStartConmon()方法中获取值的时候，一定要先判断intent是否为null。

***（2）bindService***

通过bindService()方式进行绑定，这种方式绑定service，生命周期走法：bindService-->onCreate()-->onBind()-->unBind()-->onDestroy() 
bingservice 这种方式进行启动service好处是更加便利activity中操作service，比如加入service中有几个方法，a,b ，如果要在activity中调用，
在需要在activity获取ServiceConnection对象，通过ServiceConnection来获取service中内部类的类对象，然后通过这个类对象就可以调用类中的方法，当然这个类需要继承Binder对象
