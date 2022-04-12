当我们需要根据某个条件控制某个View的显示或者隐藏的时候，通常是把可能用到的View都写在布局上，然后设置可见性为View.GONE或View.InVisible ，之后在代码中根据条件动态控制可见性。
虽然操作简单，但是耗费资源，因为即便该view不可见，仍会被父窗体绘制，仍会创建对象，仍会被实例化，仍会被设置属性。

而android.view.ViewStub，是一个大小为0 ，默认不可见的控件，只有给他设置成了View.Visible 或调用了它的 inflate() 之后才会填充布局资源，也就是说占用资源少。
所以，推荐使用viewStub。

链接：https://www.jianshu.com/p/175096cd89ac
