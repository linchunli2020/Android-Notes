

目录

<img width="577" alt="image" src="https://user-images.githubusercontent.com/67937122/161183091-16317519-4084-4083-9d35-08c9488817e3.png">

1.基础概念

1.1 touch事件定义

什么是Touch事件？

一个Touch事件在用户点击屏幕（ACTION_DOWN）时产生，抬起手指（ACTION_UP）时结束，而Touch事件又被封装到MotionEvent当中。

1.2 事件分类

Touch事件总体可以分为以下几类。

事件类型说明
      ACTION_DOWN 手指按下
      ACTION_UP 手指抬起
      ACTION_MOVE 手势移动
      ACTION_POINTER_DOWN 多个手指按下
      ACTION_POINTER_UP 多个手指抬起
      ACTION_CANCEL 取消事件
      
获取Action最常见的方式就是使用MotionEvent的getAction()方法，getAction()方法可以获ACTION_DONW、ACTION_UP、ACTION_MOVE、以及ACTION_CANCEL等事件，
我们分析事件传递时基本也是分析这些事件。

ACTION_POINTER_DOWN和ACTION_POINTER_UP，则得和ACTION_MASK相与才能得到。


1.3 事件顺序

<img width="595" alt="image" src="https://user-images.githubusercontent.com/67937122/161183271-774f43a7-f2ba-41f1-9c06-5fea61b4dae2.png">

从事件定义也可以知道，一个事件总是以ACTION_DOWN作为开始，在手势移动过程中会重复产生多个ACTION_MOVE事件，用户操作结束事件的标志为ACTION_UP，而意外终止事件则会触发ACTION_CANCEL。

1.4 事件传递层级

<img width="315" alt="image" src="https://user-images.githubusercontent.com/67937122/161183939-9a0382c7-12a0-47b4-8cca-9f99c0e7d760.png">


事件传递时，总是会从最下层开始向上层传递，也就是说出发点为Activity,逐层向上传递。当上层View响应事件后，下层的View将不再会再响应。在后面的代码中我们会分析为什么会是这种现象。

做完准备工作后开始正式对事件传递流程进行分析，先分析最上层的View如何处理事件。

2. View的事件分发
2.1 实例分析

我们先看一下在我们的代码中事件会如何执行。自定义一个TestView

      public class TestView extends View {

          public TestView(Context context) {
              super(context);
          }

          public TestView(Context context, @Nullable AttributeSet attrs) {
              super(context, attrs);
          }

          public TestView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
              super(context, attrs, defStyleAttr);
          }

          @Override
          protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
              super.onMeasure(widthMeasureSpec, heightMeasureSpec);
          }

          @Override
          public boolean onTouchEvent(MotionEvent event) {
              Log.d("test", "View onTouchEvent"+ event.getAction());
              return super.onTouchEvent(event);
          }

          @Override
          public boolean dispatchTouchEvent(MotionEvent event) {
              Log.d("test", "View dispatchTouchEvent"+ event.getAction() );
              return super.dispatchTouchEvent(event);
          }

      }

在自定义View当中存在两个事件分发相关方法onTouchEvent以及dispatchTouchEvent,我们先关注一下现象，后文通过源码分析他们两个的关系。

在XML代码中对TestView进行引用，这一步很简单，只是简单地引用就不贴代码了。在Activity当中为TestView定义事件。

        public class MainActivity extends AppCompatActivity implements View.OnTouchListener {

            private TestView testView;

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                testView2 = (TestView2) findViewById(R.id.testView2);
                View rootview = button.getRootView();
                testView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Log.d("test", "onClick" + "------" + v);
                    }
                });
                testView.setOnTouchListener(this);
            }

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                Log.d("test", "-----onTouch------" + event.getAction() + "------" + v);
                return false;
            }
        }


在Activity当中给TestView定义OnClickListener以及TochLintener。运行程序，轻轻点击TestView看一下结果：

      D/test: View dispatchTouchEvent0
      D/test: -----onTouch------0------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ........ 0,0-1080,90 #7f070071 app:id/testView}
      D/test: View onTouchEvent0
      
      D/test: View dispatchTouchEvent1
      D/test: -----onTouch------1------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ...P.... 0,0-1080,90 #7f070071 app:id/testView}
      D/test: View onTouchEvent1
      D/test: onClick------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ...P.... 0,0-1080,90 #7f070071 app:id/testView}

我们在代码中是直接把Action打印的，在输出日志中变成了0，1，这也对应我们的ACTION_DOWN和ACTION_UP两个事件.我们以一种Action为一组分析整个事件调用流程。

  ● 第一组Action=0：ACTION_DOWN事件，从输出结果可以看出，执行顺序为了dispatchTouchEvent->onTouch->onTouchEvent。
  
  ● 第二组Action=1：ACTION_UP事件，执行顺序与之前相同，但是在末尾多了一个onClick方法，这是为什么呢？
  
接下来稍微修改一下我们测试代码。修改OnToUch方法返回值为true.

        @Override
        public boolean onTouch(View v, MotionEvent event) {
            Log.d("test", "-----onTouch------" + event.getAction() + "------" + v);
            return false;  ------>return true;
        }                       
    

在看输出的结果：

      D/test: View dispatchTouchEvent0
      D/test: -----onTouch------0------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ........ 0,0-1080,90 #7f070071 app:id/testView}
      
      D/test: View dispatchTouchEvent2
      D/test: -----onTouch------2------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ........ 0,0-1080,90 #7f070071 app:id/testView}
     
     D/test: View dispatchTouchEvent1
      D/test: -----onTouch------1------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ........ 0,0-1080,90 #7f070071 app:id/testView}

