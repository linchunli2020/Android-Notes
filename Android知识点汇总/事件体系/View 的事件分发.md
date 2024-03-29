

在 view 中事件分发十分重要，了解他的原理，对我们理解View 和解决滑动冲突都十分重要。所有的Touch事件都封装到 MotionEvent里面；

事件处理包括三种情况，分别为：

      传递—-dispatchTouchEvent()函数、

      拦截——onInterceptTouchEvent()函数、

      消费—-onTouchEvent()函数和OnTouchListener；

事件类型分为 ACTION_DOWN, ACTION_UP, ACTION_MOVE , ACTION_POINTER_DOWN, ACTION_POINTER_UP , ACTION_CANCEL 等，每个事件都是以 ACTION_DOWN 开始 ACTION_UP 结束。

用下面伪代码表示事件分发过程及其关系：

      //事件分发
      public boolean dispatchTouchEvent(MotionEvent event) {
          boolean consume = false;
          //是否被拦截
          if (onInterceptTouchEvent(event))
          {
              //被拦截，处理事件
              consume = onTouchEvent(event);
          } else {
              //未被拦截，向下分发
              consume = childView.dispatchTouchEvent(event);
          }
          return consume;
      }

事件传递的基本流程：

        事件都是从Activity.dispatchTouchEvent()开始传递；

        事件由父View传递给子View，ViewGroup可以通过onInterceptTouchEvent()方法对事件拦截，停止其向子view传递；

        如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent()函数；

        如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来，也就是说ACTION_DOWN必须返回true，之后的事件才会传递进来；

        OnTouchListener优先于onTouchEvent()对事件进行消费。


1.1 Activity对点击事件的分发过程

点击事件用MotionEvent来表示，当一个点击操作发生时，事件最先传递给当前Activity，由Ativity的dispathTouchEvnent来进行事件派发，具体的工作是由Activity内部的Window来完成的。
Window会将事件传递给DecorView，DecorView一般就是当前界面的底层容器（即setContentView所设置的VIew的父容器），通过Activity.getWindow.getDecorView()可以获得。
先从Activity的dispatchTouchEvent开始分析：

      public boolean dispatchTouchEvent(MotionEvent ev){
        if(ev.getAction() == MotionEvent.ACTION_DOWN){
          onUserInteraction();
        }
        if(getWindow().superDispatchTouchEvent(ev)){
          return true;
        }
        return onTouchEvent(ev);
      }


分析上面的代码。首先事件开始交给Activit所附属的Window进行分发，
如果返回true，整个事件循环就结束了，
返回false意味着事件没人处理，所有View的onTouchEvent都返回了false，那么Activity的onTouchEvent就会被调用。

接下来看window是怎么将事件传递给ViewGroup的。

通过源码可以知道，Window是个抽象类，而Window的superDispatchTouchEvent方法也是个抽象方法，因此必须找到Window的实现类才行。

WIndow的实现类其实是PhoneWIndow，在PhoneVIew中的superDIspatchTouchEvent中将事件传递给了DecorView。源码如下所示：

      源码：
      PhoneWindow#superDispatchTouchEvnent
      public boolean superDispatchTouchEvent(MotionEvent event){
            return mDecor.superDispatchTouchEvent(event);
      }
      
目前事件传递给了DecorView，由于DecorVIew继承自FragmentLayout且是父VIew，所以最终事件会传递给View。从这里开始，事件已经传递给顶级VIew了。

1.2 顶级View对点击事件的分发过程

点击事件到达View后，会调用VIewGroup的dispatchTouchEvent方法，然后的逻辑是这样的：如果顶级VIewGroup拦截事件即OnInterceptTouchEvent返回true，则事件交给VIewGroup处理，
这时如果ViewGroup的mOnTouchLIstener被设置，则OnTouch会被调用，否则OnTouchEvnent会被调用。也就是说，如果提供的话，onTouch会屏蔽掉onTouchEvent。在onTouchEvent中，
如果设置了mOnClickListener，则onClick会被调用。如果顶级ViewGroup不拦截事件，则事件会传递给它所在的点击事件链上的子View，这时子View的dispatchTouchEvent会被调用。
到此为止，事件已经从顶级View传递给下一层View，接下来的传递过程和顶级VIew是一致的，如此循环，完成整个事件的分发。

1.3 View对点击事件的处理过程

View对点击事件的处理过程比较简单，因为VIew（这里不包含viewGroup）是一个单独的元素，它没有子元素因此无法向下传递事件，所以它只能自己处理事件。

在dispatchTouchEvent方法中，首先会判断有没有设置onTouchListener，如果onTouchListener中的onTouch方法返回true，那么onTouchEvnent就不会被调用，
可见onTouchListener的优先级高于onTouchEvent，这样做的好处是方便在外界处理点击事件。

在view的onTouchEvent方法中，可以看到，只要VIew的CLICKABLE和LONG_CLICKABLE有一个为true，那么它就会消耗这个事件，即onTouchEvent方法返回true，不管它是不是DISABLE状态。
然后就是当ACTION_UP事件发时，会出发perforClick方法，如果VIew设置了OnClickListener，那么performClick方法内部会调用它的onClick方法。





