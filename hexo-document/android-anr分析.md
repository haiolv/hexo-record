---
title: Android ANR 分析
date: 2016-06-13 11:24:36
tags: Android
categories: 编程 # 配置categories
---


* 什么是ANR
* ANR产生的原因
* ANR流程分析
* 发生ANR如何定位
* 如何避免和解决ANR

refer:[http://developer.android.com/intl/zh-cn/training/articles/perf-anr.html#anr](http://developer.android.com/intl/zh-cn/training/articles/perf-anr.html#anr "keeping your app response")

**1.什么是ANR**

* ANR:Application Not Responding，即应用无响应
* 为用户在主线程长时间被阻塞是提供交互，提高用户体验；
* android系统自身的一种检测机制

**1.1 ANR的类型**

ANR一般有三种类型：

1. KeyDispatchTimeout(5 seconds) --主要类型按键或触摸事件在特定时间内无响应

2. BroadcastTimeout(10 seconds) --BroadcastReceiver在特定时间内无法处理完成

3. ServiceTimeout(20 seconds) --小概率类型 Service在特定的时间内无法处理完成

>* No response to an input event (such as key press or screen touch events) within 5 seconds.
* A BroadcastReceiver hasn't finished executing within 10 seconds.

**1.1.1 KeyDispatchTimeout**

Akey or touch event was not dispatched within the specified time（按键或触摸事件在特定时间内无响应）

具体的超时时间的定义在framework下的ActivityManagerService.java

//How long we wait until we timeout on key dispatching.

staticfinal int KEY_DISPATCHING_TIMEOUT = 5*1000


**2.ANR产生的原因**

超时时间的计数一般是从按键分发给app开始。超时的原因一般有两种：

(1)当前的事件没有机会得到处理（即UI线程正在处理前一个事件，没有及时的完成或者looper被某种原因阻塞住了）

(2)当前的事件正在处理，但没有及时完成

(3)需要满足的条件：
	
* 主线程：只有应用的主线程响应超时才会导致ANR
* 超时时间（Timeout）:导致ANR的原因不同，系统限定的时间也不同，但只要超过时间上闲，系统就会发出Signal 3，产生ANR
* 输入操作/特定操作：输入操作指按键、触屏等操作，特定操作指广播、服务中执行耗时方法等。

**summary：**

引起ANR的根本原因总的来说如下：

* 应用程序自身引起的

	例如，主线程阻塞（block）、挂起、死锁、死循环、执行耗时操作等
* 其他应用进程引起的

	例如，其他进程的CPU占用率过高、某一时刻系统CPU负载过高等，都会导致当前进程无法抢到CPU时间片。
* IO wait
	
	文件读写耗时较久，占用CPU时间片较长，导致ANR

提示：如果你在解决其他问题时也需要查看Java进程中各个线程的函数堆栈信息，就可以使用向目标进程发送SIGNAL_QUIT（3）这个技巧。其实这个名称有点误导，它并不会让进程真正退出。
**
3.ANR出现流程分析**

* 输入时间响应超时导致ANR流程

	在系统输入管理服务进程（InputManagerService）中有一个线程（InputDispathcerThread）专门管理输入事件的分发，在该线程处理输入事件的过程中，回调用InputDispatcher对象方法不断的检测处理过程是否超时，一旦超时，则会通过一些列的回调调用InputMethod对象的notifyANR方法，其会最终出发AMS中handler对象的SHOW_NOT_RESPONDING_MSG这个事件，显示ANR对话框。

* 广播发生ANR流程

	广播分为三类：普通的，有序的，异步的。只有有序（ordered）的广播才会发生超时，而在AndroidManifest中注册的广播都会被当做有序广播来处理，会被放在广播的队列中串行处理。AMS在处理广播队列时，会设置一个超时时间，当处理一个广播达到超时时间的限制时，就会触发BroadcastQueue类对象的processNextBroadcast方法来判断是否超时，如果超时，就会终止该广播，触发ANR对话框。

* UI线程

    UI线程主要包括如下：

	* Activity:onCreate(), onResume(), onDestroy(), onKeyDown(), onClick(),etc声明周期方法里

    2. AsyncTask: onPreExecute(), onProgressUpdate(), onPostExecute(), onCancel,etc这些异步更改UI界面的方法里

    3. Mainthread handler: handleMessage(), post*(runnable r), getMainLooper(),etc通过handler发送消息到主线程的looper,即占用主线程looper的


**4.发生ANR如何定位**

系统发生ANR的时候，一般会以以下三种方式记录

* logcat日志；
* /data/anr/traces.txt
* dropBox服务；

**4.1 通过logcat文件分析**

logcat文件：

	04-01 13:12:11.572 I/InputDispatcher( 220): Application is not responding:Window{2b263310com.android.email/com.android.email.activity.SplitScreenActivitypaused=false}.
    5009.8ms since event, 5009.5ms since waitstarted
    04-0113:12:11.572 I/WindowManager( 220): Input event 
    dispatching timedout sending 
    tocom.android.email/com.android.email.activity.SplitScreenActivity
 
    04-01 13:12:14.123 I/Process( 220): Sending signal. PID: 21404 SIG: 3---发生ANR的时间和生成trace.txt的时间
    04-01 13:12:14.123 I/dalvikvm(21404):threadid=4: reacting to 
    signal 3 
    ……
    04-0113:12:15.872 E/ActivityManager( 220): ANR in 
    com.android.email(com.android.email/.activity.SplitScreenActivity)
    04-0113:12:15.872 E/ActivityManager( 220): 
    Reason:keyDispatchingTimedOut  -----ANR的类型
    04-0113:12:15.872 E/ActivityManager( 220): Load: 8.68 / 8.37 / 8.53 --CPU的负载情况
    04-0113:12:15.872 E/ActivityManager( 220): CPUusage from 4361ms to 699ms ago ----CPU在ANR发生前的使用情况
 
    04-0113:12:15.872 E/ActivityManager( 220): 5.5%21404/com.android.email: 1.3% user + 4.1% kernel / faults:
    10 minor
    04-0113:12:15.872 E/ActivityManager( 220): 4.3%220/system_server: 2.7% user + 1.5% kernel / faults: 11
    minor 2 major
    04-0113:12:15.872 E/ActivityManager( 220): 0.9%52/spi_qsd.0: 0% user + 0.9% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 0.5%65/irq/170-cyttsp-: 0% user + 0.5% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 0.5%296/com.android.systemui: 0.5% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 100%TOTAL: 4.8% user + 7.6% kernel + 87% iowait
    04-0113:12:15.872 E/ActivityManager( 220): CPUusage from 3697ms to 4223ms later:-- ANR后CPU的使用量
    04-0113:12:15.872 E/ActivityManager( 220): 25%21404/com.android.email: 25% user + 0% kernel / faults: 191 minor
    04-0113:12:15.872 E/ActivityManager( 220): 16% 21603/__eas(par.hakan: 16% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 7.2% 21406/GC: 7.2% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 1.8% 21409/Compiler: 1.8% user + 0% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 5.5%220/system_server: 0% user + 5.5% kernel / faults: 1 minor
    04-0113:12:15.872 E/ActivityManager( 220): 5.5% 263/InputDispatcher: 0% user + 5.5% kernel
    04-0113:12:15.872 E/ActivityManager( 220): 32%TOTAL: 28% user + 3.7% kernel

从LOG可以看出：

* ANR的类型
	
	上面的日志课可以看出anr的类型为keyDispatchingTimedOut

* CPU的使用情况
	CPU的使用情况包括发生ANR之前CPU的负载情况 
	如果CPU使用量接近100%，说明当前设备很忙，有可能是CPU饥饿导致了ANR
	从上面看出 total CPU达到100%,并且 io wait占到87%,所以 说明在主线程进行io操作。

如果CPU使用量很少，说明主线程被BLOCK了

如果IOwait很高，说明ANR有可能是主线程在进行I/O操作造成的


* Thread的状态
    
    ThreadState (defined at “dalvik/vm/thread.h “)  
    THREAD_UNDEFINED = -1, /* makes enum compatible with int32_t */  
    THREAD_ZOMBIE = 0, /* TERMINATED */  
    THREAD_RUNNING = 1, /* RUNNABLE or running now */  
    THREAD_TIMED_WAIT = 2, /* TIMED_WAITING in Object.wait() */  
    THREAD_MONITOR = 3, /* BLOCKED on a monitor */  
    THREAD_WAIT = 4, /* WAITING in Object.wait() */  
    THREAD_INITIALIZING= 5, /* allocated, not yet running */  
    THREAD_STARTING = 6, /* started, not yet on thread list */  
    THREAD_NATIVE = 7, /* off in a JNI native method */  
    THREAD_VMWAIT = 8, /* waiting on a VM resource */  
    THREAD_SUSPENDED = 9, /* suspended, usually by GC or debugger */  

Thread.java中定义的状态 &nbsp;&nbsp; &nbsp;Thread.cpp中定义的状态&nbsp;&nbsp; &nbsp;说明


TERMINATED&nbsp;&nbsp; &nbsp;ZOMBIE&nbsp;&nbsp; &nbsp;线程死亡，终止运行

RUNNABLE&nbsp;&nbsp; &nbsp;RUNNING/RUNNABLE&nbsp;&nbsp; &nbsp;线程可运行或正在运行

TIMED_WAITING&nbsp;&nbsp; &nbsp;TIMED_WAIT&nbsp;&nbsp; &nbsp;执行了带有超时参数的wait、sleep或join函数

BLOCKED&nbsp;&nbsp; &nbsp;MONITOR&nbsp;&nbsp; &nbsp;线程阻塞，等待获取对象锁

WAITING&nbsp;&nbsp; &nbsp;WAIT&nbsp;&nbsp; &nbsp;执行了无超时参数的wait函数

NEW&nbsp;&nbsp; &nbsp;INITIALIZING&nbsp;&nbsp; &nbsp;新建，正在初始化，为其分配资源

NEW&nbsp;&nbsp; &nbsp;STARTING&nbsp;&nbsp; &nbsp;新建，正在启动

RUNNABLE&nbsp;&nbsp; &nbsp;NATIVE&nbsp;&nbsp; &nbsp;正在执行JNI本地函数

WAITING&nbsp;&nbsp; &nbsp;VMWAIT&nbsp;&nbsp; &nbsp;正在等待VM资源

RUNNABLE&nbsp;&nbsp; &nbsp;SUSPENDED&nbsp;&nbsp; &nbsp;线程暂停，通常是由于GC或debug被暂停

?&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;UNKNOWN&nbsp;&nbsp; &nbsp;未知状态


traces.txt文件

	DALVIK THREADS:
	(mutexes: tll=0 tsl=0 tscl=0 ghl=0)
	
	"main" prio=5 tid=1 VMWAIT
	  | group="main" sCount=1 dsCount=0 obj=0x41937700 self=0x400bc0b0
	  | sysTid=9437 nice=0 sched=0/0 cgrp=apps handle=1074757424
	  | schedstat=( 0 0 0 ) utm=2 stm=1 core=0
	  #00  pc 0000dca0  /system/lib/libc.so (__futex_syscall3+8)
	  #01  pc 000121cc  /system/lib/libc.so
	  #02  pc 00053aff  /system/lib/libdvm.so (dvmLockThreadList(Thread*)+38)
	  #03  pc 0005427b  /system/lib/libdvm.so (dvmCreateInternalThread(long*, char const*, void* (*)(void*), void*)+134)
	  #04  pc 00047427  /system/lib/libdvm.so (dvmInitAfterZygote()+274)
	  #05  pc 0006580b  /system/lib/libdvm.so
	  #06  pc 0006592b  /system/lib/libdvm.so
	  #07  pc 00029020  /system/lib/libdvm.so
	  #08  pc 0002d7e8  /system/lib/libdvm.so (dvmInterpret(Thread*, Method const*, JValue*)+180)
	  #09  pc 0005fed5  /system/lib/libdvm.so (dvmCallMethodV(Thread*, Method const*, Object*, bool, JValue*, std::__va_list)+272)
	  #10  pc 0004aee7  /system/lib/libdvm.so
	  #11  pc 00048c75  /system/lib/libandroid_runtime.so
	  #12  pc 00049691  /system/lib/libandroid_runtime.so (android::AndroidRuntime::start(char const*, char const*)+368)
	  #13  pc 00000dcf  /system/bin/app_process
	  #14  pc 00017193  /system/lib/libc.so (__libc_init+38)
	  #15  pc 00000b34  /system/bin/app_process
	  at dalvik.system.Zygote.nativeForkAndSpecialize(Native Method)
	  at dalvik.system.Zygote.forkAndSpecialize(Zygote.java:117)
	  at com.android.internal.os.ZygoteConnection.runOnce(ZygoteConnection.java:231)
	  at com.android.internal.os.ZygoteInit.runSelectLoopMode(ZygoteInit.java:669)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:566)
	  at dalvik.system.NativeStart.main(Native Method)


logcat文件：

	11-05 13:31:39.511 E/ActivityManager(  498): ANR in com.boyaa.engineddz.main
	
	11-05 13:31:39.511 E/ActivityManager(  498): Reason: Executing service com.boyaa.engineddz.main/com.boyaa.boyaaad.service.FloatWindowService
	
	11-05 13:31:39.511 E/ActivityManager(  498): Load: 1.94 / 1.33 / 0.67
	
	11-05 13:31:39.511 E/ActivityManager(  498): CPU usage from 9953ms to 0ms ago:
	
	11-05 13:31:39.511 E/ActivityManager(  498):   32% 11780/com.boyaa.engineddz.main: 30% user + 2.3% kernel / faults: 125 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   9.3% 166/surfaceflinger: 3.8% user + 5.5% kernel / faults: 1250 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   3.3% 11818/com.boyaa.engineddz.main: 0% user + 3.3% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   2.4% 11817/com.boyaa.engineddz.main: 0.4% user + 2% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   2.3% 737/com.android.systemui: 1.9% user + 0.4% kernel / faults: 71 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   2.2% 498/system_server: 1.2% user + 1% kernel / faults: 46 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.7% 11523/com.boyaa.engineddz.main:pushservice: 0% user + 0.7% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.7% 11549/com.boyaa.engineddz.main:dservice_v1: 0.4% user + 0.3% kernel / faults: 182 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.6% 11799/com.boyaa.engineddz.main: 0.4% user + 0.2% kernel / faults: 524 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.5% 126/ueventd: 0% user + 0.5% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.5% 163/netd: 0.2% user + 0.3% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.5% 11507/com.boyaa.engineddz.main:dservice_v1: 0.2% user + 0.3% kernel / faults: 2 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.4% 159/vold: 0.1% user + 0.3% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.3% 6070/kworker/0:5: 0% user + 0.3% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.3% 10092/kworker/u:2: 0% user + 0.3% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.2% 262/adbd: 0% user + 0.2% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.2% 9849/kworker/0:0: 0% user + 0.2% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.2% 10291/kworker/u:3: 0% user + 0.2% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.2% 11559/com.boyaa.engineddz.main:pushservice: 0% user + 0.2% kernel / faults: 210 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0% 110/file-storage: 0% user + 0% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.1% 251/sdcard: 0% user + 0.1% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.1% 522/flush-179:0: 0% user + 0.1% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0% 1212/com.tencent.mobileqq:MSF: 0% user + 0% kernel / faults: 6 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0% 9022/com.boyaa.engineddz.main: 0% user + 0% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.1% 10309/kworker/2:2: 0% user + 0.1% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.1% 11753/kworker/3:2: 0.1% user + 0% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   0.1% 11758/logcat: 0% user + 0.1% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 22% TOTAL: 10% user + 4.5% kernel + 7.5% iowait
	
	11-05 13:31:39.511 E/ActivityManager(  498): CPU usage from 2161ms to 2685ms later:
	
	11-05 13:31:39.511 E/ActivityManager(  498):   45% 11780/com.boyaa.engineddz.main: 43% user + 1.8% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 43% 11831/Thread-555: 41% user + 1.8% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   9.2% 498/system_server: 3.7% user + 5.5% kernel / faults: 16 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498): 7.4% 515/ActivityManager: 1.8% user + 5.5% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.8% 507/FinalizerDaemon: 1.8% user + 0% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.8% 642/UEventObserver: 1.8% user + 0% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   7.4% 166/surfaceflinger: 5.5% user + 1.8% kernel / faults: 66 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.8% 452/SurfaceFlinger: 0% user + 1.8% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.8% 463/SurfaceFlinger: 0% user + 1.8% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.8% 464/EventThread: 0% user + 1.8% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   2.3% 737/com.android.systemui: 1.1% user + 1.1% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 2.3% 737/ndroid.systemui: 1.1% user + 1.1% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   2.8% 11817/com.boyaa.engineddz.main: 0% user + 2.8% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.4% 11817/.engineddz.main: 0% user + 1.4% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.4% 11819/.engineddz.main: 0% user + 1.4% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   1.4% 11799/com.boyaa.engineddz.main: 1.4% user + 0% kernel / faults: 53 minor
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.4% 11800/.engineddz.main: 1.4% user + 0% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498):   1.4% 11818/com.boyaa.engineddz.main: 0% user + 1.4% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.4% 11818/.engineddz.main: 0% user + 1.4% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 1.4% 11820/.engineddz.main: 0% user + 1.4% kernel
	
	11-05 13:31:39.511 E/ActivityManager(  498): 28% TOTAL: 14% user + 4.3% kernel + 9.5% iowai

