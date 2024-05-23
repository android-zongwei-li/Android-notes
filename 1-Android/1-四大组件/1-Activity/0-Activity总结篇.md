> version：2021/4/13 - 20240128
>
> review：2021/4/13 - 20240128
>



# 前言

- `Activity`属于 `Android`的四大组件之一
- Carons将献上一份 `Activity`的学习攻略，包括其生命周期、启动模式、启动方式等等，希望你们会喜欢。

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-e1698c9242f037e7.png?imageMogr2/auto-orient/strip|imageView2/2/w/846/format/webp)

示意图

------

# 1. 定义

即 活动，属于 **展示型组件**

> 属于`Android`四大组件之一：`Activity`、`Service`、`BroadcastReceiver`、`ContentProvider`

------

# 2. 作用

显示界面 & 与用户进行交互

> 1. 一个`Activity`通常是一个界面，是四大组件唯一能被用户感知的
> 2. 每个活动被实现为一个独立的类， & 从活动基类继承过来
> 3. `Activity`之间通过`Intent`进行通信

------

# 3. 生命周期

- 具体如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-8f75c4ffd13eab1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

> 更加详细请看文章：[Android基础：3分钟详解Activity生命周期](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F82620404)

------

# 4. 启动模式

- `Activity`的启动模式有4种，具体如下

