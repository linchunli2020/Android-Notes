<img width="525" alt="image" src="https://user-images.githubusercontent.com/67937122/161221282-53f33659-ab12-4237-9542-508a2eaaa61d.png">

（一）Activity A 启动 Activity B，回调如下：

    Activity A的onPause()-> Activity B 的onCreate() -> onStart() -> onResume() -> Activity A的onStop();

    如果B是透明主题，又或者是个DialogActivity，则不会回调A的onStop()。

（二）使用onSaveInstanceState()保存简单，轻量级的UI状态

          lateinit var textView: TextView
          var gameState: String? = null

          override fun onCreate(savedInstanceState: Bundle?) {
              super.onCreate(savedInstanceState)
              gameState = savedInstanceState?.getString(GAME_STATE_KEY)
              setContentView(R.layout.activity_main)
              textView = findViewById(R.id.text_view)
          }

          override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
              textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
          }

          override fun onSaveInstanceState(outState: Bundle?) {
              outState?.run {
                  putString(GAME_STATE_KEY, gameState)
                  putString(TEXT_VIEW_KEY, textView.text.toString())
              }
              super.onSaveInstanceState(outState)
          }


***Fragment的生命周期***

![image](https://user-images.githubusercontent.com/67937122/220003897-4c2ac563-57e5-4c1a-8552-88eba94706de.png)


**如下4种状态：**

    运行：当前Fmgment位于前台，用户可见，可以获得焦点。
    暂停：其他Activity位于前台，该Fragment依然可见，只是不能获得焦点。
    停止：该Fragment不可见，失去焦点。
    销毁：该Fragment被完全删除，或该Fragment所在的Activity被结束。
    
    
![image](https://user-images.githubusercontent.com/67937122/220004001-5bd27ae2-1810-4830-8297-453fd2ad17de.png)

从上图可以看出，Activity中的生命周期方法，Fragment中基本都有，但是Fragment比Activity多几个方法。各生命周期方法的含义如下：

**onAttach() ：**

当Fragment与Activity发生关联时调用。

**onCreate()：**

创建Fragment时被回调。

**onCreateView()：**

每次创建、绘制该Fragment的View组件时回调该方法，Fragment将会显示该方法返回的View 组件。

**onActivityCreated()：**

当 Fragment 所在的Activity被启动完成后回调该方法。

**onStart()：**

启动 Fragment 时被回调，此时Fragment可见。

**onResume()：**

恢复 Fragment 时被回调，获取焦点时回调。

**onPause()：**

暂停 Fragment 时被回调，失去焦点时回调。

**onStop()：**

停止 Fragment 时被回调，Fragment不可见时回调。

**onDestroyView()：**

销毁与Fragment有关的视图，但未与Activity解除绑定。

**onDestroy()：**

销毁 Fragment 时被回调。

**onDetach()：**

与onAttach相对应，当Fragment与Activity关联被取消时调用。

***生命周期调用***

**1）创建Fragment**
onAttach() —> onCreate() —> onCreateView() —> onActivityCreated() —> onStart() —> onResume()

**2）按下Home键回到桌面 / 锁屏**
onPause() —> onStop()

**3）从桌面回到Fragment / 解锁**
onStart() —> onResume()

**4）切换到其他Fragment**
onPause() —> onStop() —> onDestroyView()

**5）切换回本身的Fragment**
onCreateView() —> onActivityCreated() —> onStart() —> onResume()

**6） 按下Back键退出**
onPause() —> onStop() —> onDestroyView() —> onDestroy() —> onDetach()