ON_DOWN和ACTION_UP两个事件.我们以一种Action为一组分析整个事件调用流程。

  ● 第一组Action=0：ACTION_DOWN事件，从输出结果可以看出，执行顺序为了dispatchTouchEvent->onTouch接结束了，后面的onTouchEvent没有再执行。
  
  ● 第二组Action=2：ACTION_MOVE事件，与ACTION_DOWN事件事件的执行顺序相同。
  
  ● 第二组Action=1：ACTION_UP事件，与ACTION_DOWN事件事件的执行顺序相同。
  
再改一改，撤回之前对OnTouch的操作，还是让onTouch返回false,然后修改onTouchEvent的返回值为false。

        @Override
        public boolean onTouchEvent(MotionEvent event) {
            "   "     ------->super.onTouchEvent(event);
            Log.d("test", "View onTouchEvent"+ event.getAction());
            return super.onTouchEvent(event);------->return fasle;
        }

再来看一下输出结果：

    D/test: View dispatchTouchEvent0
    D/test: -----onTouch------0------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ........ 0,0-1080,90 #7f070071 app:id/testView}
    D/test: View onTouchEvent0

执行顺序变成dispatchTouchEvent->onTouch->onTouchEvent，但是整个事件分发过程中，只剩下了ACTION_DOWN这一种事件，只输出了Action=0的情况，也就是ACTION_DOWN之后的事件都不再响应，
怎么回事呢。

2.2 View源码分析

2.2.1 dispatchTouchEvent

从上面的实例来看，dispatchTouchEvent毫无疑问就是我整个事件分发的入口了，我们从这里入手。话不多说上源码。

//返回结果定义在方法内部变量result当中，当result返回true时，表示事件被消费，不再继续向下分发，为false时继续向下分发

         public boolean dispatchTouchEvent(MotionEvent event) {
            if (event.isTargetAccessibilityFocus()) {
                if (!isAccessibilityFocusedViewOrHost()) {
                    return false;
                }
                // 事件可以被关注并正常分发
                event.setTargetAccessibilityFocus(false);
            }
            //result用来存储函数最后的返回结果。
            boolean result = false;

            if (mInputEventConsistencyVerifier != null) {
                mInputEventConsistencyVerifier.onTouchEvent(event, 0);
            }

            //当actionMasked == MotionEvent.ACTION_DOWN停止滑动事件
            final int actionMasked = event.getActionMasked();
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Defensive cleanup for new gesture
                stopNestedScroll();
            }

            //判断窗口window是否被遮挡，方法返回为true，事件可以被继续分发，false不再继续分发
            if (onFilterTouchEventForSecurity(event)) {
                //View当前是否被激活，并且有滚动事件发生
                if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                    result = true;
                }
                //ListenerInfo是一个内部类，定义了一些监听事件
                ListenerInfo li = mListenerInfo;
                //这里的判断条件分别为li != nul
                //li.mOnTouchListener != null，这个li.mOnTouchListener变量就是通过setOnTouchEvent赋值的。
                //当前View是被激活的状态
                //li.mOnTouchListener.onTouch(this, event)在我们setOnTouchEvent内部有操作，当在这里我们设置View的TouchEvent事件，当返回为true时，reslult=true表示消耗这个事件，将不再继续往下传递。
                if (li != null && li.mOnTouchListener != null
                        && (mViewFlags & ENABLED_MASK) == ENABLED
                        && li.mOnTouchListener.onTouch(this, event)) {
                    result = true;
                }

                //当result=false才会执行onTouchEvent(event)，这也就解释了为什么当onTouch返回true时onTouchEvent(event)不再执行。
                //onTouchEvent(event)也返回true时，result=true
                if (!result && onTouchEvent(event)) {
                    result = true;
                }
            }

            if (!result && mInputEventConsistencyVerifier != null) {
                mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
            }

            // Clean up after nested scrolls if this is the end of a gesture;
            // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
            // of the gesture.
            if (actionMasked == MotionEvent.ACTION_UP ||
                    actionMasked == MotionEvent.ACTION_CANCEL ||
                    (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
                stopNestedScroll();
            }

            return result;
        }


dispatchTouchEvent这个类代码不多，结合注释，方法内的处理主要有：

  ● 定义了result对象来记录事件是否被消费。
  
  ● 拿到了View的mOnTouchListener，判断是否为Null，当不为空并且onTouch事件返回true时，代表事件被消费掉。
  
            if (li != null && li.mOnTouchListener != null
                        && (mViewFlags & ENABLED_MASK) == ENABLED
                        && li.mOnTouchListener.onTouch(this, event)) {
                    result = true;
            }

  ● 在下面的代码中，当result=false才会执行onTouchEvent，这也就解释了我把OnTouch返回值置为true时，onTouchEvent(event)将不再执行，
  因为这个时候result=true，而if(A&&B)当中，只有在A成立的情况下才会执行B，在这里!result显然不成立，所以onTouchEvent(event)也就不会执行。
  
            if (!result && onTouchEvent(event)) {
                        result = true;
            }

  ● 当 onTouchEvent(event)返回false时，result也就返回false了，这个时候就代表dispatchEvent没有消费事件，后续的Action也就不会再执行了。
  
分析到这里，dispatchTouchEvent、OnTouch和OnTouchEevent的调用关系应该是清楚了，我们来总结一下：

      dispatchTouchEvent作为事件分发的入口，在任何时候都会优先执行，在dispatchTouchEvent函数内部调用了onTouch和OnTouchEvent方法。
      
      当onTouch返回True时，dispatchTouchEvent返回值也为True，OnTouchEvent方法将不再执行。
      
      当onTouch返回False时，会执行OnTouchEvent方法，而OnTouchEvent方法的返回值也就决定了dispatchTouchEvent的返回值
      
      当dispatchTouchEvent返回true时表示消费事件，后续的事件将继续响应；当dispatchTouchEvent返回Fasle是表示不消费事件，不会再响应后续事件。
      
2.2.2 onTouchEvent

