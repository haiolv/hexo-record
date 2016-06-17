---
title: Android的事件处理机制
date: 2016-06-14 18:59:48
tags: [Android,Activity事件传递,dispatchEvent]
categories: [编程,Android] # 配置categories
---


## Android的事件处理机制

Android的事件产生是从我们触摸屏幕开始，在经过Input子系统，最后达到我们的应用程序(或者经过WindowManagerService到达应用程序)。

而其中Input子系统在Java层对应着InputManagerService，其主要在native层，由InputReader读取EventHub的元数据，将这些数据加工成InputEvent，最后发到InputDispatcher，而InputDispatcher则负责将事件发到应用程序，Input子系统流程可以参见这篇文章 [Android Framework——之Input子系统](http://www.cnblogs.com/haiming/p/3318614.html "Android Framework——之Input子系统")。 和 [Android 事件捕捉和处理流程分析](http://blog.csdn.net/yclzh0522/article/details/6920522 "http://blog.csdn.net/yclzh0522/article/details/6920522")

对于应用层的时间流程，主要是下面的流程图所示：
![key-transfer](http://i.imgur.com/yugKXH4.png)

其中最后一步就是我们经常说的View事件派发流程。而在流程图中，我们也看到，调了两次DecorView的方法，第一次是调用DecorView的dispatchTouchEvent，它的源码是:

	@Override
	public boolean dispatchTouchEvent(MotionEvent ev) {
	    final Callback cb = getCallback();
	    return cb != null && !isDestroyed() && mFeatureId < 0 ? cb.dispatchTouchEvent(ev)
	            : super.dispatchTouchEvent(ev);
	}

调用了Callback的dispatchTouchEvent，那这个Callback是什么类型呢？

Callback就是Window.Callback，Activity实现了这个接口。在Activity的attach函数中，会调用window的setCallback，将Activity设置给Window。所以这里getCallback返回的就是Activity，最终会调用Activity的dispatchTouchEvent。

	public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }

在ACTION_DOWN事件处理的时候会调用onUserInteraction方法，然后调用Window（实际上是PhoneWindow)的superDispatchTouchEvent，如果Window的superDispatchTouchEvent消耗了事件，则直接返回，不会调用Activity的onTouchEvent方法。

	public boolean superDispatchTouchEvent(MotionEvent event) {
	    return mDecor.superDispatchTouchEvent(event);
	}

而DecorView的superDispatchTouchEvent为:

	public boolean superDispatchTouchEvent(MotionEvent event) {
	    return super.dispatchTouchEvent(event);
	}

最终还是调用DecorView的父类的dispatchTouchEvent，DecorView的父类是FrameLayout，它没实现该方法，最终会调用ViewGroup的dispatchTouchEvent方法。从这里开始就进入了view树的事件派发流程了。

此外，关于Android view树的事件派发流程，众说纷纭，给出一篇参考文章

Reference：[Android事件传递机制](http://www.infoq.com/cn/articles/android-event-delivery-mechanism/ "http://www.infoq.com/cn/articles/android-event-delivery-mechanism/")

然后，站在前人的肩膀上，接下俩文章主要做些验证。


### Question:

* Activity中dispatchTouchEvent在什么时候调用？

* MotionEvent事件向下分发，是通过什么途径？通过dispatchKeyEvent的返回值？


简单列举下工程

* Activity:

		// haio.com.event.MainActivity
	
		@Override
	    public boolean onTouchEvent(MotionEvent event) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_DOWN:
	                System.out.println("Activity---onTouchEvent---DOWN");
	                break;
	            case MotionEvent.ACTION_MOVE:
	                System.out.println("Activity---onTouchEvent---MOVE");
	                break;
	            case MotionEvent.ACTION_UP:
	                System.out.println("Activity---onTouchEvent---UP");
	                break;
	            default:
	                break;
	        }
	        boolean isTouch = super.onTouchEvent(event);
	        System.out.println("Activity---onTouchEvent---isTouch =" + isTouch);
	        return isTouch;
	    }
	
		@Override
	    public boolean dispatchTouchEvent(MotionEvent event) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_DOWN:
	                System.out.println("Activity---dispatchTouchEvent---DOWN");
	                break;
	            case MotionEvent.ACTION_MOVE:
	                System.out.println("Activity---dispatchTouchEvent---MOVE");
	                break;
	            case MotionEvent.ACTION_UP:
	                System.out.println("Activity---dispatchTouchEvent---UP");
	                break;
	            default:
	                break;
	        }
		//        boolean isDispatch = super.dispatchTouchEvent(event);
	        boolean isDispatch = true;
	        System.out.println("Activity---dispatchTouchEvent---isDispatch =" + isDispatch);
	        return isDispatch;
	    }
	
	    @Override
	    public boolean onKeyDown(int keyCode, KeyEvent event) {
	        System.out.println("Activity---onKeyDown---DOWN");
	        boolean isDown = super.onKeyDown(keyCode, event);
	        System.out.println("Activity---onKeyDown---isDown =" + isDown);
	        return isDown;
	    }


