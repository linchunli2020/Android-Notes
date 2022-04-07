UI***自定义View的绘制主要有五个方法：onMeasure()，onLayout(), onDraw(), dispatchDraw(),drawChild()***

        1.ViewGroup包含这五个方法，而View只包含onMeasure()，onLayout(), onDraw()三个方法，不包含dispatchDraw(),drawChild()。
        
        2.绘制流程：onMeasure（测量）——》onLayout（布局）——》onDraw（绘制）。
        
        3.绘制按照视图树的顺序执行，视图绘制时会先绘制子控件。如果视图的背景可见，视图会在调用onDraw()之前调用drawBackGround()绘制背景。
          强制重绘，可以使用invalidate();
        
        4.如果发生视图的尺寸变化，则该视图会调用requestLayou()，向父控件请求再次布局。如果发生视图的外观变化，则该视图会调用invalidate()，强制重绘。
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
      
      
 转载：https://blog.csdn.net/qq941263013/article/details/82500145

 
