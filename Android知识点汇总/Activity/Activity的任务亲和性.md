
[Activity](#activity)
   [Activity的任务亲和性](#activity_taskAffinity)
   [Actvity四种启动模式](#activity_launchMode)
   


<span id="activity">Activity</span>
<span id="activity_taskAffinity">Activity的任务亲和性</span>
（一）任务亲和性 ( taskAffinity ) 简介：

① 亲和性概念 : 任务亲和性 ( taskAffinity ) 是 Activity 的属性 , 用于设置该 Activity 倾向于哪个任务 ;
   关于任务的概念参考 : 【Android 应用开发】Android 返回堆栈 与 任务

② 任务亲和性 ( taskAffinity ) 属性值 : 该值是软件包的 包名 , 定义在 AndroidManifest.xml 或 build.gradle 配置文件 中 ;

    ( 如 : “com.android.example” )
    <activity android:name=".MainActivity"
              android:launchMode="standard"
              android:taskAffinity="com.android.example"/>

③ 任务亲和性 ( taskAffinity ) 设置效果 : 具有相同的 任务亲和性 ( taskAffinity ) 属性的 Activity , 会倾向于放在同一个任务 ( 返回堆栈 ) 中 ;

（二） 任务亲和性 ( taskAffinity ) 设置：

1 . Activity 默认的 任务亲和性 ( taskAffinity ) 属性 : 如果开发者没有指定该 Activity 的 taskAffinity 属性 , 那么该值默认就是该应用的包名 ;

2 . 任务亲和性的三种情况 :

  ① 相同应用 , 相同的亲和性 ( 默认状态 ) : 相同的应用会默认其 Activity 具有相同的亲和性 , 其属性值就是 本身应用的 包名 , 默认设置下 , 每个打开的 Activity 界面都放在同一个 任务     ( 返回堆栈 ) 中 ;
  
  ② 相同应用 , 不同的亲和性 : 如果在同一个应用中 , 为 某个 Activity 设置了不同的亲和性 , 那么打开这个 Activity 界面时 , 就会在其它的任务中打开该界面 ;
  
  ③ 不同应用 , 相同的亲和性 : 如果其它应用 Activity 界面的亲和性 属性就是本应用的包名 , 那么打开该 Activity 界面时 , 该界面就会放入本应用的 返回堆栈中 ;

3 . 注意事项 : 如果要设置 任务亲和性 ( taskAffinity ) 属性 , 该值不能是应用的默认包名 , 只能设置其它的包名 ;

( 即 : 如果设置亲和性属性 , 那么就要设置成不一样的 , 默认的就不要再显示的设置一遍了)


<span id="activity_taskAffinity">Actvity四种启动模式</span>

【简介】

Activity 有四种不同的启动模式，这四种模式分别是：standard，singleTop，singleTask，singleInstance。

这四种模式中，standard模式是默认的模式，其他三个想要使用的话，需要在 AndroidMainFest 文件中进行修改（通过给对应的 activity 设置 launchMode 属性，例如：）。

【预备知识】

1.任务栈（Task Stack） 介绍（任务栈也叫返回栈（Back Stack））：

  1.1 任务栈用于存放用户开启的 Activity
  
  1.2 在应用程序创建之初，系统会启动一个任务栈（默认一个），并存储根 Activity
  
  1.3 新启动的 Activity 将会位于栈顶
  
  1.4 当任务栈的最后一个 Activity 被销毁时，将清除任务栈并退出程序，下次再进入程序时会创建新的任务栈
  
2.taskAffinity 介绍：

      2.1 taskAffinity 是 Activity 的一个属性，可在 Manifest 文件中设置。

      2.2 Task 也有该属性，它的值由第一个入栈的 Activity 决定

      2.3 Application 也有 taskAffinity 属性，它的值为 Manifest 的包名

      2.4 默认情况下（没有显示设置 Activity 的 taskAffinity），所有 Activity 的 taskAffinity 属性都从 Application 继承，也就是说所有 Activity 的 taskAffinity 值都相同，为包名。

      2.5 taskAffinity 的值应该是 xxx.xxx.xxx 这种样式，如果只是普通的字符串 xxx，是安装不了应用的。
  
3.如何在代码中获取 Activity 的 taskAffinity 属性值和 Activity 所在 Task：

        // 当前 Activity 的 taskAffinity 属性值 
        String taskAffinity = "";
        try {
            ActivityInfo activityInfo = getPackageManager().getActivityInfo(getComponentName(),
                    PackageManager.GET_META_DATA);
            taskAffinity = activityInfo.taskAffinity;
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }

        // 当前 Activity 所在 Task
        int taskId = getTaskId();


【四种模式】

（一）standard模式：

标准启动模式，也是系统默认启动模式。该模式下启动同一个activity，会在栈中产生多个该activity的实例，每个实例都会处理一个intent对象。

如果在 AndroidManifest.xml 中将 Activity B 的启动模式设置为 Standard，不管任务栈内是否已经存在 Activity B 的实例，当启动 Activity B 时，都会创建一个崭新的 Activity B 位于任务栈顶（如下图所示）：

<img width="684" alt="image" src="https://user-images.githubusercontent.com/67937122/161175846-cb590c20-1aaa-47ce-b931-5f6350254d64.png">

（二）singleTop模式：

如果启动的activity已存在于任务栈的栈顶，那么再启动这个activity不会创建新的实例，而是重用栈顶的那个实例，并且会调用该activity的onNewIntent方法。

（使用场景像是阅读类/新闻类app的内容页）

(1）如果在 AndroidManifest.xml 中将 Activity D 的启动模式设置为 SingleTop 并且任务栈内存在 Activity D 实例且位于栈顶时，当启动 Activity D 时，会复用之前创建的 Activity D 的实例，并且 onNewIntent() 方法被调用。

<img width="695" alt="image" src="https://user-images.githubusercontent.com/67937122/161176002-eaf21949-bc39-4f90-9c9b-8198e7b9123e.png">

(2）如果在 AndroidManifest.xml 中将 Activity D 的启动模式设置为 SingleTop 并且任务栈内并不存在 Activity D 的实例时，当启动 Activity D 时，会创建一个崭新的 Activity D 实例在栈顶。

<img width="697" alt="image" src="https://user-images.githubusercontent.com/67937122/161176093-9bd5adb2-6bac-4aa5-846d-e5994fea58fd.png">

(3）如果在 AndroidManifest.xml 中将 Activity D 的启动模式设置为 SingleTop 并且任务栈内存在 Activity D 的实例但其实例未在栈顶时，当启动 Activity D 时，会再次创建一个崭新的 Activity D 实例在栈顶。

<img width="680" alt="image" src="https://user-images.githubusercontent.com/67937122/161176173-cfed8835-5c44-4def-97cd-edd3464e576e.png">

（三）singleTask模式：

检查整个任务栈的活动，如果发现已经存在该活动就将位于该活动上方的活动全部出栈，该活动成为新的栈顶。

（使用场景像是浏览器的主页面，不管从多少个应用启动浏览器，只会启动主页面一次，其余情况都会走onNewIntent，并且会清空主页面上面的其它页面）

(1)如果在 AndroidManifest.xml 中将 Activity C 的启动模式设置为 SingleTask，如果此时任务栈内已经存在 Activity C 的实例且未位于栈顶，当启动 Activity C 时，会将 Activity C 上方的实例全部出栈让其位于任务栈顶并 Activity C 中的 onNewIntent() 方法会被调用。

<img width="689" alt="image" src="https://user-images.githubusercontent.com/67937122/161176270-c158bbc6-1f32-4877-84d5-1ac44d42a03f.png">

(2)如果在 AndroidManifest.xml 中将 Activity C 的启动模式设置为 SingleTask，并且此时任务栈内并不存在 Activity C 的实例，当启动 Activity C 时，会创建一个崭新的 Activity C 实例在栈顶。

<img width="694" alt="image" src="https://user-images.githubusercontent.com/67937122/161176309-d396d77a-60ae-4b2e-9cbe-6952ea7b1697.png">

（四）singleInstance模式：

最特殊的模式，系统为该模式的活动分配一个独立的任务栈，该任务栈有且只有一个该活动实例。也就是说，如果已经创建过目标活动实例，那么将不会创建新的任务栈，而是唤醒之前创建过的活动实例。

(使用场景如闹铃提醒，将闹铃提醒与闹铃设置分离)

以singleInstance模式启动的Activity具有全局唯一性，即整个系统中只会存在一个这样的实例。

以singleInstance模式启动的Activity具有独占性，即它会独自占用一个任务，被他开启的任何activity都会运行在其他任务中。

被singleInstance模式的Activity开启的其他activity，能够开启一个新任务，但不一定开启新的任务，也可能在已有的一个任务中开启。取决于开启的activity的taskAffinity任务是否存在。

如果在 AndroidManifest.xml 中将 Activity E 的启动模式设置为 SingleInstance，并且任务栈内不存在 Activity E 的实例，当启动 Activity E 时，会在创建一个新的任务栈，并且栈内只有 Activity E 一个实例。

<img width="677" alt="image" src="https://user-images.githubusercontent.com/67937122/161176461-915e0a7e-8fd6-4ce5-ada3-c2d0f28ee648.png">

问1:如果此时基于上面的任务栈，从 Activity D 中开启一个启动模式为 Standard 的 Activity F，那任务栈会发生什么样的变化呢？请看下图：

此时 Activity E 依旧会独立的存在于自己的任务栈中，而新创建的 Activity F 将会和 Activity D 位于相同的任务栈的栈顶。

<img width="676" alt="image" src="https://user-images.githubusercontent.com/67937122/161176610-237798c9-5b8d-4992-99f9-1d36228cf451.png">


问2:如果启动模式为 SingleInstance 的 Activity E 已经独立存在于自己的任务栈中，此时再启动 Activity E， 则会复用已经创建的 Activity E 的实例，并且 Activity E 的 onNewIntent() 方法被调用。发生的变化如下图所示：

<img width="674" alt="image" src="https://user-images.githubusercontent.com/67937122/161176787-ccb1295c-f298-4010-a3cb-faa6a5103dcd.png">


问3:如果基于上面的任务栈，从 Activity E 中开启一个启动模式为 Standard 的 Activity F，那任务栈会发什么样的变化呢，请看下图：

<img width="670" alt="image" src="https://user-images.githubusercontent.com/67937122/161176946-3b5dc7f9-d1bc-4315-b7c9-5f047c941159.png">

因为 singleInstance 的属性是禁止与其他 Activities 共享任务栈，所以启动模式为 SingleInstance 的 Activity 启动其他 Activity 时会默认带有 FLAG_ACTIVITY_NEW_TASK 属性。所以 Activity E 启动 Activity F 后，最后会存在三个任务栈，Activity F 会单独存在于一个任务栈中







转载：

https://juejin.cn/post/6844903781486821389

https://mrfzh.github.io/2019/09/15/Activity-%E7%9A%84%E5%90%AF%E5%8A%A8%EF%BC%9A%E5%9B%9B%E7%A7%8D%E5%90%AF%E5%8A%A8%E6%A8%A1%E5%BC%8F%E5%8F%8A%E5%90%84%E7%A7%8D-FLAG/

https://www.jianshu.com/p/0a512e390dfa