分析了半天，我们最常用的setOnCLickListener怎么没有出现呢，什么时候会执行onClick方法。我们接着看OnTouchEvent的代码：

          public boolean onTouchEvent(MotionEvent event) {
              final float x = event.getX();
              final float y = event.getY();
              final int viewFlags = mViewFlags;
              final int action = event.getAction();

              //当前视图是否可被执行点击、长按等操作
              //可通过java代码或者xml代码设置enable状态或者clickable状态。
              //当这些状态为false时，则clickable = false，否则为true。
              final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                      || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                      || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

              //判断视图是否已经被销毁
              if ((viewFlags & ENABLED_MASK) == DISABLED) {
                  if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                      setPressed(false);
                  }
                  mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
              //一个已经销毁的视图，被点击时依旧消费事件，只是不能响应事件
                  return clickable;
              }
              if (mTouchDelegate != null) {
                  if (mTouchDelegate.onTouchEvent(event)) {
                      return true;
                  }
              }

              if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
                  switch (action) {
                      case MotionEvent.ACTION_UP:
                          mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                          if ((viewFlags & TOOLTIP) == TOOLTIP) {
                              handleTooltipUp();
                          }
                           //判断是否可点击
                          if (!clickable) {
                              removeTapCallback();
                              removeLongPressCallback();
                              mInContextButtonPress = false;
                              mHasPerformedLongPress = false;
                              mIgnoreNextUpEvent = false;
                              break;
                          }
                          boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                          if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                              // 判断是否可以获取焦点，如果可以，则获取焦点
                              boolean focusTaken = false;
                              if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                                  focusTaken = requestFocus();
                              }

                              if (prepressed) {
                                  // The button is being released before we actually
                                  // showed it as pressed.  Make it show the pressed
                                  // state now (before scheduling the click) to ensure
                                  // the user sees it.
                                  setPressed(true, x, y);
                              }


                              if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                                  // This is a tap, so remove the longpress check
                                  removeLongPressCallback();

                                  // Only perform take click actions if we were in the pressed state
                                  if (!focusTaken) {
                                      // Use a Runnable and post this rather than calling
                                      // performClick directly. This lets other visual state
                                      // of the view update before click actions start.
                                      if (mPerformClick == null) {
                                          mPerformClick = new PerformClick();
                                      }
                                      //最终ACTION_UP要执行的方法，post到UI Thread的中的一个Runnable，如果不存在message queue，则直接执行performClick();
                                      if (!post(mPerformClick)) {
                                          performClick();
                                      }
                                  }
                              }

                              if (mUnsetPressedState == null) {
                                  mUnsetPressedState = new UnsetPressedState();
                              }

                              if (prepressed) {
                                  postDelayed(mUnsetPressedState,
                                          ViewConfiguration.getPressedStateDuration());
                              } else if (!post(mUnsetPressedState)) {
                                  // If the post failed, unpress right now
                                  mUnsetPressedState.run();
                              }

                              removeTapCallback();
                          }
                          mIgnoreNextUpEvent = false;
                          break;

                      case MotionEvent.ACTION_DOWN:
                          if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {
                              mPrivateFlags3 |= PFLAG3_FINGER_DOWN;
                          }
                          mHasPerformedLongPress = false;

                          if (!clickable) {
                              checkForLongClick(0, x, y);
                              break;
                          }

                          if (performButtonActionOnTouchDown(event)) {
                              break;
                          }

                          // 判断当前view是否是在滚动器当中
                          boolean isInScrollingContainer = isInScrollingContainer();

                         //如果是在滚动器当中，在滚动器当中的话延迟返回事件，延迟时间为 ViewConfiguration.getTapTimeout()=100毫秒
                          if (isInScrollingContainer) {
                              mPrivateFlags |= PFLAG_PREPRESSED;
                              if (mPendingCheckForTap == null) {
                                  mPendingCheckForTap = new CheckForTap();
                              }
                              mPendingCheckForTap.x = event.getX();
                              mPendingCheckForTap.y = event.getY();
                              postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                          } else {
                              // 不在滚动器当中则立即做出响应
                              setPressed(true, x, y);
                              checkForLongClick(0, x, y);
                          }
                          break;

                      case MotionEvent.ACTION_CANCEL:
                          if (clickable) {
                              setPressed(false);
                          }
                          removeTapCallback();
                          removeLongPressCallback();
                          mInContextButtonPress = false;
                          mHasPerformedLongPress = false;
                          mIgnoreNextUpEvent = false;
                          mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                          break;

                      case MotionEvent.ACTION_MOVE:
                          if (clickable) {
                              drawableHotspotChanged(x, y);
                          }

                          // 判断当前滑动事件是否还在当前view当中
                          if (!pointInView(x, y, mTouchSlop)) {
                              // 如果超出view，则取消所事件
                              removeTapCallback();
                              removeLongPressCallback();
                              if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                                  setPressed(false);
                              }
                              mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                          }
                          break;
                  }

                  return true;
              }

              return false;
          }

onTouchEvent里面做的事情也不多，主要是分类处理各个不同的Action事件，还是结合注释，我们分析一下大概流程：

  ● 在程序的入口处我们初始化了一个clickable的变量，这个变量在View包括继承自View的自定义View当中默认是fasle的，但是在Button、TextView当中默认就是true。
  当我们设置了setOnClickListener时会也会执行clickable=true。之后便是对不同Action的处理。
  
           final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                          || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                          || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

  ● 从上面事件的触发流程可以知道，第一个事件一定会是ACTION_DOWN。这一部分主要是判断是否在滚动器中，当在滚动器中会有100毫秒的延迟响应，用来判断是否要响应事件。不再滚动器则直接响应事件。
  
  ● ACTION_MOVE只是做了处理响应事件的操作。
  
  ● 最终action = MotionEvent.ACTION_UP执行的是performClick()方法，再来看一下这个方法
  
          public boolean performClick() {
              final boolean result;
              final ListenerInfo li = mListenerInfo;
              if (li != null && li.mOnClickListener != null) {
                  playSoundEffect(SoundEffectConstants.CLICK);
                  //这里实际上就是调用了我们的setOnClickListerner.onClick()方法
                  li.mOnClickListener.onClick(this);
                  result = true;
              } else {
                  result = false;
              }
              sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

              notifyEnterOrExitForAutoFillIfNeeded(true);

              return result;
          }