traces.txt文件：

	----- pid 498 at 2015-11-05 13:32:52 -----
	Cmd line: system_server
	
	DALVIK THREADS:
	(mutexes: tll=0 tsl=0 tscl=0 ghl=0)
	
	"main" prio=5 tid=1 NATIVE
	  | group="main" sCount=1 dsCount=0 obj=0x41937700 self=0x400bc0b0
	  | sysTid=498 nice=0 sched=0/0 cgrp=apps handle=1074757424
	  | schedstat=( 0 0 0 ) utm=669 stm=136 core=1
	  #00  pc 0000cb90  /system/lib/libc.so (__ioctl+8)
	  #01  pc 00027fed  /system/lib/libc.so (ioctl+16)
	  #02  pc 00016f61  /system/lib/libbinder.so (android::IPCThreadState::talkWithDriver(bool)+124)
	  #03  pc 00017653  /system/lib/libbinder.so (android::IPCThreadState::joinThreadPool(bool)+154)
	  #04  pc 000011d9  /system/lib/libsystem_server.so (system_init+312)
	  #05  pc 0001fb70  /system/lib/libdvm.so (dvmPlatformInvoke+112)
	  #06  pc 0004e8b9  /system/lib/libdvm.so (dvmCallJNIMethod(unsigned int const*, JValue*, Method const*, Thread*)+360)
	  #07  pc 00029020  /system/lib/libdvm.so
	  #08  pc 0002d7e8  /system/lib/libdvm.so (dvmInterpret(Thread*, Method const*, JValue*)+180)
	  #09  pc 0006017f  /system/lib/libdvm.so (dvmInvokeMethod(Object*, Method const*, ArrayObject*, ArrayObject*, ClassObject*, bool)+374)
	  #10  pc 00067125  /system/lib/libdvm.so
	  #11  pc 00029020  /system/lib/libdvm.so
	  #12  pc 0002d7e8  /system/lib/libdvm.so (dvmInterpret(Thread*, Method const*, JValue*)+180)
	  #13  pc 0005fed5  /system/lib/libdvm.so (dvmCallMethodV(Thread*, Method const*, Object*, bool, JValue*, std::__va_list)+272)
	  #14  pc 0004aee7  /system/lib/libdvm.so
	  #15  pc 00048c75  /system/lib/libandroid_runtime.so
	  #16  pc 00049691  /system/lib/libandroid_runtime.so (android::AndroidRuntime::start(char const*, char const*)+368)
	  #17  pc 00000dcf  /system/bin/app_process
	  #18  pc 00017193  /system/lib/libc.so (__libc_init+38)
	  #19  pc 00000b34  /system/bin/app_process
	  at com.android.server.SystemServer.init1(Native Method)
	  at com.android.server.SystemServer.main(SystemServer.java:983)
	  at java.lang.reflect.Method.invokeNative(Native Method)
	  at java.lang.reflect.Method.invoke(Method.java:511)
	  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:804)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:571)
	  at dalvik.system.NativeStart.main(Native Method)

	提示正在执行JNI层函数，导致这一步极有可能就是native层内存不够了，就要通过dumpsys meminfo 来查看下

