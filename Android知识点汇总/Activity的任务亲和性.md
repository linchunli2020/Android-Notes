
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