看到这里我们知道setOnClickListener方法在调用到action = MotionEvent.ACTION_UP方法之后才会执行。这也就解释了为什么在上面的实例当中onClick方法总是在最后执行，而当TouchEvent返回false之后setOnClickListener方法则不会再执行。因为这时候TouchEvent只会响应MotionEvent.ACTION_DOWN事件，而setOnClickListener是在View响应MotionEvent.ACTION_UP事件之后才执行。
2.3 View 事件分发流程图

<img width="576" alt="image" src="https://user-images.githubusercontent.com/67937122/161186502-d8fcb90f-b064-4b57-9e0b-b0a5fe4d8380.png">


3.ViewGroup事件分发

3.1 实例分析

与测试View时的基本流程一样，我们自定义一个TestLayout用来输出事件的调用顺序，具体代码如下：

          public class TestLayout extends LinearLayout {
              public TestLayout(Context context) {
                  super(context);
              }

              public TestLayout(Context context, @Nullable AttributeSet attrs) {
                  super(context, attrs);
              }

              public TestLayout(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
                  super(context, attrs, defStyleAttr);
              }


              @Override
              public boolean onInterceptTouchEvent(MotionEvent ev) {
                  Log.d("test", "TestLayout onInterceptTouchEvent-- action=" + ev.getAction());
                  return super.onInterceptTouchEvent(ev);
              }

              @Override
              public boolean dispatchTouchEvent(MotionEvent event) {
                  Log.d("test","TestLayout dispatchTouchEvent-- action=" + event.getAction());
                  return super.dispatchTouchEvent(event);
              }

              @Override
              public boolean onTouchEvent(MotionEvent event) {
                  Log.d("test","TestLayout onTouchEvent-- action=" + event.getAction());
                  return super.onTouchEvent(event);
              }
          }

因为LinearLayout是直接继承自ViewGroup的，所以为了布局方便这里我们的自定义TestView选择继承LinearLayout。

在ViewGroup当中涉及到事件分发的方法,相比View来说除了dispatchTouchEvent和onTouchEvent外还有onInterceptTouchEvent。

为了测试方便，我们把TestView的代码改为最开始的样子：


          public class TestView extends View {

              public TestView(Context context) {
                  super(context);
              }

              public TestView(Context context, @Nullable AttributeSet attrs) {
                  super(context, attrs);
              }

              public TestView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
                  super(context, attrs, defStyleAttr);
              }

              @Override
              protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
                  super.onMeasure(widthMeasureSpec, heightMeasureSpec);
              }

              @Override
              public boolean onTouchEvent(MotionEvent event) {
                  Log.d("test", "View onTouchEvent"+ event.getAction());
                  return super.onTouchEvent(event);
              }

              @Override
              public boolean dispatchTouchEvent(MotionEvent event) {
                  Log.d("test", "View dispatchTouchEvent"+ event.getAction() );
                  return super.dispatchTouchEvent(event);
              }

          }

XML依然只是引入最简单的布局，这里省略掉，看一下Activity当中的设置：

          public class MainActivity extends AppCompatActivity implements View.OnTouchListener {

              private TestView testView;
              private TestLayout testLayout;

              @Override
              protected void onCreate(Bundle savedInstanceState) {
                  super.onCreate(savedInstanceState);
                  setContentView(R.layout.activity_main);
                  testView = (TestView) findViewById(R.id.testView);
                  testLayout = (TestLayout) findViewById(R.id.test_layout);
                  testLayout.setOnClickListener(new View.OnClickListener() {
                      @Override
                      public void onClick(View v) {
                          Log.d("test", "TestLayout onClick" + "------" + v);
                      }
                  });
                  testView.setOnClickListener(new View.OnClickListener() {
                      @Override
                      public void onClick(View v) {
                          Log.d("test", "onClick" + "------" + v);
                      }
                  });
                  testLayout.setOnTouchListener(this);
                  testView.setOnTouchListener(this);

              }

              @Override
              public boolean onTouch(View v, MotionEvent event) {
                  Log.d("test", "-----onTouch------" + event.getAction() + "------" + v);
                  return false;
              }
          }