logcat文件：

	11-05 15:32:55.465 E/ActivityManager(  498): ANR in com.boyaa.engineddz.main
	
	11-05 15:32:55.465 E/ActivityManager(  498): Reason: Broadcast of Intent { act=com.igexin.sdk.action.lKooQgpZeM7hxbzRkb34f4 flg=0x10 cmp=com.boyaa.engineddz.main/com.boyaa.godsdk.core.GetuiReceiver (has extras) }
	
	11-05 15:32:55.465 E/ActivityManager(  498): Load: 0.42 / 0.27 / 0.25
	
	11-05 15:32:55.465 E/ActivityManager(  498): CPU usage from 6489ms to 25ms ago:
	
	11-05 15:32:55.465 E/ActivityManager(  498):   4.6% 16150/com.boyaa.engineddz.main: 0.3% user + 4.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   4.4% 16149/com.boyaa.engineddz.main: 0.1% user + 4.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   2.6% 498/system_server: 1.3% user + 1.2% kernel / faults: 61 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   2% 16107/com.boyaa.engineddz.main: 0.3% user + 1.7% kernel / faults: 4 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1.7% 16126/com.boyaa.engineddz.main: 0.4% user + 1.2% kernel / faults: 264 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1.5% 737/com.android.systemui: 1% user + 0.4% kernel / faults: 2 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1% 16221/com.boyaa.engineddz.main:pushservice: 0.4% user + 0.6% kernel / faults: 151 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.9% 16173/com.boyaa.engineddz.main:dservice_v1: 0% user + 0.9% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.9% 16188/com.boyaa.engineddz.main:pushservice: 0.4% user + 0.4% kernel / faults: 2 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.7% 166/surfaceflinger: 0.1% user + 0.6% kernel / faults: 75 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.7% 14398/kworker/0:0: 0% user + 0.7% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.7% 16212/com.boyaa.engineddz.main:dservice_v1: 0.3% user + 0.4% kernel / faults: 112 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.3% 248/sensors.qcom: 0% user + 0.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.3% 262/adbd: 0% user + 0.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.3% 16197/kworker/0:2: 0% user + 0.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 159/vold: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 163/netd: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 709/MC_Thread: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 719/wpa_supplicant: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 8772/com.boyaa.engineddz.main: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 9486/com.boyaa.engineddz.main: 0% user + 0% kernel / faults: 1 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 15494/kworker/0:1: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.1% 15928/kworker/3:0: 0% user + 0.1% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0% 15930/kworker/1:0: 0% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.1% 15934/kworker/u:2: 0% user + 0.1% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   0.1% 16104/logcat: 0.1% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498): 5.6% TOTAL: 1.1% user + 4% kernel + 0.4% iowait
	
	11-05 15:32:55.465 E/ActivityManager(  498): CPU usage from 2311ms to 2839ms later:
	
	11-05 15:32:55.465 E/ActivityManager(  498):   12% 498/system_server: 5.5% user + 7.4% kernel / faults: 36 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):     7.4% 515/ActivityManager: 1.8% user + 5.5% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     3.7% 507/FinalizerDaemon: 3.7% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     1.8% 689/WifiStateMachin: 1.8% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     1.8% 12027/Binder_D: 1.8% user + 0% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   2.3% 262/adbd: 0% user + 2.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     2.3% 262/adbd: 0% user + 2.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   2.3% 737/com.android.systemui: 2.3% user + 0% kernel / faults: 2 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):     2.3% 737/ndroid.systemui: 1.1% user + 1.1% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   3.6% 16149/com.boyaa.engineddz.main: 0% user + 3.6% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     3.6% 16149/.engineddz.main: 0% user + 3.6% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     1.8% 16156/.engineddz.main: 0% user + 1.8% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1.3% 15494/kworker/0:1: 0% user + 1.3% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1.4% 16126/com.boyaa.engineddz.main: 0% user + 1.4% kernel / faults: 43 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1.8% 16150/com.boyaa.engineddz.main: 0% user + 1.8% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     1.8% 16150/.engineddz.main: 0% user + 1.8% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):     1.8% 16152/.engineddz.main: 0% user + 1.8% kernel
	
	11-05 15:32:55.465 E/ActivityManager(  498):   1.4% 16212/com.boyaa.engineddz.main:dservice_v1: 0% user + 1.4% kernel / faults: 18 minor
	
	11-05 15:32:55.465 E/ActivityManager(  498): 9% TOTAL: 3.3% user + 5.6% kernel