* HButton

	View(如 Button等) 涉及事件分发共涉及两个个事件

	* dispatchTouchEvent
	* onTouchEvent

			//haio.com.event.widget.HButton
			@Override
		    public boolean onTouchEvent(MotionEvent event) {
		        switch (event.getAction()) {
		            case MotionEvent.ACTION_DOWN:
		                System.out.println("HButton---onTouchEvent---DOWN");
		                break;
		            case MotionEvent.ACTION_MOVE:
		                System.out.println("HButton---onTouchEvent---MOVE");
		                break;
		            case MotionEvent.ACTION_UP:
		                System.out.println("HButton---onTouchEvent---UP");
		                break;
		            default:
		                break;
		        }
		        boolean isTouch = super.onTouchEvent(event);
		        System.out.println("HButton---onTouchEvent---isTouch =" + isTouch);
		        return isTouch;
		    }
		
		    @Override
		    public boolean dispatchTouchEvent(MotionEvent event) {
		        switch (event.getAction()) {
		            case MotionEvent.ACTION_DOWN:
		                System.out.println("HButton---dispatchTouchEvent---DOWN");
		                break;
		            case MotionEvent.ACTION_MOVE:
		                System.out.println("HButton---dispatchTouchEvent---MOVE");
		                break;
		            case MotionEvent.ACTION_UP:
		                System.out.println("HButton---dispatchTouchEvent---UP");
		                break;
		            default:
		                break;
		        }
		        boolean isDispatch = super.dispatchTouchEvent(event);
			//        boolean isDispatch = true;
		        System.out.println("HButton---dispatchTouchEvent---isDispatch =" + isDispatch);
		        return isDispatch;
		    }

* ViewGroup

	ViewGroup 涉及事件分发共涉及三个事件

	* dispatchTouchEvent
	* onInterceptTouchEvent
	* onTouchEvent

			//haio.com.event.widget.HLinearLayout
	
		    @Override
		    public boolean onTouchEvent(MotionEvent event) {
		        switch (event.getAction()) {
		            case MotionEvent.ACTION_DOWN:
		                System.out.println("HLinearLayout---onTouchEvent---DOWN");
		                break;
		            case MotionEvent.ACTION_MOVE:
		                System.out.println("HLinearLayout---onTouchEvent---MOVE");
		                break;
		            case MotionEvent.ACTION_UP:
		                System.out.println("HLinearLayout---onTouchEvent---UP");
		                break;
		            default:
		                break;
		        }
		        boolean isTouch = super.onTouchEvent(event);
		        System.out.println("HLinearLayout---onTouchEvent---isTouch =" + isTouch);
		        return isTouch;
		    }
		
		    @Override
		    public boolean onInterceptTouchEvent(MotionEvent event) {
		        switch (event.getAction()) {
		            case MotionEvent.ACTION_DOWN:
		                System.out.println("HLinearLayout---onInterceptTouchEvent---DOWN");
		                break;
		            case MotionEvent.ACTION_MOVE:
		                System.out.println("HLinearLayout---onInterceptTouchEvent---MOVE");
		                break;
		            case MotionEvent.ACTION_UP:
		                System.out.println("HLinearLayout---onInterceptTouchEvent---UP");
		                break;
		            default:
		                break;
		        }
		
		        boolean isIntercept = super.onInterceptTouchEvent(event);
		        System.out.println("HLinearLayout---onInterceptTouchEvent---isIntercept =" + isIntercept);
		        return isIntercept;
		    }
		
		    @Override
		    public boolean dispatchTouchEvent(MotionEvent event) {
		        switch (event.getAction()) {
		            case MotionEvent.ACTION_DOWN:
		                System.out.println("HLinearLayout---dispatchTouchEvent---DOWN");
		                break;
		            case MotionEvent.ACTION_MOVE:
		                System.out.println("HLinearLayout---dispatchTouchEvent---MOVE");
		                break;
		            case MotionEvent.ACTION_UP:
		                System.out.println("HLinearLayout---dispatchTouchEvent---UP");
		                break;
		            default:
		                break;
		        }
		        boolean isDispatch = super.dispatchTouchEvent(event);
			//        boolean isDispatch = true;
		        System.out.println("HLinearLayout---dispatchTouchEvent---isDispatch =" + isDispatch);
		        return isDispatch;
		    }


上述代码，细心的小伙伴们会发现，将MainActivity 中dispatchTouchEvent的返回值改为true,并屏蔽掉

	super.dispatchTouchEvent(event)

的调用，然后分别点击 Button 和 TextView的日志为：

	1.点击按钮：
	06-14 17:41:58.639 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---DOWN
	06-14 17:41:58.639 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:41:58.689 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---MOVE
	06-14 17:41:58.689 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:41:58.694 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---UP
	06-14 17:41:58.694 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true

	2.点击文本：
	06-14 17:41:58.694 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:51:09.684 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---DOWN
	06-14 17:51:09.684 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:51:09.809 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---UP
	06-14 17:51:09.809 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true