运行程序，点击TestView所在区域，看一下输出的结果：

          D/test: TestLayout dispatchTouchEvent-- action=0
          D/test: TestLayout onInterceptTouchEvent-- action=0
          D/test: View dispatchTouchEvent0
          D/test: -----onTouch------0------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ........ 0,0-1080,90 #7f070071 app:id/testView}
          D/test: View onTouchEvent0
          D/test: TestLayout dispatchTouchEvent-- action=2
          D/test: TestLayout onInterceptTouchEvent-- action=2
          D/test: View dispatchTouchEvent2
          D/test: -----onTouch------2------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ...P.... 0,0-1080,90 #7f070071 app:id/testView}
          D/test: View onTouchEvent2
          D/test: TestLayout dispatchTouchEvent-- action=1
          D/test: TestLayout onInterceptTouchEvent-- action=1
          D/test: View dispatchTouchEvent1
          D/test: -----onTouch------1------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ...P.... 0,0-1080,90 #7f070071 app:id/testView}
          D/test: View onTouchEvent1
          D/test: onClick------com.example.liubohua.viewtestapplication.TestView{40f09bf V.ED..C.. ...P.... 0,0-1080,90 #7f070071 app:id/testView}
          
输出的日志有点多，我们以一个Action为一组进行分析，之前我们也说过，一个Action代表一类的Touch事件。

  ● 第一租Action=0：ACTION_DOIWN事件，先执行了dispatchTouchEvent，然后是onInterceptTouchEvent，这两个都是TestLayout的方法,之后的事件被传递到了TestView当中，
  按照我们之前说的顺序dispatchTouchEvent->onTouch->onTouchEvent进行事件分发。
  
  ● 第二组Action=2：ACTION_MOVE事件，与第一组的时间顺序相同。
  
  ● 第三组Action=1：ACTION_UP事件，与上述步骤基本相同，但是在输出末尾执行了View的onClick事件，这就与我们之前分析的View当中的事件分发机制不谋而合。
  
我们之前说ViewGroup的事件分发与dispatchTouchEvent、onTouchEvent、onInterceptTouchEvent三个方法有关，
但是从这个例子当中我们发现所执行的方法只有dispatchTouchEvent和onInterceptTouchEvent，那么onTouchEvent在什么时候执行呢？我们设置的OnTouch方法以及OnClick方法在什么时候调用呢？
带着疑问我们点击一下TestView以外的区域,再查看输出日志:

          D/test: TestLayout dispatchTouchEvent-- action=0
          D/test: TestLayout onInterceptTouchEvent-- action=0
          D/test: -----onTouch------0------com.example.liubohua.viewtestapplication.TestLayout{15fd3de V.E...C.. ........ 0,0-1080,1680 #7f070074 app:id/test_layout}
          D/test: TestLayout onTouchEvent-- action=0
          D/test: TestLayout dispatchTouchEvent-- action=2
          D/test: -----onTouch------2------com.example.liubohua.viewtestapplication.TestLayout{15fd3de V.E...C.. ...P.... 0,0-1080,1680 #7f070074 app:id/test_layout}
          D/test: TestLayout onTouchEvent-- action=2
          D/test: TestLayout dispatchTouchEvent-- action=1
          D/test: -----onTouch------1------com.example.liubohua.viewtestapplication.TestLayout{15fd3de V.E...C.. ...P.... 0,0-1080,1680 #7f070074 app:id/test_layout}
          D/test: TestLayout onTouchEvent-- action=1
          D/test: TestLayout onClick------com.example.liubohua.viewtestapplication.TestLayout{15fd3de V.E...C.. ...P.... 0,0-1080,1680 #7f070074 app:id/test_layout}

同样的还是以不同的Action来分组分析：

  ● 第一租Action=0：ACTION_DOIWN事件，执行顺序为dispatchTouchEvent->onInterceptTouchEvent->onTouch->onTouchEvent，而且四个方法全部都是TestLayout内的方法输出，
  没有了View的参与瞬间清爽了很多。
  
  ● 第二组Action=2：ACTION_MOVE事件，执行顺序为dispatchTouchEvent->onTouch->onTouchEvent，到了这里onInterceptTouchEvent方法又不见了。
  
  ● 第三组Action=1：ACTION_UP事件，这里就更过分了，执行的流程当中不仅没有了onInterceptTouchEvent方法，在输出末尾还读了TestLayOut的onClick事件，这么任性到底是要闹哪样。
我再看到这里的时候也是一头雾水，这事件执行的还真是随意，一会一个样子。没办法还是去源码里面找答案吧。

3.2 ViewGroup源码分析

与View一样，整个事件分发的入口依然是dispatchTouchEvent，我们从这里入手。

          @Override
          public boolean dispatchTouchEvent(MotionEvent ev) {
              if (mInputEventConsistencyVerifier != null) {
                  mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
              }

              // 如果事件指向了一个可达的视图，则事件正常调度分发。
              if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
                  ev.setTargetAccessibilityFocus(false);
              }

              boolean handled = false;
              //进行安全策略检查，返回true正常进行下一步操作
              if (onFilterTouchEventForSecurity(ev)) {
                  final int action = ev.getAction();
                  final int actionMasked = action & MotionEvent.ACTION_MASK;

                  //当action为MotionEvent.ACTION_DOWN时做一些初始化操作
                  if (actionMasked == MotionEvent.ACTION_DOWN) {
                     //在这里把所有的TouchTarget置空。这里需要说明一下，当TouchTarget不为null时，表示已经找到能够处理touch事件的目标。
                      //touchTarget是一个链表，用来存储可以用来响应事件的view
                      cancelAndClearTouchTargets(ev);
                      //重置了所有的TouchState
                      resetTouchState();
                  }

                  // intercepted变量用来记录是否需要拦截事件
                  final boolean intercepted;
                  //当actionMasked == MotionEvent.ACTION_DOWN或者已经找到可以处理事件的view时满足判断条件
                  if (actionMasked == MotionEvent.ACTION_DOWN
                          || mFirstTouchTarget != null) {
                      //disallowIntercept为是否开启禁止拦截的标志，当disallowIntercept=true时禁止拦截，否则开启
                      //调用requestDisallowInterceptTouchEvent(boolean disallowIntercept)方法可以设置是否关闭拦截
                      final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                      //判断disallowIntercept=false，开启拦截
                      if (!disallowIntercept) {
                          //执行onInterceptTouchEvent方法，也就是我们自己定义的回调
                          //默认onInterceptTouchEvent(ev)方法返回false，不拦截事件，当返回true时表示事件拦截
                          intercepted = onInterceptTouchEvent(ev);
                          ev.setAction(action); // restore action in case it was changed
                      } else {
                          //判断disallowIntercept=true，赋值intercepted = false，表示不拦截事件
                          intercepted = false;
                      }
                  } else {
                      // 当没有View处理事件，并且不是ACTION_DOWN时，继续拦截。
                      intercepted = true;
                  }

                  //如何事件被拦截，或者有View处理该事件，则正常处理。
                  if (intercepted || mFirstTouchTarget != null) {
                      ev.setTargetAccessibilityFocus(false);
                  }

                  // canceled变量初始化，
                  final boolean canceled = resetCancelNextUpFlag(this)
                          || actionMasked == MotionEvent.ACTION_CANCEL;

                  // split变量决定是否吧事件分发给多个子View，可以通过setMotionEventSplittingEnabled(boolean split)方法进行设置
                  final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
                  TouchTarget newTouchTarget = null;
                  boolean alreadyDispatchedToNewTouchTarget = false;
                  //没有取消并且没有拦截事件时进入判断
                  if (!canceled && !intercepted) {
                  //查找优先处理action的view，
                  //在View当中可以通过getRootView().getParent().requestSendAccessibilityEvent(View child, AccessibilityEvent event));方法进行设置
                      View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                              ? findChildWithAccessibilityFocus() : null;
                      //不满足条件时表示不需要再重新寻找响应事件的View
                      if (actionMasked == MotionEvent.ACTION_DOWN
                              || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                              || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                          final int actionIndex = ev.getActionIndex(); // always 0 for down
                          final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                                  : TouchTarget.ALL_POINTER_IDS;

                          // Clean up earlier touch targets for this pointer id in case they
                          // have become out of sync.
                          removePointersFromTouchTargets(idBitsToAssign);

                          final int childrenCount = mChildrenCount;
                          //判断childrenCount是否等于0并且newTouchTarget == null
                          if (newTouchTarget == null && childrenCount != 0) {
                              //获取事件的坐标值，没有在坐标内的View不需要再响应事件
                              final float x = ev.getX(actionIndex);
                              final float y = ev.getY(actionIndex);

                              final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                              final boolean customOrder = preorderedList == null
                                      && isChildrenDrawingOrderEnabled();
                              final View[] children = mChildren;
                              //倒序遍历所有的子View，这里采用一种倒序的方式，在java代码中后addView或者在xml当中后添加的布局会先被响应事件。
                              for (int i = childrenCount - 1; i >= 0; i--) {
                                  final int childIndex = getAndVerifyPreorderedIndex(
                                          childrenCount, i, customOrder);
                                  final View child = getAndVerifyPreorderedView(
                                          preorderedList, children, childIndex);
                                   //如果存在优先要处理Action的View，则进行下次循环直到找到View
                                  //当childWithAccessibilityFocus=null时正常分发事件到每一个View
                                  if (childWithAccessibilityFocus != null) {
                                      if (childWithAccessibilityFocus != child) {
                                          continue;
                                      }
                                      childWithAccessibilityFocus = null;
                                      i = childrenCount - 1;
                                  }
                                  //如果当前点击的位置没有被子View占据，则直接进入下一次循环
                                  if (!canViewReceivePointerEvents(child)
                                          || !isTransformedTouchPointInView(x, y, child, null)) {
                                      ev.setTargetAccessibilityFocus(false);
                                      continue;
                                  }
                                  //在TouchTarget当中在此寻找该View，如果找到了直接退出循环，没有找到则继续向下执行
                                  newTouchTarget = getTouchTarget(child);
                                  if (newTouchTarget != null) {
                                      // Child is already receiving touch within its bounds.
                                      // Give it the new pointer in addition to the ones it is handling.
                                      newTouchTarget.pointerIdBits |= idBitsToAssign;
                                      break;
                                  }

                                  //递归的方式在子View的dispatchTouchEvent方法，寻找可以响应action的View，当返回为True时，进入循环，表示找到可以响应的View。
                                  resetCancelNextUpFlag(child);
                                  if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                      // Child wants to receive touch within its bounds.
                                      mLastTouchDownTime = ev.getDownTime();
                                      if (preorderedList != null) {
                                          // childIndex points into presorted list, find original index
                                          for (int j = 0; j < childrenCount; j++) {
                                              if (children[childIndex] == mChildren[j]) {
                                                  mLastTouchDownIndex = j;
                                                  break;
                                              }
                                          }
                                      } else {
                                          mLastTouchDownIndex = childIndex;
                                      }
                                      mLastTouchDownX = ev.getX();
                                      mLastTouchDownY = ev.getY();
                                      //将找到的View加入到TouchTarget当中
                                      newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                      alreadyDispatchedToNewTouchTarget = true;
                                      //找到View 跳出循环
                                      break;
                                  }

                                  // The accessibility focus didn't handle the event, so clear
                                  // the flag and do a normal dispatch to all children.
                                  ev.setTargetAccessibilityFocus(false);
                              }
                              if (preorderedList != null) preorderedList.clear();
                          }
                          //当不能再找到可以响应action的子View时，将TouchTarget链表指到最初找到的View
                          if (newTouchTarget == null && mFirstTouchTarget != null) {
                              // Did not find a child to receive the event.
                              // Assign the pointer to the least recently added target.
                              newTouchTarget = mFirstTouchTarget;
                              while (newTouchTarget.next != null) {
                                  newTouchTarget = newTouchTarget.next;
                              }
                              newTouchTarget.pointerIdBits |= idBitsToAssign;
                          }
                      }
                  }

                  //当没有View消费事件时，mFirstTouchTarget == null
                  if (mFirstTouchTarget == null) {
                      // 这里调用dispatchTransformedTouchEvent，其实也就是调用ViewGroup自身TouchEvent事件
                      handled = dispatchTransformedTouchEvent(ev, canceled, null,
                              TouchTarget.ALL_POINTER_IDS);
                  } else {
                      // Dispatch to touch targets, excluding the new touch target if we already
                      // dispatched to it.  Cancel touch targets if necessary.
                      TouchTarget predecessor = null;
                      TouchTarget target = mFirstTouchTarget;
                      while (target != null) {
                          final TouchTarget next = target.next;
                          //alreadyDispatchedToNewTouchTarget=true表示在上面遍历View的过程中已经消费了事件
                          if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                              handled = true;
                          } else {
                              //调用子View的dispatchOnTouchEvent进行事件处理
                              final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                      || intercepted;
                              if (dispatchTransformedTouchEvent(ev, cancelChild,
                                      target.child, target.pointerIdBits)) {
                                  handled = true;
                              }
                              if (cancelChild) {
                                  if (predecessor == null) {
                                      mFirstTouchTarget = next;
                                  } else {
                                      predecessor.next = next;
                                  }
                                  target.recycle();
                                  target = next;
                                  continue;
                              }
                          }
                          predecessor = target;
                          target = next;
                      }
                  }

                  // Update list of touch targets for pointer up or cancel, if needed.
                  if (canceled
                          || actionMasked == MotionEvent.ACTION_UP
                          || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                      resetTouchState();
                  } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                      final int actionIndex = ev.getActionIndex();
                      final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                      removePointersFromTouchTargets(idBitsToRemove);
                  }
              }

              if (!handled && mInputEventConsistencyVerifier != null) {
                  mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
              }
              return handled;
          }

整个代码有点多，有兴趣的可以逐行分析。这里我们挑出重点步骤分析一下。

首先是初始化的部分，初始化是在ACTION_DOWN产生时发生的。这里有一个非常重要的操作就是把touchTarget是链表置空。

touchTarget链表本身是用来存储在一次分发过程中可响应事件的View的链表，每一次ACTION_DOWN产生事件产生，都代表一个新的事件开始，这个时候清空touchTarget重新寻找可以响应的View，
一旦找到可以相应的View，就存储进来，在后续的ACTION_MOVE等事件发生时，直接在touchTarget链表中寻找事件的消费者。

        //当action为MotionEvent.ACTION_DOWN时做一些初始化操作
        if (actionMasked == MotionEvent.ACTION_DOWN) {
           //在这里把所有的TouchTarget置空。这里需要说明一下，当TouchTarget不为null时，表示已经找到能够处理touch事件的目标。
            //touchTarget是一个链表，用来存储可以用来响应事件的view
            cancelAndClearTouchTargets(ev);
            //重置了所有的TouchState
            resetTouchState();
        }

初始化之后是对拦截器的判断，onInterceptTouchEvent的执行与否就和这里密切相关。
首先初始化了一个标志位disallowIntercept，这个标志位用来标记时候禁用拦截器，在我们的代码中可以通过requestDisallowInterceptTouchEvent(boolean disallowIntercept)方法
来对disallowIntercept的值进行设置。
当disallowIntercept=false时表示不禁用拦截器，正常执行onInterceptTouchEvent(ev);方法，并且将该方法的返回值返回给intercepted变量用来标志是否拦截事件。
当disallowIntercept=true时表示禁用拦截器，执行intercepted = false;也就是对事件不进行拦截。
这里也就解释了为什么点击ViewGroup空白处时onInterceptTouchEvent(ev)只调用一次了

        //intercepted变量用来记录是否需要拦截事件
        final boolean intercepted;
        //当actionMasked == MotionEvent.ACTION_DOWN或者已经找到可以处理事件的view时满足判断条件
        if (actionMasked == MotionEvent.ACTION_DOWN
                || mFirstTouchTarget != null) {
            //disallowIntercept为是否开启禁止拦截的标志，当disallowIntercept=true时禁止拦截，否则开启
            //调用requestDisallowInterceptTouchEvent(boolean disallowIntercept)方法可以设置是否关闭拦截
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            //判断disallowIntercept=false，开启拦截
            if (!disallowIntercept) {
                //执行onInterceptTouchEvent方法，也就是我们自己定义的回调
                //默认onInterceptTouchEvent(ev)方法返回false，不拦截事件，当返回true时表示事件拦截
                intercepted = onInterceptTouchEvent(ev);
                ev.setAction(action); // restore action in case it was changed
            } else {
                //判断disallowIntercept=true，赋值intercepted = false，表示不拦截事件
                intercepted = false;
            }
        } else {
            // 当没有View处理事件，并且不是ACTION_DOWN时，继续拦截。
            intercepted = true;
        }

接下来会对子View进行遍历，这里采用一个倒序遍历的方式，在java代码中后addView或者在xml当中后添加的布局会先被响应事件，这也符合我们的视图逻辑，在最上层的View应该最优先响应事件。

           for (int i = childrenCount - 1; i >= 0; i--) {
                  final int childIndex = getAndVerifyPreorderedIndex(
                                              childrenCount, i, customOrder);
                  final View child = getAndVerifyPreorderedView(
                                              preorderedList, children, childIndex);

再往下执行，有一个判断语句if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign))当它返回true时代表找到了可以响应事件的View，返回Fasle表示没有找到。
dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)方法中有个重要的参数child，当child！=null时，调用了子View的dispatchTouchEvent方法判断是否对事件
进行响应。如果child==null，则调用super.dispatchTouchEvent(event);也就是View的dispatchTouchEvent(event);方法进行处理。

          private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
                      View child, int desiredPointerIdBits) {
                  final boolean handled;

                  // Canceling motions is a special case.  We don't need to perform any transformations
                  // or filtering.  The important part is the action, not the contents.
                  final int oldAction = event.getAction();
                  if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
                      event.setAction(MotionEvent.ACTION_CANCEL);
                      if (child == null) {
                          handled = super.dispatchTouchEvent(event);
                      } else {
                          handled = child.dispatchTouchEvent(event);
                      }
                      event.setAction(oldAction);
                      return handled;
                  }
          }