traces.txt文件

	----- pid 16107 at 2015-11-05 15:32:52 -----
	Cmd line: com.boyaa.engineddz.main
	
	DALVIK THREADS:
	(mutexes: tll=0 tsl=0 tscl=0 ghl=0)
	
	"main" prio=5 tid=1 MONITOR
	  | group="main" sCount=1 dsCount=0 obj=0x41937700 self=0x400bc0b0
	  | sysTid=16107 nice=-15 sched=0/0 cgrp=apps handle=1074757424
	  | schedstat=( 0 0 0 ) utm=475 stm=38 core=3
	  at java.lang.ProcessManager.exec(ProcessManager.java:~206)
	  - waiting to lock <0x4252a508> (a java.util.HashMap) held by tid=14 (Thread-815)
	  at java.lang.ProcessBuilder.start(ProcessBuilder.java:195)
	  at com.tencent.stat.common.l.b((null):-1)
	  at com.tencent.stat.common.k.z((null):-1)
	  at com.tencent.stat.common.c.<init>((null):-1)
	  at com.tencent.stat.common.c.<init>((null):-1)
	  at com.tencent.stat.common.a.a((null):-1)
	  at com.tencent.stat.common.a.<init>((null):-1)
	  at com.tencent.stat.a.k.<init>((null):-1)
	  at com.tencent.stat.StatService.d((null):-1)
	  at com.tencent.stat.StatService.a((null):-1)
	  at com.tencent.stat.StatService.startStatService((null):-1)
	  at java.lang.reflect.Method.invokeNative(Native Method)
	  at java.lang.reflect.Method.invoke(Method.java:511)
	  at com.tencent.connect.a.a.c(ProGuard:90)
	  at com.tencent.connect.auth.QQAuth.<init>(ProGuard:43)
	  at com.tencent.connect.auth.QQAuth.createInstance(ProGuard:78)
	  at com.tencent.tauth.Tencent.<init>(ProGuard:49)
	  at com.tencent.tauth.Tencent.createInstance(ProGuard:57)
	  at com.boyaa.godsdk.core.QQShareSDK.initSDK(QQShareSDK.java:149)
	  at java.lang.reflect.Method.invokeNative(Native Method)
	  at java.lang.reflect.Method.invoke(Method.java:511)
	  at com.boyaa.godsdk.core.utils.ReflectUtils.invokeMethod(ReflectUtils.java:-1)
	  at com.boyaa.godsdk.core.GodSDK.invokeInitPlugin(GodSDK.java:-1)
	  at com.boyaa.godsdk.core.GodSDK.access$8(GodSDK.java:-1)
	  at com.boyaa.godsdk.core.GodSDK$5.handleMessage(GodSDK.java:-1)
	  at android.os.Handler.dispatchMessage(Handler.java:99)
	  at android.os.Looper.loop(Looper.java:137)
	  at android.app.ActivityThread.main(ActivityThread.java:4881)
	  at java.lang.reflect.Method.invokeNative(Native Method)
	  at java.lang.reflect.Method.invoke(Method.java:511)
	  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:804)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:571)
	  at dalvik.system.NativeStart.main(Native Method)

	死锁导致 anr，可以最终朔源，看看是有谁持有锁，发现是tid=14的线程持有锁，那么就要分析tid=14的线程函数栈，发现时沃商店在执行shell脚本的时候 导致死锁了。