然后，我们在将MainActivity 中dispatchTouchEvent的返回值改为false，查看日志：

	06-14 17:54:23.509 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---DOWN
	06-14 17:54:23.509 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false
	06-14 17:54:23.524 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---MOVE
	06-14 17:54:23.524 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false
	06-14 17:54:23.579 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---UP
	06-14 17:54:23.579 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false


从上面，我们得知，dispatchTouchEvent返回true 表明当前view/viewGroup负责处理该事件 ，false表明不负责处理该事件， 从日志打印结果中看出return true是不会有任何区别的。


那么，问题来了，Activity是通过调用上面来分发事件的呢？

我们将 MainActivity 中dispatchTouchEvent 改成如下，放开 super.dispatchTouchEvent(event) 的调用，并且返回true，表示 负责处理该事件。

    public boolean dispatchTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                System.out.println("Activity---dispatchTouchEvent---DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                System.out.println("Activity---dispatchTouchEvent---MOVE");
                break;
            case MotionEvent.ACTION_UP:
                System.out.println("Activity---dispatchTouchEvent---UP");
                break;
            default:
                break;
        }


        boolean isDispatch = super.dispatchTouchEvent(event);
	//        boolean isDispatch = false;
        isDispatch = true;


        System.out.println("Activity---dispatchTouchEvent---isDispatch =" + isDispatch);
        return isDispatch;
    }


日志打印为：

	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---isIntercept =false
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HButton---onTouch---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---DOWN
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---isTouch =true
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---MOVE
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---MOVE
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---MOVE
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---isIntercept =false
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---MOVE
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HButton---onTouch---MOVE
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---MOVE
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---isTouch =true
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.234 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---UP
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---UP
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---UP
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---isIntercept =false
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---UP
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HButton---onTouch---UP
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---UP
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---isTouch =true
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.239 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.249 17504-17504/haio.com.event I/System.out: HButton---onClick---DOWN


事件分发了，这说明：

dispatchTouchEvent返回true 表明当前view/viewGroup负责处理该事件 false表明不负责处理该事件 在MainActivity中 如果你把return值改为false 打印结果和return true是不会有任何区别的 因为你根本就没有向下进行分发 MainActivity中分发是调用super.dispatchTouchEvent的。

注意，一个事件分发的完整过程：

	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---isIntercept =false
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HButton---onTouch---DOWN
	06-14 17:58:12.159 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---DOWN
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---isTouch =true
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---isDispatch =true
	06-14 17:58:12.169 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =true
	
后面三行打印，说明HButton、HLinearLayout、Activity都负责此事件，因此后续事件还会分发到HButton、HLinearLayout、Activity上。



#### 如果 子View不消耗事件，则回溯给Activity进行执行
	
当你把HButton中的dispatchTouchEvent返回值改变为false的时候 说明没有调用View.onTouchEvent来消费事件,日志如下：

	06-14 18:09:10.934 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---DOWN
	06-14 18:09:10.959 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---DOWN
	06-14 18:09:10.979 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---DOWN
	06-14 18:09:10.979 17504-17504/haio.com.event I/System.out: HLinearLayout---onInterceptTouchEvent---isIntercept =false
	06-14 18:09:10.979 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---DOWN
	06-14 18:09:10.979 17504-17504/haio.com.event I/System.out: HButton---onTouch---DOWN
	06-14 18:09:10.979 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---DOWN
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---isTouch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---isDispatch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HLinearLayout---onTouchEvent---DOWN
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HLinearLayout---onTouchEvent---isTouch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---isDispatch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---DOWN
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---isTouch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false
	06-14 18:09:11.014 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---MOVE
	06-14 18:09:11.044 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---MOVE
	06-14 18:09:11.044 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---isTouch =false
	06-14 18:09:11.044 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false
	06-14 18:09:11.044 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---UP
	06-14 18:09:11.049 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---UP
	06-14 18:09:11.049 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---isTouch =false
	06-14 18:09:11.049 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false


如上，当你把HButton中的dispatchTouchEvent返回值改变为false的时候 说明没有调用View.onTouchEvent来消费事件, 所以返回之后最终MainActivity的onTouchEvent还是会对Event进行处理, 假如某层dispatchTouchEvent返回值为false,那么后续的move up等等action就不会再被分发到当前层了.


注意，针对以上，下面日志反应一个事件的分发过程
	
	06-14 18:09:10.979 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---DOWN
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HButton---onTouchEvent---isTouch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HButton---dispatchTouchEvent---isDispatch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HLinearLayout---onTouchEvent---DOWN
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HLinearLayout---onTouchEvent---isTouch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: HLinearLayout---dispatchTouchEvent---isDispatch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---DOWN
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---isTouch =false
	06-14 18:09:10.994 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false

打印说明HButton、HLinearLayout都不负责此事件（返回false），因此后续事件就不会分发到HButton、HLinearLayout上。

	06-14 18:09:11.044 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---UP
	06-14 18:09:11.049 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---UP
	06-14 18:09:11.049 17504-17504/haio.com.event I/System.out: Activity---onTouchEvent---isTouch =false
	06-14 18:09:11.049 17504-17504/haio.com.event I/System.out: Activity---dispatchTouchEvent---isDispatch =false