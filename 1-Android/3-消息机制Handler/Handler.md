主要场景是子线程完成耗时操作的过程中，通过 Handler 向主线程发送消息 Message，用来刷新 UI 界面。

了解 Handler 的发送消息和处理消息的源码实现。

# 从 new Handler() 开始

```kotlin
    final Looper mLooper;
    final MessageQueue mQueue;

    public Handler() {
        this(null, false);
    }

    public Handler(@Nullable Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
		
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

![image-20240704180957828](images/Handler/image-20240704180957828.png)

在无参构造器里调用了重载的构造方法并分别传入 null 和 false。并且在构造方法中给两个全局变量赋值：mLooper 和 mQueue。

这两者都是通过 Looper 来获取，具体代码如下：

![Drawing 1.png](images/Handler/CgqCHl7fQeSAQBZAAAB4NOhhh9U599.png)

可以看出，myLooper 通过一个线程本地变量中的存根，然后 mQueue 是 Looper 中的一个全局变量，类型是 MessageQueue 类型。

接下来的分析重点就是这个 Looper 是什么？以及何时被初始化？

# Looper 介绍

不知你有没有思考过一个问题，启动一个 Java 程序的入口函数是 main 方法，但是当 main 函数执行完毕之后此程序停止运行，也就是进程会自动终止。但是当我们打开一个 Activity 之后，只要我们不按下返回键 Activity 会一直显示在屏幕上，也就是 Activity 所在进程会一直处于运行状态。实际上 Looper 内部维护一个无限循环，保证 App 进程持续进行。

## Looper初始化

ActivityThread 的 main 方法是一个新的 App 进程的入口，其具体实现如下：

![Drawing 2.png](images/Handler/Ciqc1F7fQfGAVPtfAAJYK7Tmh_o873.png)

图中 1 处就是初始化当前进程的 Looper 对象；
图中 2 处调用 Looper 的 loop 方法开启无限循环。

prepareMainLooper 方法如下：

![Drawing 3.png](images/Handler/CgqCHl7fQjWAR3Q-AAK5dlc_5mM274.png)

图中 1 处在 prepareMainLooper 中调用 prepare 方法创建 Looper 对象，仔细查看发现其实就是 new 出一个 Looper。核心之处在于将 new 出的 Looper 设置到了线程本地变量 sThreadLocal 中。也就是说创建的 Looper 与当前线程发生了绑定。

Looper 的构造方法如下：

![Drawing 4.png](images/Handler/CgqCHl7fQjyAeVt9AABjclbZivQ040.png)

可以看出，在构造方法中初始化了消息队列 MessageQueue 对象。

prepare 方法执行完之后，会在图中 3 处调用 myLooper() 方法，从 sThreadLocal 中取出 Looper 对象并赋值给 sMainLooper 变量。

![Drawing 5.png](images/Handler/CgqCHl7fQkWASNQUAABHck7yJJQ286.png)

注意：

图中 2 处在创建 Looper 对象之前，会判断 sThreaLocal 中是否已经绑定过 Looper 对象，如果是则抛出异常。这行代码的目的是确保在一个线程中 Looper.prepare() 方法只能被调用 1 次。比如以下代码：

![Drawing 6.png](images/Handler/Ciqc1F7fQkyAbmLLAADFNEwNOZw252.png)

执行上述代码程序会秒崩，打印日志如下：

![Drawing 7.png](images/Handler/Ciqc1F7fQlOAVw0MAAKdzfUus70602.png)

注意：

不是说调用 2 次 prepare 才会抛异常吗？为什么 MainActivity 中只调用了 1 遍就导致程序崩溃？ 这是因为在 MainActivity 所在进程被创建时，Looper 的 prepare 方法已经在 main 方法中调用了 1 遍。这会直接导致一个非常重要的结果：

prepare 方法在一个线程中只能被调用 1 次；
Looper 的构造方法在一个线程中只能被调用 1 次；
最终导致 MessageQueue 在一个线程中只会被初始化 1 次。

也就是说 UI 线程中只会存在 1 个 MessageQueue 对象，后续我们通过 Handler 发送的消息都会被发送到这个 MessageQueue 中。

## Looper 负责做什么事情

用一句话总结 Looper 做的事情就是：不断从 MessageQueue 中取出 Message，然后处理 Message 中指定的任务。

在 ActivityThread 的 main 方法中，除了调用 Looper.prepareMainLooper 初始化 Looper 对象之外，还调用了 Looper.loop 方法开启无限循环，Looper 的主要功能就是在这个循环中完成的。

![Drawing 8.png](images/Handler/Ciqc1F7fQlyAJUK5AAO0v8INUxQ853.png)

很显然，loop 方法中执行了一个死循环，这也是一个 Android App 进程能够持续运行的原因。

图中 1 处不断地调用 MessageQueue 的 next 方法取出 Message。如果 message 不为 null 则调用图中 2 处进行后续处理。具体就是从 Message 中取出 target 对象，然后调用其 dispatchMessage 方法处理 Message 自身。那这个 target 是谁呢？查看 Message.java 源码可以看出 target 就是 Handler 对象，如下所示：

![Drawing 9.png](images/Handler/CgqCHl7fQmOAVKfEAACBfkLYTLI319.png)

Handler 的 dispatchMessage 方法如下：

![Drawing 10.png](images/Handler/Ciqc1F7fQmmANSf3AAEB0-dfs24152.png)

可以看出，在 dispatchMessage 方法中会调用一个空方法 handleMessage，而这个方法也正是我们创建 Handler 时需要覆盖的方法。那么 Handler 是何时将其设置为一个 Message 的 target 的呢？

## Handler 的 sendMessage 方法

Handler 有几个重载的 sendMessage 方法，但是基本都大同小异。我用最普通的 sendMessage 方法来分析，代码具体如下：

![Drawing 11.png](images/Handler/CgqCHl7fQnGAOpvAAAHydqxDjMQ700.png)

可以看出，经过几层调用之后，sendMessage 最终会调用 enqueueMessage 方法将 Message 插入到消息队列 MessageQueue 中。而这个消息队列就是我们刚才分析的在 ActivityThread 的 main 方法中通过 Looper 创建的 MessageQueue。

Handler 的 enqueueMessage 方法

![Drawing 12.png](images/Handler/Ciqc1F7fQnmALaGYAAULXonPvxo483.png)

在图中 1 处 enqueueMessage 方法中，将 Handler 自身设置为 Message的target 对象。因此后续 Message 会调用此 Handler 的 dispatchMessage 来处理；
图中 2 处会判断如果 Message 中的 target 没有被设置，则直接抛出异常；
图中 3 处会按照 Message 的时间 when 来有序得插入 MessageQueue 中，可以看出 MessageQueue 实际上是一个有序队列，只不过是按照 Message 的执行时间来排序。

至此 Handler 的发送消息和消息处理流程已经介绍完毕，接下来看几个面试中经常被问到的与 Handler 相关的题目。

## Handler 的 post(Runnable) 与 sendMessage 有什么区别

看一下 post(Runnable) 的源码实现如下：

![Drawing 13.png](images/Handler/Ciqc1F7fQoSAbccXAAC4I5kW5RU026.png)

实际上 post(Runnable) 会将 Runnable 赋值到 Message 的 callback 变量中，那么这个 Runnable 是在什么地方被执行的呢？Looper 从 MessageQueue 中取出 Message 之后，会调用 dispatchMessage 方法进行处理，再看下其实现：

![Drawing 14.png](images/Handler/CgqCHl7fQoqAdt7dAAEcRaEW5Ac045.png)

可以看出，dispatchMessage 分两种情况：

如果 Message 的 Callback 不为 null，一般为通过 post(Runnabl) 方式，会直接执行 Runnable 的 run 方法。因此这里的 Runnable 实际上就是一个回调接口，跟线程 Thread 没有任何关系。
如果 Message 的 Callback 为 null，这种一般为 sendMessage 的方式，则会调用 Handler 的 hanlerMessage 方法进行处理。

## Looper.loop() 为什么不会阻塞主线程

Looper 中的 loop 方法实际上是一个死循环。但是我们的 UI 线程却并没有被阻塞，反而还能够进行各种手势操作，这是为什么呢？在 MessageQueue 的 next 方法中，有如下一段代码：

![Drawing 15.png](images/Handler/Ciqc1F7fQpGAKsbzAAEINuv-lzg977.png)

nativePollOnce 方法是一个 native 方法，当调用此 native 方法时，主线程会释放 CPU 资源进入休眠状态，直到下条消息到达或者有事务发生，通过往 pipe 管道写端写入数据来唤醒主线程工作，这里采用的 epoll 机制。关于 nativePollOnce 的详细分析可以参考：[nativePollOnce函数分析](https://www.kancloud.cn/alex_wsc/android-deep2/413394)。

## Handler 的 sendMessageDelayed 或者 postDelayed 是如何实现的

在向 MessageQueue 队列中插入 Message 时，会根据 Message 的执行时间排序。而消息的延时处理的核心实现是在获取 Message 的阶段，接下来看下 MessageQueue 的 next 方法。

![Drawing 16.png](images/Handler/CgqCHl7fQpyABhRKAAHHTc9iME0070.png)

图中蓝框处表示从 MessageQueue 中取出一个 Message，但是当前的系统时间小于 Message.when，因此会计算一个 timeout，目的是实现在 timeout 时间段后再将 UI 线程唤醒，因此后续处理 Message 的代码只会在 timeout 时间之后才会被 CPU 执行。

> 注意：在上述代码中也能看出，如果当前系统时间大于或等于 Message.when，那么会返回 Message 给 Looper.loop()。但是这个逻辑只能保证在 when 之前消息不被处理，不能够保证一定在 when 时被处理。

# 总结

- 应用启动是从 ActivityThread 的 main 开始的，先是执行了 Looper.prepare()，该方法先是 new 了一个 Looper 对象，在私有的构造方法中又创建了 MessageQueue 作为此 Looper 对象的成员变量，Looper 对象通过 ThreadLocal 绑定 MainThread 中；
- 当我们创建 Handler 子类对象时，在构造方法中通过 ThreadLocal 获取绑定的 Looper 对象，并获取此 Looper 对象的成员变量 MessageQueue 作为该 Handler 对象的成员变量；
- 在子线程中调用上一步创建的 Handler 子类对象的 sendMesage(msg) 方法时，在该方法中将 msg 的 target 属性设置为自己本身，同时调用成员变量 MessageQueue 对象的 enqueueMessag() 方法将 msg 放入 MessageQueue 中；
- 主线程创建好之后，会执行 Looper.loop() 方法，该方法中获取与线程绑定的 Looper 对象，继而获取该 Looper 对象的成员变量 MessageQueue 对象，并开启一个会阻塞（不占用资源）的死循环，只要 MessageQueue 中有 msg，就会获取该 msg，并执行 msg.target.dispatchMessage(msg) 方法（msg.target 即上一步引用的 handler 对象），此方法中调用了我们第二步创建 handler 子类对象时覆写的 handleMessage() 方法。