当找到可以响应事件的View时进入判断，最终通过newTouchTarget = addTouchTarget(child, idBitsToAssign);方法将子View添加到链表TouchTarget当中。
在最后一部分的代码里决定了具体后续将如何响应事件。
前面我们分析过复合条件的View都会被存储到mFirstTouchTarget所对应的链表当中，当mFirstTouchTarget == null时就需要ViewGroup自己处理事件，这个时候同样调用
dispatchTransformedTouchEvent方法，但是chlid参数传参为null，上面我们也分析过，会调用View的dispatchTouchEvent(event)方法。
当mFirstTouchTarget != null表示存在可以响应事件的子View，同样也是调用dispatchTransformedTouchEvent方法，但是chlid参数传参为则传入可实际响应事件的View。

        //当没有View消费事件时，mFirstTouchTarget == null
        if (mFirstTouchTarget == null) {
            // 这里调用dispatchTransformedTouchEvent，其实也就是调用ViewGroup自身TouchEvent事件
            handled = dispatchTransformedTouchEvent(ev, canceled, null,
                    TouchTarget.ALL_POINTER_IDS);
        } else {
            // Dispatch to touch targets, excluding the new touch target if we already
            // dispatched to it.  Cancel touch targets if necessary.
            TouchTarget predecessor = null;
            TouchTarget target = mFirstTouchTarget;
            while (target != null) {
                final TouchTarget next = target.next;
                //alreadyDispatchedToNewTouchTarget=true表示在上面遍历View的过程中已经消费了事件
                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                    handled = true;
                } else {
                    //调用子View的dispatchOnTouchEvent进行事件处理
                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
                            || intercepted;
                    if (dispatchTransformedTouchEvent(ev, cancelChild,
                            target.child, target.pointerIdBits)) {
                        handled = true;
                    }
                    if (cancelChild) {
                        if (predecessor == null) {
                            mFirstTouchTarget = next;
                        } else {
                            predecessor.next = next;
                        }
                        target.recycle();
                        target = next;
                        continue;
                    }
                }
                predecessor = target;
                target = next;
            }
        }