如何避免和解决ANR

1>：UI线程尽量只做跟UI相关的工作

2>：耗时的工作（比如数据库操作，I/O，连接网络或者别的有可能阻碍UI线程的操作）把它放入单独的线程处理

3>：尽量用Handler来处理UIthread和别的thread之间的交互

4>：是在绕不开主线程，可以尝试通过Handler延迟加载

5>.广播中如果有耗时操作，建议放在service中去执行，或者通过handlerThread分发执行。

6>.分析着重点：

* cpu占用率方面
	* 可以通过分析各进程的CPU时间占用率，来判断是否为某些进程长期占用CPU导致该进程无法获取到足够的CPU处理时间，而导致ANR
	* 重点关注下CPU的负载，即logcat中Load1(Load: 0.42 / 0.27 / 0.25，即cpu平均负载)，各个进程总的CPU时间占用率，用户CPU时间占用率，核心态CPU时间占用率，以及iowait CPU时间占用率。
* 内存方面
	* 主要看当前应用native和dalvik层内存使用情况
	* 结合系统给每个应用分配的最大内存来分析


附：cpu 负载

CPU负载/平均负载

>CPU负载是指某一时刻系统中运行队列长度之和加上当前正在CPU上运行的进程数，而CPU平均负载可以理解为一段时间内正在使用和等待使用CPU的活动进程的平均数量。在Linux中“活动进程”是指当前状态为运行或不可中断阻塞的进程。通常所说的负载其实就是指平均负载。
用一个从网上看到的很生动的例子来说明（不考虑CPU时间片的限制），把设备中的一个单核CPU比作一个电话亭，把进程比作正在使用和等待使用电话的人，假如有一个人正在打电话，有三个人在排队等待，此刻电话亭的负载就是4。使用中会不断的有人打完电话离开，也会不断的有其他人排队等待，为了得到一个有参考价值的负载值，可以规定每隔5秒记录一下电话亭的负载，并将某一时刻之前的一分钟、五分钟、十五分钟的的负载情况分别求平均值，最终就得到了三个时段的平均负载。
实际上我们通常关心的就是在某一时刻的前一分钟、五分钟、十五分钟的CPU平均负载，例如以上日志中这三个值分别是3.85、3.41、3.16，说明前一分钟内正在使用和等待使用CPU的活动进程平均有3.85个，依此类推。在大型服务器端应用中主要关注的是第五分钟和第十五分钟的两个值，但是Android主要应用在便携手持设备中，有特殊的软硬件环境和应用场景，短时间内的系统的较高负载就有可能造成ANR，所以笔者认为一分钟内的平均负载相对来说更具有参考价值。
CPU的负载和使用率没有必然关系，有可能只有一个进程在使用CPU，但执行的是复杂的操作；也有可能等待和正在使用CPU的进程很多，但每个进程执行的都是简单操作。
实际处理问题时偶尔会遇到由于平均负载高引起的ANR，典型的特征就是系统中应用进程数量多，CPU总使用率较高，但是每个进程的CPU使用率不高，当前应用进程主线程没有异常阻塞，一分钟内的CPU平均负载较高。