![img](https:////upload-images.jianshu.io/upload_images/944365-b6244df4e6c21315.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- 4种启动模式的区别

![img](https:////upload-images.jianshu.io/upload_images/944365-f8ff1efde0a29112.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

> 更加详细请看文章：[Android基础：最易懂的Activity启动模式详解](https://www.jianshu.com/p/399e83d02e33)

------

# 5. 启动方式

- 启动`Activity`的方式主要是：显式`Intent` & 隐式`Intent`
- 具体介绍如下：

### 5.1 显式Intent（3种）



```java
// 1. 使用构造函数 传入 Class对象
 Intent intent = new Intent(this, SecondActivity.class); 
 startActivity(intent);

// 2. 使用 setClassName（）传入 包名+类名 / 包Context+类名
 Intent intent = new Intent(); 
 // 方式1：包名+类名
 // 参数1 = 包名称
 // 参数2 = 要启动的类的全限定名称 
 intent.setClassName("com.hc.hctest", "com.hc.hctest.SecondActivity"); 

 // 方式2：包Context+类名
 // 参数1 = 包Context，可直接传入Activity
 // 参数2 = 要启动的类的全限定名称 
 intent.setClassName(this, "com.hc.hctest.SecondActivity"); 

 startActivity(intent);

// 3. 通过ComponentName（）传入 包名 & 类全名
 Intent intent = new Intent(); 
 // 参数1 = 包名称
 // 参数2 = 要启动的类的全限定名称 
 ComponentName cn = new ComponentName("com.hc.hctest", "com.hc.hctest.SecondActivity"); 
 intent.setComponent(cn); 
 startActivity(intent);
```

### 5.2 隐式Intent



```cpp
// 通过Category、Action设置
Intent intent = new Intent(); 
intent.addCategory(Intent.CATEGORY_DEFAULT); 
intent.addCategory("com.hc.second"); 
intent.setAction("com.hc.action"); 
startActivity(intent);
```

### 5.3 匹配规则

![img](https:////upload-images.jianshu.io/upload_images/944365-b7156549d2e3d095.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

> 更加详细请看文章：[Android：关于 Intent组件的那些小事（介绍、使用方法等）](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F82767978)

------

# 6. 启动过程

`Activity`的启动过程具体如下：

### 6.1 示意图

![img](https:////upload-images.jianshu.io/upload_images/944365-8746d7ac74220415.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

### 6.2 具体描述

当请求启动`Activity`时：

1. `Launcher`进程通过`Binder`驱动向`ActivityManagerService`类发起`startActivity`请求；
2. `ActivityManagerService`类接收到请求后，向`ActivityStack`类发送启动`Activity`的请求；
3. `ActivityStack`类记录需启动的`Activity`的信息 & 调整`Activity`栈 将其置于栈顶、通过 `Binder` 驱动 将 `Activity` 的启动信息传递到`ApplicationThread`线程中（即`Binder`线程）
4. `ApplicationThread`线程通过`Handler`将`Activity`的启动信息发送到主线程`ActivityThread`
5. 主线程`ActivityThread`类接收到该信息 & 请求后，通过`ClassLoader`机制加载相应的`Activity`类，最终调用`Activity`的`onCreate（）`，最后 启动完毕

------

# 7. 卡顿原因

`Activity`的卡顿原因主要归结如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-d052fd1a02f5b4a2.png?imageMogr2/auto-orient/strip|imageView2/2/w/850/format/webp)

示意图

关于内存泄漏 & 性能优化，请看系列文章：
 [Android性能优化：这是一份全面&详细的内存优化指南](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F79549417)
 [Android性能优化：手把手带你全面了解 内存泄露 & 解决方案](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F79407707)
 [Android性能优化：那些关于Bitmap图片资源优化的小事](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F79549382)
 [Android性能优化：手把手带你全面了解 绘制优化](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F79674623)
 [Android性能优化：布局优化 详细解析（含、、讲解 ）](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F79620486)

------

# 8. 加速启动方式

加速启动`Activity`的方式归结如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-19fa54eef58f1744.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 9. 缓存方式（状态保存）

- 问题描述

![img](https:////upload-images.jianshu.io/upload_images/944365-8fe90db664407e3b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- 具体说明

![img](https:////upload-images.jianshu.io/upload_images/944365-305b102f39ac27ce.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 10. Activity 与Fragment的交互方式

- 主要有：接口、Bundle、广播
- 具体请看文章：[Android：手把手教你 实现Activity 与 Fragment 相互通信（含Demo）](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F75453770)

至此，关于`Android`四大组件之一的`Activity`讲解完毕。

------

# 11. 总结

本文全面讲解了 `Activity`，现在大家对 `Activity`应该十分了解了。









## 一、前言

Activity 作为 Android 的核心组件，在工作和面试中都尤为重要。我们不仅要会使用，还要对它的相关特性（生命周期、启动模式等）了如指掌，也要对其源码（启动流程、和Window与View的关系等）有深入理解。本文会对掌握 Activity 所需了解的知识点做一个整理，然后通过其他几篇笔记来对其中的各个知识点做详细总结。

## 二、Activity

### 2.1 **Activity 是什么？** 

Activity 实际上只是一个与用户交互的接口而已。

对设备用户而言，Activity是可以和用户进行交互的界面，它接受点击、滑动等事件，并且将处理结果反馈给用户。

从代码层面，对开发而言，它是继承自Context的一个类，因此通过它可以和系统底层以及资源进行交互，然后它的主要的职责就是UI的交互。

activity是独立平等的，用来处理用户操作。几乎所有的activity都是用来和用户交互的，所以activity类会创建了一个窗口，开发者可以通过setContentView(View)的接口把UI放到给窗口上。



### 2.2 Activity 生命周期（重点）

生命周期的内容比较多，也有许多 case 需要总结，所以这部分会单独做一篇笔记（降低耦合）。

笔记：1-Activity 生命周期总结篇.md

### 2.3  Activity 的启动模式（重点）

启动模式的内容也是多且复杂，需要掌握其基本原理、适用场景，我单独做一篇笔记。

笔记：2-Activity 启动模式总结篇.md

### 2.4 Activity 启动流程（重点）

笔记：3-Activity启动流程总结篇.md

### 2.6 Activity 组件之间的通信

笔记：4-Activity与各组件之间的通信.md

### 2.7 scheme 跳转协议

Android 中的 scheme 是一种页面内跳转协议，通过自定义 scheme 协议，可以非常方便的跳转到 app 中的各个页面，通过 scheme 协议，服务器可以定制化告诉 app 跳转到哪个页面，可以通过通知栏消息定制化跳转页面，可以通过 H5 页面跳转到相应页面等等。



### Activity 的管理 —— Task 栈

Android中的activity全都归属于task管理 。task 是多个 activity 的集合，这些 activity 按照启动顺序排队存入一个栈（即“back stack”）。android默认会为每个App维持一个task来存放该app的所有activity，task的默认name为该app的packagename。我们也可以在 AndroidMainfest.xml 中申明 activity 的 taskAffinity 属性来自定义task，但不建议使用，如果其他app也申明相同的task，它就有可能启动到你的activity，带来各种安全问题（比如拿到你的Intent）。



### 2.9 其他





# Q&A

- 1.Activity是什么？
- 2.典型情况下的Activity生命周期？
- 3.异常情况下的Activity的生命周期 & 数据如何保存和恢复？ 

> [1~3题答案](https://blog.csdn.net/clandellen/article/details/79257489)

- 4.从Activity A跳转到Activity B之后，然后再点击back建之后，它们的生命周期调用流程是什么？

> 从Activity A跳转到Activity B  
> Activity A --> onPause()  
> Activity B --> onCreate()  
> Activity B --> onStart()  
> Activity B --> onResume()  
> Activity A --> onStop()    
> 然后在Activity B点击back键  
> Activity B --> onPause()  
> Activity A --> onRestart()  
> Activity A --> onStart()  
> Activity A --> onResume()  
> Activity B --> onStop()  
> Activity B --> onDestory()  

- 5.如何统计Activity的工作时间？

> 这道题目也在考察Activity的生命周期，Activity开始工作的起点是onResume()而工作的停止点为onPause(),因此当每次Activity调用onResume()的时候记录一个时间a，每次调用onPause()的时候再记录一个时间b,那么由b-a可得当次Activity工作的时间。

- 6.给我说说Activity的启动模式 & 使用场景。

> 这道题目答案在上面1~3题答案的链接里。

- 7.如何在任意位置关掉应用所有Activity & 如何在任意位置关掉指定的Activity？

> 这道题目的答案也很简单，封装一个类,成员变量有一个List<Activity>集合，当Activity执行onCreate()方法时将当前的Activity实例加入，当Activity执行onDestory()方法时，移除当前Activity实例即可，如何关闭应用所有的Activity,答案就是遍历这个List<Activity>且逐一调用finish()方法即可，至于如何在任意位置关闭当前的Activity,这里需要考虑给每个启动的Activity一个tag,根据这个tag和集合可达到在任意位置关闭指定Activity的效果。

- 8.Activity的启动流程(从源码角度解析)？

> [点击查看答案](https://www.cnblogs.com/kross/p/4025075.html)

- 9.启动一个其它应用的Activity的生命周期分析。

> 这里不仅仅考察Activtiy的生命周期问题，还考察返回栈的问题。具体答案请查看1~3题答案链接。

- 10.Activity任务栈是什么？在项目中有用到它吗？说给我听听

> [点击查看答案](https://www.cnblogs.com/z964166725/p/8729208.html)

- 11.什么情况下Activity不走onDestory?
- 12.什么情况下Activity会单独执行onPause?

> 11和12题答案也在1-3的答案链接里。

- 13.a->b->c界面，其中b是SingleInstance的，那么c界面点back返回a界面，为什么？

> 这题实际在考察你对Activity启动模式SingleInstance的知识点，首先我们来看看SingleInstance的特性：


>单实例模式：SingleInstance
>  这是一种加强的singleTask模式，它除了具有singleTask模式所有的特性外，还加强了一点，那就是具有此种模式的Activity只能单独位于一个任务栈中，换句话说，比如Activity A是singleInstance模式，当A启动后，系统会为它创建一个新的任务栈，然后A独自在这个新的任务栈中，由于栈内复用的特性，后续的请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了。

>对于SingleInstance,面试时你有说明它的以下几个特点：

>（1）以singleInstance模式启动的Activity具有全局唯一性，即整个系统中只会存在一个这样的实例。
>（2）以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activiyt时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。
>（3）以singleInstance模式启动的Activity具有独占性，即它会独自占用一个任务，被他开启的任何activity都会运行在其他任务中。
>（4）被singleInstance模式的Activity开启的其他activity，能够在新的任务中启动，但不一定开启新的任务，也可能在已有的一个任务中开启。

> 换句话说，其实SingleInstance就是我们刚才分析的SingleTask中，分享Activity为栈底元素的情况。


- 14.如果一个Activity弹出一个Dialog,那么这个Acitvity会回调哪些生命周期函数呢？

> 也许你会不假思索的回答会回调onPause()方法,然而...
> [点击查看答案](https://blog.csdn.net/yz_cfm/article/details/85476263)

- 15.Activity之间如何通信 & Activity和Fragment之间通信 & Activity和Service之间通信？

> 答案在1~3题答案链接中

- 16.说说Activity横竖屏切换的生命周期。

> 答案在1~3题答案链接中

- 17.前台切换到后台，然后在回到前台时Activity的生命周期。

> 答案在1~3题答案链接中

- 18.下拉状态栏时Activity的生命周期？

> [点击查看答案](https://www.jianshu.com/p/781bc86f8042)

- 19.Activity与Fragment的生命周期比较？

> [点击查看答案](https://www.jianshu.com/p/6fb2936d2d3c)

- 20.了解哪些Activity常用的标记位Flags？

常用标记位

>1.Intent.FLAG_ACTIVITY_NEW_TASK，是为Activity指定“singleTask”启动模式
>
>2.Intent.FLAG_ACTIVITY_SINGLE_TOP，是为Activity指定“singleTop”启动模式
>
>3.FLAG_ACTIVITY_CLEAR_TOP，如果跟singleTask启动模式一起出现，如果被启动的Activity已经存在实例，则onNewIntent方法会被回调，如果被启动的Activity采用standard模式启动，那么连同它跟它之上的Activity都要出栈，并且创建新的实例放入栈顶。
>
>4.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS，新的Activity不会在最近启动的Activity的列表中保存。等同于指定属性android:excludeFromRecents="true"

- 21.谈谈隐式启动和显示启动Activity的方式？

> [点击查看答案](https://www.jianshu.com/p/27d0ec1c52b2)

- 22.Activity用Intent传递数据和Bundle传递数据的区别？为什么不用HashMap呢？

> [点击查看答案](https://www.cnblogs.com/On1Key/p/5180022.html)



# 参考

[Carson带你学Android：关于Activity的知识都在这里了](https://www.jianshu.com/p/32938446e4e0)
