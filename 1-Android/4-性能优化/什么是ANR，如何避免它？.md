> version：2021/10/28
>
> review：

可选：[原文地址](https://developer.android.google.cn/guide/fragments/create)

目录

[TOC]



# 一、什么是ANR

在 Android 系统中，如果应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应（ANR：ApplicationNotResponding）对话框。 用户可以选择让程序继续运行，但是，他们在使用你的应用程序时，并不希望每次都要处理这个对话框。因此 ，在程序里对响应性能的设计很重要，这样系统就不会显示 ANR 给用户。 

# 二、ANR的触发

## 1、各组件触发ANR的时间

Activity、BroadCastReceiver、Service触发ANR的时间

Android 系统会监控程序的响应状况，不同的组件发生 ANR 的时间不一样：
Activity：5 秒。应用在 5 秒内未响应用户的输入事件（如按键或者触摸）
BroadCastReceiver ：10 秒。BroadcastReceiver 未在 10 秒内完成相关的处理
Service：20 秒（均为前台）。Service 在20 秒内无法处理完成 

## 2、引起ANR的原因

* 主线程被 IO 操作（从 4.0 之后网络 IO 不允许在主线程中）阻塞；
* 主线程中存在耗时的计算；
* 主线程中错误的操作，比如 Thread.wait 或者 Thread.sleep 等。

## 3、ANR信息查看

如果开发机器上出现问题，我们可以查看/data/anr/traces.txt，最新的 ANR 信息在最开始部分。

# 三、避免ANR的建议

核心思路：快速响应，耗时操作放到子线程中，不要阻塞UI线程。

1、使用 AsyncTask 处理耗时 IO 操作。
2、使用 Thread 或者 HandlerThread 时，要设置线程优先级。未调用 Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，仍然会降低程序响应，因为默认 Thread 的优先级和主线程相同。 
3、使用 Handler 处理工作线程结果，而不是使用 Thread.wait()或者 Thread.sleep() 来阻塞主线程。 
4、Activity 的 onCreate 和 onResume 回调中尽量避免耗时的代码。 
5、BroadcastReceiver 中 onReceive 代码也要尽量减少耗时，建议使用 IntentService 处理。 

小结：
将所有耗时操作，比如访问网络，Socket 通信，查询大量 SQL 语句，复杂逻辑计算等都放在子线程中去，然后通过 handler.sendMessage、runonUIThread、 AsyncTask、RxJava 等方式更新 UI。无论如何都要确保用户界面的流畅度。如果耗时操作需要让用户等待，那么可以在界面上显示度条。



# 相关问题

## 一、概念

<font color='orange'>Q：ANR的原理</font>

**ANR的监测机制**：首先分析Service和输入事件大致工作流程，然后从Service，InputEvent两种不同的ANR监测机制的源码实现开始，分析了Android如何发现各类ANR。在启动服务、输入事件分发时，植入超时检测，用于发现ANR。

**ANR的报告机制**：分析Android如何输出ANR日志。当ANR被发现后，两个很重要的日志输出是：CPU使用情况和进程的函数调用栈，这两类日志是我们解决ANR问题的利器。

**监测ANR的核心原理**是消息调度和超时处理。

只有被ANR监测的场景才会有ANR报告以及ANR提示框。

<font color='orange'>Q：ANR产生的原因是什么？</font>

ANR全称ApplicationNotResponding，出现ANR，系统会弹出一个弹出让用户选择继续等待或者关闭程序。引起ANR问题的根本原因是我们的程序处理的不够及时，不能快速的响应用户的请求。

各个组件系统定义的响应时间不同，Activity 5 秒、BroadCastReceiver 10 秒、Service：20 秒。

<font color='orange'>Q：ANR异常发生条件。</font>

<font color='orange'>Q：什么情况下会发生ANR？</font>

常见的引起ANR的场景有：

在主线程中执行耗时操作，比如IO、使用Binder调用服务、大量运算、主线程的错误调用sleep等。

另一个是在设备低性能也会引起。因此要注意定标准，在设备各种情况下都要充分验证，尤其是低性能设备。

<font color='orange'>Q：ANR怎么产生?怎么捕捉？</font>



<font color='orange'>Q：安卓系统通过什么方式检测ANR</font>



<font color='orange'>Q：什么是ANR,如何避免它？</font>

避免：

写代码时：

1、减少在主线程中的耗时操作。把耗时的操作都放到子线程、或者其他进程中处理。

2、在Activity的生命周期方法中，尽量避免计算等操作，尽量让主线程只负责处理UI相关的事情。

然后在项目开发流程中，

1、要通过review避免

2、通过跑monkey等测试来尽可能在上线前暴露出问题。

## 二、检测

<font color='orange'>Q：如何检测是否发生了ANR</font>

跑Monkey测试，导出/data/anr/traces.txt文件查看。

<font color='orange'>Q：ANR异常如何查找并分析？</font>

可以通过导出/data/anr/traces.txt文件查看。

分析：

首先要定位：

## 三、定位

<font color='orange'>Q：ANR定位和修正</font>

分析Android如何输出ANR日志。当ANR被发现后，两个很重要的日志输出是：**CPU使用情况和进程的函数调用栈**，这两类日志是我们解决ANR问题的利器。

在anr文件中，检索关键字来定位。

## 四、解决

<font color='orange'>Q：如何分析ANR及如何处理</font>



# 总结

1、ANR问题，和内存泄漏、OOM以及其他异常有一点是类似的，就是**能定位到就解决了一大半**。

Tips1：定位问题。

引起ANR问题的根本原因是我们的程序处理的不够及时，不能快速的响应用户的请求。

对于ANR问题，要了解产生ANR的原因，首先**开发阶段**要尽量避免写出可能引起ANR的代码，这个步骤的基本保障有两点：一个是通常靠经验来避免，经验可以通过学习总结、内部培训可以达到；一个是通过代码review做一层保障。

测试阶段，尽量通过压测或者其他测试，尽可能的暴露问题。

线上阶段，要能够检测、上报ANR。

然后就是定位，解决。

**ANR问题解决四步骤：避免、检测、定位、解决。**



## 【精益求精】我还能做（补充）些什么？

1、



# 参考

1、[深入解析](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649493643&idx=1&sn=34b51d1f61bd2ecaa8fd0a2d39c4d1d1&chksm=8eec9b74b99b126246acc4547597dfe55c836b8f689b2d1a65bdf1ee2054ced2fc070bfa2678&mpshare=1&scene=24&srcid=0116vzNfMMv2dLizhAT8mEYq)

