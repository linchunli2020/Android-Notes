Acivity启动过程一直是自己头疼理解的一部分，打算恶补一下。在了解Activity的启动过程前，先了解下Android系统机制的基础知识：

⭕️ zygote是什么？

zygote意为“受精卵”。Android是基于Linux系统的，而在Linux中，所有的进程都是由init进程直接或者间接fork出来的，zygote进程也不例外。

Launcher进程是什么？

Launcher进程是Android系统启动的第一个应用，也就是我们通常说的手机主页面，它也是异常普通的Android app。

在Android进程里，zygote是一个进程的名字。Android是基于Linux的，当手机开机的时候，linux的内核加载完成以后就会启动一个叫“init”的进程。在linux中，所有的进程都是由init进程fork出来的，zygote进程也不例外。

我们都知道，每一个app其实都是：

    一个单独的Dalvik虚拟机
    一个单独的进程
    
所以当系统里的第一个zygote进程运行以后，在这之后在开启app，就相当于开启一个新的进程。而为了实现资源共用和更快的启动速度，Android系统启动新进程的方式，是通过fork第一个zygote进程实现的。所以说，除了第一个zygote进程，其它应用所在的进程都是zygote进程的子进程，这也就是它称之为“受精卵”的原因。它能快速的分裂，并且产生遗传物质一样的细胞。


AMS是什么？

AMS，全称ActivityManagerService，服务端对象，负责系统中所有Activity，service，BroadcastRecevier和ContentProvider的管理。在SystemServier启动的时候，就会初始化AMS，从下面的代码中可以看出：
          public final class SystemServer {

            //zygote的主入口
              public static void main(String[] args) {
                  new SystemServer().run();
              }

              public SystemServer() {
                  // Check for factory test mode.
                  mFactoryTestMode = FactoryTest.getMode();
              }

              private void run() {

                ...ignore some code...

                //加载本地系统服务库，并进行初始化 
                  System.loadLibrary("android_servers");
                  nativeInit();

                  createSystemContext();

                  //初始化SystemServiceManager对象，下面的系统服务开启都需要调用SystemServiceManager.startService(Class<T>)
                  //这个方法通过反射来启动对应的服务
                  mSystemServiceManager = new SystemServiceManager(mSystemContext);

                  //开启服务
                  try {
                      startBootstrapServices();
                      startCoreServices();
                      startOtherServices();
                  } catch (Throwable ex) {
                      throw ex;
                  }

                  ...ignore some code...

              }

            //初始化系统上下文对象mSystemContext，mSystemContext实际上是一个ContextImpl对象
            //调用ActivityThread.systemMain()的时候，会调用ActivityThread.attach(true)
            private void createSystemContext() {
                  ActivityThread activityThread = ActivityThread.systemMain();
                  mSystemContext = activityThread.getSystemContext();
                  mSystemContext.setTheme(android.R.style.Theme_DeviceDefault_Light_DarkActionBar);
              }

            //在这里开启了几个核心的服务，因为这些服务之间相互依赖，所以都放在了这个方法里面。
            private void startBootstrapServices() {

              ...ignore some code...

              //初始化AMS
              mActivityManagerService = mSystemServiceManager.startService(
                          ActivityManagerService.Lifecycle.class).getService();
                  mActivityManagerService.setSystemServiceManager(mSystemServiceManager);

                  //初始化PowerManagerService，因为其他服务需要依赖这个Service，因此需要尽快的初始化
              mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);

                  // 现在电源管理已经开启，ActivityManagerService负责电源管理功能
                  mActivityManagerService.initPowerManagement();

                //初始化PackageManagerService
                mPackageManagerService = PackageManagerService.main(mSystemContext, mInstaller,
                     mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);

            ...ignore some code...

            }

          }


