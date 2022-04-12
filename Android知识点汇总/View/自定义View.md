UI***自定义View的绘制主要有五个方法：onMeasure()，onLayout(), onDraw(), dispatchDraw(),drawChild()***

        1.ViewGroup包含这五个方法，而View只包含onMeasure()，onLayout(), onDraw()三个方法，不包含dispatchDraw(),drawChild()。
        
        2.绘制流程：onMeasure（测量）——》onLayout（布局）——》onDraw（绘制）。
        
        3.绘制按照视图树的顺序执行，视图绘制时会先绘制子控件。如果视图的背景可见，视图会在调用onDraw()之前调用drawBackGround()绘制背景。
          强制重绘，可以使用invalidate();
        
        4.如果发生视图的尺寸变化，则该视图会调用requestLayout()，向父控件请求再次布局。如果发生视图的外观变化，则该视图会调用invalidate()，强制重绘。
          如果requestLayout()或invalidate()有一个被调用，框架会对视图树进行相关的测量、布局和绘制。
          注意：视图树是单线程操作，直接调用其它视图的方法必须要在UI线程里。跨线程的操作必须使用Handler。
        
        5.onLayout()：对于View来说，onLayout()只是一个空实现；而对于ViewGroup来说，onLayout()使用了关键字abstract的修饰，要求其子类必须重载该方法，
          目的就是安排其children在父视图的具体位置。
          
        6.draw过程：drawBackground()绘制背景——> onDraw()对View的内容进行绘制——> dispatchDraw()对当前View的所有子View进行绘制
          ——> onDrawScrollBars()对View的滚动条进行绘制。
 
 <img width="472" alt="image" src="https://user-images.githubusercontent.com/67937122/162101922-de4f01df-3286-4527-b6b2-7bb902249a14.png">


***方法说明：***

**1.onMeasure(int widthMeasureSpec, int heightMeasureSpec)：**
  
  用于计算自己及所有子对象的大小。这个方法是所有View、ViewGroup及其派生类都具有的方法。自定义控件时，可以重载该方法，重新计算所有对象的大小。 
  MeasureSpec包含了测量的模式和测量的大小，通过MeasureSpec.getMode()获取测量模式，通过MeasureSpec.getSize()获取测量大小。
  mode共有三种情况： 分别为
      **MeasureSpec.UNSPECIFIED（ View想多大就多大）**,
      **MeasureSpec.EXACTLY（默认模式，精确值模式：将layout_width或layout_height属性指定为具体数值或者match_parent。）**,
      **MeasureSpec.AT_MOST（ 最大值模式：将layout_width或layout_height指定为wrap_content。）**。
      
**2.onLayout(boolean changed, int left, int top, int right, int bottom)：**
  
  布局发生变化时调用此方法。这个方法是所有View、ViewGroup及其派生类都具有的方法。自定义控件时，可以重载该方法，在布局发生改变时实现特效等定制处理。

**3.onDraw(Canvas canvas)：**

UI绘制最重要的方法，用于UI重绘。
这个方法是所有View、ViewGroup及其派生类都具有的方法。自定义控件时，可以重载该方法，并在内容基于canvas绘制自定义的图形、图像效果。

**4.dispatchDraw(Canvas canvas)：**

ViewGroup及其派生类具有的方法，主要用于控制子View的绘制分发。自定义控件时，重载该方法可以改变子View的绘制，进而实现一些复杂的视效。

**5.drawChild(Canvas canvas, View child, long drawingTime)：**

ViewGroup及其派生类具有的方法，用于直接绘制具体的子View。自定义控件时，重载该方法可以直接绘制具体的子View。 

 
***MeasureSpec约束是由父控件传递给子控件的，有哪几种模式？***

      /**
       * 父控件不强加任何约束给子控件，它可以是它想要任何大小
       */
      public static final int UNSPECIFIED = 0 << MODE_SHIFT;
      /**
       * 父控件已为子控件确定了一个确切的大小，孩子将被给予这些界限，不管子控件自己希望的是多大
       */
      public static final int EXACTLY = 1 << MODE_SHIFT;
      /**
       * 父控件会给子控件尽可能大的尺寸
       */
      public static final int AT_MOST = 2 << MODE_SHIFT;
      
——————————————————————————————————————————————————————————————————————————————————      
***自定义view注意事项***

**onDraw中如果定义一些实例对象 会有什么问题吗？**

  ● 在View的onDraw方法中不要创建太多的临时对象，也就是new出来的对象。因为onDraw方法会被频繁调用，如果有大量的临时对象，就会引起内存抖动，影响View的效果
  
  ● View中如果有线程或者动画，需要及时停止，在View的onDetachedFromWindow方法可以停止线程和动画，因为当View被remove或是包含此View的Activity退出时，
     就会调用View的onDetachedFromWindow方法。如果不处理的话很可能会导致内存泄漏。
     
  ● 尽量不要在View中使用Handler，View中已经提供了post系列方法，完全可以替代Handler的作用。

——————————————————————————————————————————————————————————————————————————————————
***自定义view同时触发onTouch onClick解决方案***

onTouch的return值为true时不能响应onClick事件，设置为false后，就会同时触发两个事件，然后就在网上找解决办法，

有的说记录坐标，根据结束坐标的位置和开始位置的差值来判断，有的说用什么线程来判断。最后在技术群里一个朋友给出了思路，然后成功解决了。

办法其实很简单：

定义一个boolean的 全局变量isMove= false，

然后在onTouch方法里的MotionEvent.ACTION_MOVE:里边设置isMove =true;

在MotionEvent.ACTION_UP:判断isMove的值 

        if (isMove == false) {
           //对click事件的处理} 
        else if (isMove == true){
           //对onTouch事件的处理，我仅仅是更新坐标
        } 
        
记得一定要设置在break之前再次设置isMove =false;
      
      
 转载：
 https://blog.csdn.net/qq941263013/article/details/82500145
 
 https://juejin.cn/post/6844903648758071309
 
 https://www.cxyzjd.com/article/weixin_40417131/79699135
 
https://juejin.cn/post/6844904013763182605

 
