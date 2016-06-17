---
title: Android Activity启动过程
date: 2016-06-13 11:24:36
tags: [Android,Activity启动,startActivity]
categories: 编程 # 配置categories
---

## Android Activity的启动过程

### Android进程创建

应用程序的入口一般来说，我们都认为是Launcher Activity的onCreate方法，但追求根本，Android应用程序的入口应该是ActivityThread的main方法。

**ActivityThread类**：该类为应用程序的主线程类，所有的应用程序都有且仅有一个ActivityThread类，程序的入口为该类中的static main()函数。

**Activity类**：该类为APK（AndroidPackage，是一种通过AndroidSDK编译的工程打包成的安装程序文件）程序的一个最小运行单元，一个APK程序中可以包含多个Activity对象，ActivityThread主类会根据用户操作选择运行哪个Activity对象。

**ActivityManagerService类**：简称AMS，它的作用是管理所有应用程序中的Activity。

	//ActivityThread的main方法

    public static void main(String[] args) {
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        Security.addProvider(new AndroidKeyStoreProvider());

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        AsyncTask.init();

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

如上所示，其main方法中 初始化了并构建了主线程Looper。

至于，ActivityThread在是什么时候被调用，如下：

	// 在ActivityManagerService.java中：  

	Process.ProcessStartResult startResult = Process.start("android.app.ActivityThread",  
                    app.processName, uid, uid, gids, debugFlags,  
                    app.info.targetSdkVersion, null, null);  

对于应用程序的创建，最终通过 AMS 调度Process.start()静态方法来启动新程序,实际上最终将调用其main函数.

那么，应用程序是通过怎样的一个方式通知创建其进程的？

应用程序是用过向socket服务端写数据，把创建进程的请求通过socket通讯方式来让framework的进程孵化类zygote创建新进程。Zygote进程、SystemServer进程、各APK进程和创建进程的socket服务端/客户端的关系如下图所示：

![zygote](http://i.imgur.com/F1FDTyz.png)

>Android进程孵化环境
>
>当Android内核启动后，此时系统的状态和普通的Linux系统基本相同，通过配置Android中的init.rc文件，可以指定内核启动后都要执行什么程序，而在此配置文件中指定的之后所要启动的程序才是Android系统和普通Linux应用系统的区别。Android系统里init.rc中所启动的一个重要进程被称作zygote进程，也称为“种子进程”，从进程的角度来看，种子进程仅仅是一个Linux进程而已，它和一个只包含main()函数的C程序所产生的进程是同一个级别，但种子进程里面所运行的程序基本上就是Android内核的精华所在，其内部主要完成了两件事情。第一件事情是装载了一段程序代码，这些代码都是用C语言写的，这段代码的作用只是为了能够执行Java编译器编译出的字节码，功能类似Java虚拟机，在Android中称为Dalvik虚拟机。第二件事情必须基于第一件事情之后，即当Dalvik虚拟机代码初始化完成后，开始执行ZygoteInit.java类中的main()函数。ZygoteInit.java这个Jar包的目录位置信息也是在init.rc中进行配置的，是使用一个“zygote”标志符，Dalvik虚拟机就会从init.rc配置项的键值对中得到ZygoteInit类所在的Jar包，而这个Jar包正是Android的另一个核心--framework.jar。
>
>接下ZygoteInit类中main()函数所做的事情和Linux本身就没多大关系了，该main()函数中才刚刚开始启动Android的核心功能。首先加载一些类文件，这些类将作为以后所有其它Apk程序共享的类，接着，会创建一个Socket服务端，该服务端将用于通过Socket启动新进程。zygote进程被称为“种子”进程的原因就是，当其内部的Socket服务端收到启动新的Apk进程的请求时，会使用Linux的一个系统调用folk()函数从自身复制出一个新的进程，新进程和Zygote进程将共享已经装载的类，这些类都是在framework.jar中定义的。

	// ZygoteInit.java的main函数如下：  
	public static void main(String argv[]) {  
	    try {  
	        …  
	        registerZygoteSocket(); // 注册一个socket server来监听zygote命令  
	        preloadClasses();//预加载java class  
	        preloadResources();//预加载资源文件  
	        …  
			gc();/*初始化GC垃圾回收机制*/  
			if (argv[1].equals("true")) {  

				/* 通过main中传递过来的第二个参数startsystemserver=”true” 启动systemserver, 在startSystemServer()中会fork一个新的进程命名为system_server， 执行的是com.android.server包中的SystemServer.java文件中的main函数*/  
	
		        startSystemServer();
				 ///*************  
			} else if(…)   
			…
  
			if (ZYGOTE_FORK_MODE) {  
			     runForkMode();      /*将进入Zygote的子进程*/  
			} else {  
			     runSelectLoopMode();/* Zygote进程进入无限循环，不再返回。接下来的zygote将会作为一个孵化服务进程来运行。*/  
			}  
			closeServerSocket();  
		}  
		…  
	}  
         

简而言之：android程序进程创建的整个流程如下

* Application层的程序发起创建应用程序的命令；
* Ams调度框架通过framework发起socket通讯通知新进程创建；
* zygote孵化进程接收socket信息并调用内核创建新进程；

也就是说 ，最终是通过zygote fork一个应用进程。

综上所述，创建程序新进程的任务最关键就是zygote进程。Zygote进程起到一个承上启下的作用。对于framework，zygote进程接收上层应用通过socket发送过来的新进程创建命令，对于kernel而言，zygote进程主要调用了内核的fork()系统调用来进行新进程的创建，所以zygote在android系统中扮演一个非常重要的角色，是新进程创建的一个孵化器。

![process-flow](http://i.imgur.com/mDrhXtj.png)

Reference: 

* [Android 源码 repo](https://android.googlesource.com/ "https://android.googlesource.com/")
* [Android Anatomy and Physiology](https://www.google.com.hk/#q=Android+Anatomy+and+Physiology "https://www.google.com.hk/#q=Android+Anatomy+and+Physiology")
* 柯元旦《android内核剖析》电子工业出版社


### Activity启动

Android应用程序启动过程，即为LauncherActivity的启动过程，Android应用程序从Launcher启动 程序 流程如下所示：

	/***************************************************************** 
	  * Launcher通过Binder告诉ActivityManagerService， 
	  * 它将要启动一个新的Activity； 
	  ****************************************************************/  
	 Launcher.startActivitySafely->    
	 Launcher.startActivity->
    
	     //要求在新的Task中启动此Activity    
	     //intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)    
	     Activity.startActivity->    
	     Activity.startActivityForResult->    
	     Instrumentation.execStartActivity->    

	     // ActivityManagerNative.getDefault()返回AMS Proxy接口    
	     ActivityManagerNative.getDefault().startActivity->    
	     ActivityManagerProxy.startActivity->    
     
            ActivityManagerService.startActivity-> (AMS)    
    		ActivityManagerService.startActivityAsUser->     
     
     		ActivityStack.startActivityMayWait->    
    		ActivityStack.resolveActivity(获取ActivityInfo)    
       		//aInfo.name为main Activity,如：com.my.test.MainActivity    
       		//aInfo.applicationInfo.packageName为包名，如com.my.test    
     		ActivityStack.startActivityLocked->    
       		//ProcessRecord callerApp; 调用者即Launcher信息    
       		//ActivityRecord sourceRecord; Launcher Activity相关信息    
       		//ActivityRecord r=new ActivityRecord(...)，将要创建的Activity相关信息      
     		ActivityStack.startActivityUncheckedLocked->    
      		//Activity启动方式：ActivityInfo.LAUNCH_MULTIPLE/LAUNCH_SINGLE_INSTANCE/    
      		//             ActivityInfo.LAUNCH_SINGLE_TASK/LAUNCH_SINGLE_TOP)    
      		// 创建一个新的task,即TaskRecord,并保存在ActivityRecord.task中    
      		//r.setTask(new TaskRecord(mService.mCurTask, r.info, intent), null, true)    
      		// 把新创建的Activity放在栈顶       
      		ActivityStack.startActivityLocked->    
      		ActivityStack.resumeTopActivityLocked->    
      		ActivityStack.startPausingLocked (使Launcher进入Paused状态)->      
   
      /***************************************************************** 
       * AMS通过Binder通知Launcher进入Paused状态 
       ****************************************************************/  
       ApplicationThreadProxy.schedulePauseActivity->     
       //private class ApplicationThread extends ApplicationThreadNative    
       ApplicationThread.schedulePauseActivity->    
     
       ActivityThread.queueOrSendMessage->    
      
        // 调用Activity.onUserLeaveHint    
        // 调用Activity.onPause    
        // 通知activity manager我进入了pause状态    
        ActivityThread.handlePauseActivity->    
   
        /***************************************************************** 
         * Launcher通过Binder告诉AMS，它已经进入Paused状态 
         ****************************************************************/  
        ActivityManagerProxy.activityPaused->    
        ActivityManagerService.activityPaused->    
        ActivityStack.activityPaused->(把Activity状态修改为PAUSED)    
        ActivityStack.completePauseLocked->    
       
        // 参数为代表Launcher这个Activity的ActivityRecord    
        // 使用栈顶的Activity进入RESUME状态    
        ActivityStack.resumeTopActivityLokced->    
        //topRunningActivityLocked将刚创建的放于栈顶的activity取回来    
        // 即在ActivityStack.startActivityUncheckedLocked中创建的    
   
        /***************************************************************** 
         * AMS创建一个新的进程，用来启动一个ActivityThread实例， 
         * 即将要启动的Activity就是在这个ActivityThread实例中运行 
         ****************************************************************/  
        ActivityStack.startSpecificActivityLocked->    
     
         // 创建对应的ProcessRecord    
         ActivityManagerService.startProcessLocked->    
             
	          // 启动一个新的进程    
	          // 新的进程会导入android.app.ActivityThread类，并且执行它的main函数,    
	          // 即实例化ActivityThread, 每个应用有且仅有一个ActivityThread实例    
	          Process.start("android.app.ActivityThread",...)->    
   
	          // 通过zygote机制创建一个新的进程    
	          Process.startViaZygote->    
       
	          // 这个函数在进程中创建一个ActivityThread实例，然后调用    
	          // 它的attach函数，接着就进入消息循环    
	          ActivityThread.main->    
   
          /***************************************************************** 
           * ActivityThread通过Binder将一个ApplicationThread类的Binder对象 
           * 传递给AMS，以便AMS通过此Binder对象来控制Activity整个生命周期 
           ****************************************************************/  
          ActivityThread.attach->    
          IActivityManager.attachApplication(mAppThread)->    
          ActivityManagerProxy.attachApplication->    
          ActivityManagerService.attachApplication->    
     
          // 把在ActivityManagerService.startProcessLocked中创建的ProcessRecord取出来    
          ActivityManagerService.attachApplicationLocked->    
   
          /***************************************************************** 
           * AMS通过Binder通知ActivityThread一切准备OK,它可以真正启动新的Activity了 
           ****************************************************************/              
          // 真正启动Activity    
          ActivityStack.realStartActivityLocked->    
          ApplicationThreadProxy.scheduleLaunchActivity->    
          ApplicationThread.scheduleLaunchActivity->    
          ActivityThread.handleLaunchActivity->    
            // 加载新的Activity类，并执行它的onCreate    
            ActivityThread.performLaunchActivity    
               /*---------------------------------------------
			   1) Instrumentation.newActivity: 加载新类，即创建Activity对象；  
               2) ActivityClientRecord.packageInfo.makeApplication：创建Application对象；  
                  <LoadedApk.makeApplication>  
               3) Activity.attach(Context context, ActivityThread aThread,  
                     Instrumentation instr, IBinder token, int ident,  
                     Application application, Intent intent, ActivityInfo info,  
                     CharSequence title, Activity parent, String id,  
                     NonConfigurationInstances lastNonConfigurationInstances,  
                     Configuration config)：把Application attach到Activity, 即把Activtiy  
                                            相关信息设置到新创建的Activity中  
               4) Instrumentation.callActivityOnCreate：调用onCreate；
				---------------------------------------------*/    
       
            // 使用Activity进入RESUMED状态，并调用onResume    
            ActivityThread.handleResumeActivity    


从Activity的启动流程中，我们可以看到几个比较重要的类：

#### Instrumentation

顾名思义，仪器仪表，用于在应用程序中进行“测量”和“管理”工作。一个应用程序中只有一个Instrumentation实例对象，且每个Activity都有此对象的引用。Instrumentation将在任何应用程序运行前初始化，可以通过它监测系统与应用程序之间的所有交互，即类似于在系统与应用程序之间安装了个“窃听器”。

当ActivityThread 创建(callActivityOnCreate)、暂停、恢复某个Activity时，通过调用此对象的方法来实现，如：

1) 创建: callActivityOnCreate 

2) 暂停: callActivityOnPause

3) 恢复: callActivityOnResume

#### ActivityManagerService

AMS提供的主要功能：

* 统一调度各应用程序的Activities；
* 内存管理
* 进程管理

##### 启动一个Activity的方式

* 在应用程序中调用startActivity()启动指定Activity；
* 在Launcher中点击一个应用程序的icon，启动新的Activity；
* 按“back”返回键，结束当前Activity，返回到上一个Activity；
* 长按“Home”键显示当前正在运行的程序列表，从中选择一个要启动的Activity；

这四种方式最终都会按第一种方式进行Activity的启动，区别主要在于前端消息的接受和处理上不同。


#### ActivityStack

ActivityManagerService使用它来管理系统中所有的Activities的状态，Activities使用stack的方式进行管理。它是真正负责做事的家伙，很勤快的，但外界无人知道！

#### TaskRecord类

ActivityManagerService中使用任务的概念来确保Activity启动和退出的顺序。TaskRecord中的几个重要变量如下：

		final int taskId;       // 每个任务的标识.
		Intent intent;          // 创建该任务时对应的intent
		int numActivities;   //该任务中的Activity数目
		final ArrayList<ActivityRecord> mActivities = new ArrayList<ActivityRecord>();  //按照出现的先后顺序列出该任务中的所有Activity

#### ActivityRecord

ActivityManagerService使用ActivityRecord数据类来保存每个Activity的信息，ActivityRecord类基于IApplicationToken.Stub类，也是一个Binder,所以可以被IPC调用。

主要包含的变量有：

* 环境信息：Activity的工作环境，比如进程名称、文件路径、数据路径、图标、主题等，这些信息一般是固定的，比如以下变量：

		final String packageName; // the package implementing intent's component
		final String processName; // process where this component wants to run
		final String baseDir;   // where activity source (resources etc) located
		final String resDir;   // where public activity source (public resources etc) located
		final String dataDir;   // where activity data should go
		int theme;              // resource identifier of activity's theme.
		int realTheme;          // actual theme resource we will use, never 0.

* 运行状态数据信息：如idle、stop、finishing等，一般为boolean类型，如下

		boolean haveState;      // have we gotten the last activity state?
		boolean stopped;        // is activity pause finished?
		boolean delayedResume;  // not yet resumed because of stopped app switches?
		boolean finishing;      // activity in pending finish list?
		boolean configDestroy;  // need to destroy due to config change?

#### ProcessRecord---记录了一个进程的相关信息。

~\frameworks\base\services\java\com\android\server\am

一般情况下，一个应用程序（APK）运行会对应一个进程，而ProcessRecord就是用来记录一个进程的相关信息，主要包含以下几个方面：

* 进程文件信息：与该进程对应的APK文件的内部信息，如：

		final ApplicationInfo info; // all about the first app in the process
	    final String processName;   // name of the process
	    final ArrayMap<String, ProcessStats.ProcessState> pkgList 
	            = new ArrayMap<String, ProcessStats.ProcessState>();   //保存进程中所有APK文件包名

* 进程的内存状态信息：用于Linux系统的out of memory(OOM)情况的处理，当发生内存紧张时，Linux系统会根据进程的内存状态信息杀掉低优先级的进程，包括的变量有：

		int maxAdj;                 // Maximum OOM adjustment for this process
        int curRawAdj;              // Current OOM unlimited adjustment for this process
        int setRawAdj;              // Last set OOM unlimited adjustment for this process
        int curAdj;                 // Current OOM adjustment for this process
        int setAdj;                 // Last set OOM adjustment for this process
   变量中Adj的含义是调整值（adjustment）

* 进程中包含的Activity、ContentProvider、Service、Receiver等，如下

		final ArrayList<ActivityRecord> activities = new ArrayList<ActivityRecord>();
		final ArraySet<ServiceRecord> services = new ArraySet<ServiceRecord>();
		final ArraySet<ServiceRecord> executingServices = new ArraySet<ServiceRecord>();
		final ArraySet<ConnectionRecord> connections = new ArraySet<ConnectionRecord>();
		final ArraySet<ReceiverList> receivers = new ArraySet<ReceiverList>();
		final ArrayMap<String, ContentProviderRecord> pubProviders = new ArrayMap<String,             ContentProviderRecord>();
		final ArrayList<ContentProviderConnection> conProviders = new ArrayList<ContentProviderConnection>();

####  IApplicationThread接口AMS->Application

IApplicationThread为AMS作为客户端访问Application服务器端的Binder接口。当创建Application时，将把此Binder对象传递给AMS，然后AMS把它保存在mProcessNames.ProcessRecord.thread中。当需要通知Application工作时，则调用IApplicationThread中对应的接口函数。

![IApplicationThread](http://i.imgur.com/E67npgC.png)

Reference:

[http://www.kancloud.cn/digest/androidframeworks/127785](http://www.kancloud.cn/digest/androidframeworks/127785 "http://www.kancloud.cn/digest/androidframeworks/127785")

#### 关于Activity启动的流程图

![launcher-flow](http://i.imgur.com/Vie24ZR.jpg)