5.本次我爱 主要做的工作：

1）数据测量，以及命令使用

* 通过应用性能工具（traceView/systemTrace(android高版本支持)）等分析各个方法占用时间，为分析打好依据；
* 通过 adb shell am start -W -n package/activity来测试记录app的启动时间；
* 通过查看/proc/stat 、 /proc/pid/stat来得到cpu占用率相关信息
* 通过查看/proc/meminfo、dumpsys meminfo、dumpsys meminfo package|pid来确认单个应用的内存使用情况（此步骤包括采用Heap分析工具分析）

2）优化版本

* 第一版本：godsdk全部异步Looper执行
* 第二版本：分两步，1.解析配置文件异步；2.各个sdk初始化整体在UI中执行
* 第三版本：分三步，1.解析配置文件异步；2.sdk初始化在UI中执行；3.一次将sdk的初始化post到主线程中；
* 第四版本：分三步，1.解析配置文件异步；2.各个sdk初始化整体在UI中执行；3.优化具体sdk中的封装；

3）修改依据

* 通过分析traceView 文件按照初始化占用时间由长到短列出表格，然后一次采用第三方demo测试确认是否可以异步执行；
* 分开依次post到主线程 和 同时一起放到主线程，关于上一步数据发现分开和一起其实差别不大，因为导致时间占用较长主要就是那几个sdk导致的，而大多数sdk的初始化占用时间很少，另外，分开post反而会占用更多的资源，导致ANR或者初始化时间更长。
* 延迟加载
* 传入数据给客户端，这一步做了两部小改动
	* 对于那些只有在初始化后才能获取pmode的sdk，做了编译前pmode动态写入操作，即在打包过程中，就把用户动态写入的pmode或者数据写入smali文件中，这样在初始化的时候就避免只有初始化成功后才能获取的情况；
	* 获取pmode不一定要等到初始化成功去获取，只要等到各个sdk对象new成功后就可以获取（提前了，较之前对于异步获取godsdk提供的数据有时候获取不到的情况有所改善）

4）整体来说，修改ANR，关键之处就在于各个sdk里的处理了