# 引言

显示的原理就是通过不断刷新屏幕缓冲区的数据，显示器就可以显示出来。

# 概念

刷新率：
屏幕以一定的频率进行刷新，刷新的过程是先进行逐行扫描，扫描完一帧的数据后就接着扫描下一帧图像的第一行开始重复这样过程就可以形成一个稳定的帧刷新频率。一般情况下是60Hz，即1S中刷新16.6帧图像的速度进行屏幕刷新。用户就可以看到流畅的画面图像，这样的屏幕刷新的速率就叫做帧率

帧率：
是针对图像内存数据而言的。对于Android系统而言，合成和渲染图像数据到内存也是按照一定的速度存放到graphicBuffer的，这个出图的速率就叫做帧率

# 卡顿与丢帧

图像数据是由应用绘制得到，通过系统服务的合成渲染存放到显存，再有屏幕通过刷新读取到到屏幕进行刷新显示，这是显示的大致过程。
那么如果刷新率>帧率，那么就是说，出图的速度低于屏幕刷新的速度，这是就会造成刷新的时候经常出现读取到的图像数据是同一幅图像的数据，因为你出图速度不够所以只能读取旧的数据，这样就出现卡顿的现象。
当刷新率<帧率的时候，就是说出图了很多帧的图像数据，但是因为你的刷新速度跟不上，就不能把所有由服务端输出的图像数据全部读取到屏幕并且刷新显示出来，所以就出现了丢失了一些图像帧数据的现象。

这个就是卡顿丢帧的现象的根本原因就是刷新率和帧率的同步问题引起的。为了解决这个问题就引入了Vsync的机制。
具体Vsync机制是怎么对帧率和刷新率进行同步的呢？
主要是通过一个固定的Vsync信号，来同步应用，系统服务和屏幕显示这几个部分的工作和数据流转的。对应的是绘制，合成渲染，刷新显示的环节。

# Vsync的产生

可以由硬件产生也可以由软件产生，硬件产生的设备是HWC，软件是通过VsyncThrerad线程模拟来产生。产生了Vsync的事件后会通过一个叫做DispSyncThread的线程来控制分发到另外两个叫做
EventThread-app和EventThread-sf的线程，分别提供给应用端app和服务端（SurfaceFlinger）使用。这几个工作线程都是在SurfaceFlinger服务启动初始化init的时候就创建的。EventThread-app和EventThread-sf的线程需要对同步信号进行一些相位的偏移来满足流水线上的配合。VsyncThread就是在没有HWC硬件产生Vsync信号的时候会被创建。

# Vsync的分发过程

硬件产生的Vsync信号时，就会回调mEventHandler.onVSyncReceived去处理任务
软件时通过线程的循环threadloop，执行sleep的操作，当sleep时间到了就回调mEventHandler.onVSyncReceived去处理任务

当Vsync信号来的时候，就会唤醒EventThread线程，执行所有注册到EventThread的connection的分发出去，就是通过BitTube的mSendFd分发出去，分发出去后mReceivedFd就能接收到数据
SurfaceFlinger和App都在监听这个mReceivedFd事件，当到来的时候就可以响应这个事件进行任务处理了。这就是Vsync事件的基本分发流程了。







在Android系统中，Vsync信号的接收是一个涉及多个组件和层次的过程。以下是对Android系统中Vsync信号接收的详细解释：

### 一、Vsync信号的来源

Vsync信号通常由显示设备（如屏幕）产生，并以固定的频率（如60Hz，即每秒60次）发送给Android系统。这个信号标志着显示设备已经完成了当前帧的刷新，并准备开始下一帧的显示。

### 二、Vsync信号的接收与分发

在Android系统中，Vsync信号的接收和分发主要由SurfaceFlinger服务负责。SurfaceFlinger是Android系统中的一个核心服务，负责管理屏幕上的所有图层（Layers）和它们的合成。

1. 硬件产生的Vsync信号
   - 当硬件（如显示设备）产生Vsync信号时，这个信号会被直接发送给Android系统。
   - 在Android系统中，HWC（Hardware Composer）硬件抽象层可能会接收并处理这个Vsync信号。HWC是Android 3.0（Honeycomb）引入的，用于加速显示合成的硬件加速接口。
2. 软件模拟的Vsync信号
   - 如果硬件不支持产生Vsync信号，或者在某些特定情况下，Android系统可能会通过软件来模拟Vsync信号。这通常是通过VsyncThread线程来完成的，该线程会按照一定的时间间隔（如16.67ms，对应60Hz的刷新率）来模拟Vsync信号的产生。
3. Vsync信号的分发
   - 无论是硬件产生的还是软件模拟的Vsync信号，最终都会被分发到Android系统中的相关组件。
   - 在Android系统中，Vsync信号的分发通常是通过DisplayEventReceiver（DER）和Choreographer来完成的。DER是一个用于接收和分发Vsync事件的类，而Choreographer则是一个用于管理动画和输入事件的类。
   - 当Vsync信号到来时，DER会接收到这个信号，并将其分发给Choreographer和其他需要这个信号的组件（如SurfaceFlinger）。
4. SurfaceFlinger的处理
   - SurfaceFlinger在接收到Vsync信号后，会遍历其管理的图层列表，以查找是否有新的图像缓冲区需要被合成到屏幕上。
   - 如果有新的缓冲区可用，SurfaceFlinger会将其与之前的缓冲区进行合成，并将合成后的图像发送到显示设备进行显示。
   - 如果没有新的缓冲区可用，SurfaceFlinger可能会选择继续使用上一个缓冲区，以避免屏幕出现空白或闪烁。

### 三、总结

在Android系统中，Vsync信号的接收是一个涉及硬件、软件以及多个系统组件的复杂过程。这个信号对于保持屏幕显示的流畅性和一致性至关重要，它通过确保GPU渲染的每一帧都与显示器的刷新周期对齐，从而消除了图像撕裂和卡顿现象。





# 参考

[Android图形-Vsync机制](https://blog.csdn.net/haigand/article/details/132239549)