分析到这里ViewGroup的事件分发也基本完成了。这里没有再分析onTouchEevent和onTouch方法，这两个方法和View当中的处理并没有区别。

3.3 ViewGroup事件分发流程图

![image](https://user-images.githubusercontent.com/67937122/161189882-7f782187-61b8-4731-9127-e0fe6f7fa996.png)

事件分发到这里就结束了?在我们的自己的布局当中看是这样的，但是除了在我们XML中定义的布局之外，Android视图还有默认的Activity层级。

4. Activity事件分发
4.1 实例分析

之前的布局都不变,再Activity当中做出修改：

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.i("test", "MainActivity--dispatchTouchEvent--action=" + ev.getAction());
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public void onUserInteraction() {
        Log.d("test", "MainActivity--onUserInteraction");
        super.onUserInteraction();
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("test", "MainActivity--onTouchEvent--action=" + event.getAction());
        return super.onTouchEvent(event);
    }

这三个都是Activity提供的可重写的方法，再点击一下看看如何输出日志：

    I/test: MainActivity--dispatchTouchEvent--action=0
    D/test: MainActivity--onUserInteraction
    D/test: TestLayout dispatchTouchEvent-- action=0
    D/test: TestLayout onInterceptTouchEvent-- action=0false
    D/test: -----onTouch------0------com.example.liubohua.viewtestapplication.TestLayout{6de08b5 V.E...C.. ........ 0,0-1920,864 #7f070074 app:id/test_layout}
    D/test: TestLayout onTouchEvent-- action=0
    I/test: MainActivity--dispatchTouchEvent--action=1
    D/test: TestLayout dispatchTouchEvent-- action=1
    D/test: -----onTouch------1------com.example.liubohua.viewtestapplication.TestLayout{6de08b5 V.E...C.. ...P.... 0,0-1920,864 #7f070074 app:id/test_layout}
    D/test: TestLayout onTouchEvent-- action=1

依然是分组分析。

  ● 第一租Action=0：ACTION_DOIWN事件，先执行了Activity的dispatchTouchEvent->onUserInteraction,
    接着执行TestLayout的dispatchTouchEvent->onInterceptTouchEvent->onTouch->onTouchEvent。
    
  ● 第二组Action=1：ACTION_UP事件，先执行了Activity的dispatchTouchEvent,接着执行TestLayout的dispatchTouchEvent->onTouch->onTouchEvent。

话不多说，咱们直接看源码。

4.2 源码分析

dispatchTouchEvent同样还是整个事件的入口

          public boolean dispatchTouchEvent(MotionEvent ev) {
                  if (ev.getAction() == MotionEvent.ACTION_DOWN) {
                      onUserInteraction();
                  }
                  if (getWindow().superDispatchTouchEvent(ev)) {
                      return true;
                  }
                  return onTouchEvent(ev);
              }

这里代码很少，我们逐行分析。

当MotionEvent.ACTION_DOWN事件产生时，就执行了onUserInteraction()，没有任何前置条件，在一次事件流程中一定会执行且只执行一次。

          public void onUserInteraction() {
          }

onUserInteraction()里面没有任何实现，这个方法提供出来就是为用户提供的。

接下执行getWindow，获取到一个PhoneWindow实例，去PhoneWindow当中在看一下superDispatchTouchEvent(ev)方法都做了些什么。

          @Override
          public boolean superDispatchTouchEvent(MotionEvent event) {
              return mDecor.superDispatchTouchEvent(event);
          }


又调用了mDecor的方法，我们再跟进去看看。

          public class DecorView extends FrameLayout{
              public boolean superDispatchTouchEvent(MotionEvent event) {
                  return super.dispatchTouchEvent(event);
              }
          }


好了到这里我们就应明了了，DecorView里面直接调用了super.dispatchTouchEvent(event)，而这个super是谁呢？就是FrameLayout，
FrameLayout的dispatchTouchEvent(event)就是ViewGoup的dispatchTouchEvent(event)。
尾声
到这里差不多我们的事件分发就结束了，从Activity一路分发到了View当中。但是在本文中并没有介绍Touch具体是从哪里产生并分发到我们的Activity当中的，
有兴趣的可以了解一下Android 事件来源介绍的很详细。


转自：https://www.jianshu.com/p/bc4c9e5f4b1c





