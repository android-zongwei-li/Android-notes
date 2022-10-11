

## 一、前言

本文的目标是了解 Android 从点击应用图标到界面显示的过程中都做了什么。

主要会对以下问题进行分析：

- ActivityThread 是什么，它是一个线程吗，如何被启动的？
- ActivityClientRecord 与 ActivityRecord 是什么？
- Context 是什么，ContextImpl，ContextWapper 是什么？
- Instrumentation 是什么？
- Application 是什么，什么时候创建的，每个应用程序有几个 Application？
- 点击 Launcher 启动 Activity 和应用内部启动 Activity 的区别？
- Activity 启动过程，onCreate()，onResume() 回调时机及具体作用？

## 二、Launcher

关于 Android 是如何从开机到 Launcher 启动的过程在笔记（Android 系统启动流程.md）中写了。整体过程如下：

![这里写图片描述](https://img-blog.csdn.net/20180212172032747?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

我们知道 Android 系统启动后已经启动了 Zygote，ServiceManager，SystemServer 等系统进程；ServiceManager 进程中完成了 Binder 初始化；SystemServer 进程中 ActivityManagerService，WindowManagerService，PackageManagerService 等系统服务在 ServiceManager 中已经注册；最后启动了 Launcher 桌面应用。

其实 Launcher 本身就是一个应用程序，运行在自己的进程中，我们看到的桌面就是 Launcher 中的一个 Activity。

应用安装的时候，通过 PackageManagerService 解析 apk 的 AndroidManifest.xml 文件，提取出这个 apk 的信息写入到 packages.xml 文件中，这些信息包括：权限、应用包名、icon、apk 的安装位置、版本、userID 等等。packages.xml 文件位于系统目录下/data/system/packages.xml。（疑问：**怎么查看packages.xml？是不是需要root权限？**）

同时桌面 Launcher 会为安装过的应用生成不同的应用入口，对应桌面上的应用图标，下面分析点击应用图标的到应用启动的过程。

## 三、点击 Launcher 中的应用图标

![这里写图片描述](https://img-blog.csdn.net/20180212172050357?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

点击 Launcher 中应用图标将会执行以下方法：

```java
Launcher.startActivitySafely()
Launcher.startActivity()
//以上两个方法主要是检查将要打开的 Activity 是否存在

Activity.startActivity()
//这段代码大家已经很熟悉，经常打开 Activity 用的就是这个方法

Activity.startActivityForResult()
//默认 requestCode = -1，也可通过调用 startActivityForResult() 传入 requestCode。 
//然后通过 MainThread 获取到 ApplicationThread 传入下面方法。

Instrumentation.execStartActivity()
//通过 ActivityManagerNative.getDefault() 获取到 ActivityManagerService 的代理为进程通讯作准备。

ActivityManagerNative.getDefault().startActivity()
ActivityManagerProxy.startActivity()
//调用代理对象的 startActivity() 方法，发送 START_ACTIVITY_TRANSACTION 命令。
```

在 system_server 进程中的服务端 ActivityManagerService 收到 START_ACTIVITY_TRANSACTION 命令后进行处理，调用 startActivity() 方法。

```java
ActivityManagerService.startActivity() -> startActivityAsUser(intent, requestCode, userId)
//通过 UserHandle.getCallingUserId() 获取到 userId 并调用 startActivityAsUser() 方法。

ActivityStackSupervisor.startActivityMayWait() -> resolveActivity()
//通过 intent 创建新的 intent 对象，即使之前 intent 被修改也不受影响。 然后调用 resolveActivity()。
//然后通过层层调用获取到 ApplicationPackageManager 对象。

PackageManagerService.resolveIntent() -> queryIntentActivities()
//获取 intent 所指向的 Activity 信息，并保存到 Intent 对象。

PackageManagerService.chooseBestActivity()
//当存在多个满足条件的 Activity 则会弹框让用户来选择。

ActivityStackSupervisor.startActivityLocked()
//获取到调用者的进程信息。 通过 Intent.FLAG_ACTIVITY_FORWARD_RESULT 判断是否需要进行 startActivityForResult 处理。 
//检查调用者是否有权限来调用指定的 Activity。 
//创建 ActivityRecord 对象，并检查是否运行 App 切换。

ActivityStackSupervisor.startActivityUncheckedLocked() -> startActivityLocked()
//进行对 launchMode 的处理[可参考 Activity 启动模式]，创建 Task 等操作。
//启动 Activity 所在进程，已存在则直接 onResume()，不存在则创建 Activity 并处理是否触发 onNewIntent()。

ActivityStack.resumeTopActivityInnerLocked()
//找到 resume 状态的 Activity，执行 startPausingLocked() 暂停该 Activity，同时暂停所有处于后台栈的 Activity，找不到 resume 状态的 Activity 则回桌面。
//如果需要启动的 Activity 进程已存在，直接设置 Activity 状态为 resumed。 调用下面方法。

ActivityStackSupervisor.startSpecificActivityLocked()
//进程存在调用 realStartActivityLocked() 启动 Activity，进程不存在则调用下面方法。
```

### fork 新进程

从 Launcher 点击图标，如果应用没有启动过，则会 fork 一个新进程。创建新进程的时候，ActivityManagerService 会保存一个 ProcessRecord 信息，Activity 应用程序中的AndroidManifest.xml 配置文件中，我们没有指定 Application 标签的 process 属性，系统就会默认使用 package 的名称。每一个应用程序都有自己的 uid，因此，这里 uid + process 的组合就可以为每一个应用程序创建一个 ProcessRecord。每次在新建新进程前的时候会先判断这个 ProcessRecord 是否已存在，如果已经存在就不会新建进程了，这就属于应用内打开 Activity 的过程了。

```java
ActivityManagerService.startProcessLocked()
//进程不存在请求 Zygote 创建新进程。 创建成功后切换到新进程。
```

进程创建成功切换至 App 进程，进入 app 进程后将 ActivityThread 类加载到新进程，并调用 ActivityThread.main() 方法

```java
ActivityThread.main()
//创建主线程的 Looper 对象，创建 ActivityThread 对象，ActivityThread.attach() 建立 Binder 通道，开启 Looper.loop() 消息循环。

ActivityThread.attach()
//开启虚拟机各项功能，创建 ActivityManagerProxy 对象，调用基于 IActivityManager 接口的 Binder 通道 ActivityManagerProxy.attachApplication()。

ActivityManagerProxy.attachApplication()
//发送 ATTACH_APPLICATION_TRANSACTION 命令
```

此时只创建了应用程序的 ActivityThread 和 ApplicationThread，和开启了 Handler 消息循环机制，其他的都还未创建， ActivityThread.attach(false) 又会最终到 ActivityMangerService 的 attachApplication，这个工程其实是将本地的 ApplicationThread 传递到 ActivityMangerService。然后 ActivityMangerService 就可以通过 ApplicationThread 的代理 ApplicationThreadProxy 来调用应用程序 ApplicationThread.bindApplication，通知应用程序的 ApplicationThread 已和 ActivityMangerService 绑定，可以不借助其他进程帮助直接通信了。此时 Launcher 的任务也算是完成了。

在 system_server 进程中的服务端 ActivityManagerService 收到 ATTACH_APPLICATION_TRANSACTION 命令后进行处理，调用 attachApplication()。

```java
ActivityMangerService.attachApplication() -> attachApplicationLocked()
//首先会获取到进程信息 ProcessRecord。 绑定死亡通知，移除进程启动超时消息。 获取到应用 ApplicationInfo 并绑定应用 IApplicationThread.bindApplication(appInfo)。
//然后检查 App 所需组件。
```

- Activity: 检查最顶层可见的 Activity 是否等待在该进程中运行，调用 ActivityStackSupervisor.attachApplicationLocked()。
- Service：寻找所有需要在该进程中运行的服务，调用 ActiveServices.attachApplicationLocked()。
- Broadcast：检查是否在这个进程中有下一个广播接收者，调用 sendPendingBroadcastsLocked()。

此处讨论 Activity 的启动过程，只讨论 ActivityStackSupervisor.attachApplicationLocked() 方法。

```java
ActivityStackSupervisor.attachApplicationLocked() -> realStartActivityLocked()
//将该进程设置为前台进程 PROCESS_STATE_TOP，调用 ApplicationThreadProxy.scheduleLaunchActivity()。

ApplicationThreadProxy.scheduleLaunchActivity()
//发送 SCHEDULE_LAUNCH_ACTIVITY_TRANSACTION 命令
```









参考：

https://www.jianshu.com/p/a72c5ccbd150

https://www.jianshu.com/p/6037f6fda285

https://blog.csdn.net/freekiteyu/article/details/79318031



https://www.csdn.net/article/2014-08-15/2821226#0-tsina-1-80696-397232819ff9a47a7b7e80a40613cfe1

