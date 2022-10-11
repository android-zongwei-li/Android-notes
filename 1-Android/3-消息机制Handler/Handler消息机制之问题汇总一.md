#一、Handler
1、消息机制是什么？Handler是什么？
1）在Android中，消息机制主要就是指Handler机制。
2）Handler是Android中的消息机制。它可以发送和处理延时消息/任务，并且可以把消息/任务发送到其他线程（线程切换）。实际开发中，主要是用来解决在子线程中无法更新UI的问题。

2、为什么不能在子线程中访问UI？
ViewRootImpl会对UI操作进行验证，禁止在子线程中访问UI:
```
void checkThread(){
  if(mThread != Thread.currentThread()){
    throw new CalledFromWrongThreadException("Only th original thread that created a view hierarchy can touch its views");
  }
}
```

3、Android为什么要把UI更新限制在主线程，以及Handler的由来？
最根本的目的在于解决：多线程并发问题。
分析：如果在一个Activity中有多个线程，并且没有加锁，那么同时去操作UI时就出现界面错乱的问题。但是如果对这些更新UI的操作都做加锁处理，又会导致性能下降。出于对性能问题的考虑，Android提供了一套更新UI的机制——Handler。我们只需要遵循这种机制，就可以方便的进行UI操作，而不用再去关心多线程的问题，所有更新UI的操作，都是在主线程的消息队列中去轮训的。

4、在子线程中创建Handler报错是为什么?
子线程默认是没有Looper的，进而也就没有MessageQueue。因此要在子线程中创建Handler，那么首先得在子线程中调用Looper.prepare()。
```
public static final void prepare() {  
    if (sThreadLocal.get() != null) {  
        throw new RuntimeException("Only one Looper may be created per thread");  
    }  
    sThreadLocal.set(new Looper(true));  
}  

private Looper(boolean quitAllowed) {  
    mQueue = new MessageQueue(quitAllowed);  
    mRun = true;  
    mThread = Thread.currentThread();  
}  
```
Looper.prepare()后，当前的线程就会和一个Looper关联，进而和MessageQueue关联。然后又因为Handler在创建时，会和Looper关联，所有到此Handler、Looper、MessageQueue、Thread之间一一对应的关系也就建立了。

到这个阶段，我们就可以使用Handler发送消息了。但是，现在只是可以把消息加入到MessageQueue中，想要当前的子线程会去取出消息，还需要执行Looper.loop()。

小结：要在子线程中new Handler就分为三步：
1）Looper.prepare()   --负责为Thread准备MessageQueue。
2）Looper.loop()    ------负责从MessageQueue中取消息。
3）new Handler()。  ---负责消息发送与处理。
其实在主线程也是这样操作的，只不过系统已经帮我们做了前两步了。

5、为什么通过Handler能实现线程的切换？
Handler创建的时候会和Looper进行关联，进而关联了MessageQueue和Thread，并且一个Handler对应的Looper、MessageQueue和Thread是唯一的。因此，不管Handler在什么地方使用（其他线程），它发送的消息都是加入到自己对应的线程/MessageQueue中了，然后取消息也是在它对应的线程，因此也就实现了跨线程。

6、Handler.post的逻辑在哪个线程执行的，是由Looper所在线程还是Handler所在线程决定的？
这个问题，应该可以分为两步：
1）post方法本身是在调用post方法的线程中执行的，一直执行到把消息/Runnable加入到MessageQueue中
2）post出去的Runnable的run()方法，是在Looper所在的线程执行的。在这个线程中Looper.loop()会一直取消息，取出后，调用当前线程对应的Handler.dispatchMessage执行。

7、Looper和Handler一定要处于一个线程吗？子线程中可以用MainLooper去创建Handler吗？
Handler在任何线程创建都可以，它创建的时候默认是和当前线程的Looper关联，但也可以指定Looper。
比如，在子线程中，可以创建一个Handler，然后指定Looper为MainLooper。那么此时这个Handler所发出的消息，都是在主线程收到的。

8、Handler的post/send()的原理
一系列post/send方法最终都是通过enqueueMessage()方法将msg加入到MessageQueue中。

9、Handler的post方法发送的是同步消息吗？可以发送异步消息吗？
1）用户层面发送的都是同步消息，不能发异步消息
2）异步消息只能系统发送。

疑问：$\color{red}{什么是同步消息、异步消息？}$

10、Handler的post()和postDelayed()方法的异同？
1）底层都是调用的sendMessageDelayed()
2）post()传入的时间参数为0
3）postDelayed()传入的时间参数是需要延迟的时间间隔。

11、Handler的postDelayed的底层机制
postDelayed --> sendMessageDelayed -->  sendMessageAtTime -->  enqueueMessage。

12、Handler的dispatchMessage()分发消息的处理流程？
```
public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

13、Handler为什么要有Callback的构造方法？
不需要派生Handler。

14、Handler构造方法中通过Looper.myLooper();是如何获取到当前线程的Looper的？
myLooper()内部使用ThreadLocal实现，因此能够获取各个线程自己的Looper。

15、主线程如何向子线程发送消息？
1）通过在主线程调用子线程中Handler的post方法，完成消息的投递。
2）通过HandlerThread实现该需求。
$\color{red}{HandlerThread是什么？}$

16、MessageQueue.next()会因为发现了延迟消息，而进行阻塞。那么为什么后面加入的非延迟消息没有被阻塞呢？





#二、MessageQueue
1、MessageQueue是什么？
1）消息队列
2）内部存储结构并不是真正的队列，而是用单链表的数据结构来存储消息列表
3）只能存储消息，不能处理消息

2、MessageQueue的主要两个操作是什么?有什么用？
1）enqueueMessage：往消息队列中插入一条消息
2）next：取出一条消息，并从消息队列中移除
3）本质采用单链表的数据结构来维护消息队列，而不是采用队列

3、MessageQueue的enqueueMessage()方法的原理，如何进行线程同步的？
```
boolean enqueueMessage(Message msg, long when) {
    //1. 内部是单链表的插入操作
    synchronized (this) {
        ......
    }
    return true;
}
```
1）单链表的插入操作
2）如果消息队列被阻塞回调用nativeWake去唤醒。
3）用synchronized代码块去进行同步。

4、MessageQueue的next()方法内部的原理？
```
     /**
     * 功能：读取并且删除数据
     * 内部无限循环，如果消息队列中没有消息就会一直阻塞。
     * 一旦有新消息到来，next方法就会返回该消息并且将其从单链表中移除
    */
    Message next() {
        int nextPollTimeoutMillis = 0;
        for (;;) {
            /**======================================================================
             * 1、精确阻塞指定时间。第一次进入时因为nextPollTimeoutMillis=0，因此不会阻塞。
             *   1-如果nextPollTimeoutMillis=-1，一直阻塞不会超时。
             *   2-如果nextPollTimeoutMillis=0，不会阻塞，立即返回。
             *   3-如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)，如果期间有程序唤醒会立即返回。
             *====================================================================*/
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // 当前时间
                final long now = SystemClock.uptimeMillis();
                Message msg = mMessages;
                /**=======================================================================
                 * 2、当前Msg为消息屏障
                 *   1-说明有重要的异步消息需要优先处理
                 *   2-遍历查找到异步消息并且返回。
                 *   3-如果没查询到异步消息，会continue，且阻塞在nativePollOnce直到有新消息
                 *====================================================================*/
                if (msg != null && msg.target == null) {
                   // 遍历寻找到异步消息，或者末尾都没找到异步消息。
                    do {
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                /**================================================================
                 *  3、获取到消息
                 *    1-消息时间已到，返回该消息。
                 *    2-消息时间没到，表明有个延时消息，会修正nextPollTimeoutMillis。
                 *    3-后面continue，精确阻塞在nativePollOnce方法
                 *===================================================================*/
                if (msg != null) {
                    // 延迟消息的时间还没到，因此重新计算nativePollOnce需要阻塞的时间
                    if (now < msg.when) {
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // 返回获取到的消息(可以为一般消息、时间到的延迟消息、异步消息)
                        return msg;
                    }
                } else {
                    /**=============================
                     * 4、没有找到消息或者异步消息
                     *==============================*/
                    nextPollTimeoutMillis = -1;
                }

                /**===========================================
                 * 5、没有获取到消息，进行下一次循环。
                 *   (1)此时可能处于的情况：
                 *      1-没有获取到消息-nextPollTimeoutMillis = -1
                 *      2-没有获取到异步消息(接收到同步屏障却没找到异步消息)-nextPollTimeoutMillis = -1
                 *      3-延时消息的时间没到-nextPollTimeoutMillis = msg.when-now
                 *   (2)根据nextPollTimeoutMillis的数值，最终都会阻塞在nativePollOnce(-1)，
                 *      直到enqueueMessage将消息添加到队列中。
                 *===========================================*/
                if (pendingIdleHandlerCount <= 0) {
                    // 用于enqueueMessage进行精准唤醒
                    mBlocked = true;
                    continue;
                }
            }
        }
    }
```
小结（原理：分为三种情况进行处理）：
1）如果是一般消息，会去获取消息，没有获取到就会阻塞(native方法)，直到enqueueMessage插入新消息。获取到直接返回Msg。
2）如果是同步屏障，会去循环查找异步消息，没有获取到会进行阻塞。获取到直接返回Msg。
3）如果是延时消息，会计算时间间隔，并进行精准定时阻塞(native方法)。直到时间到达或者被enqueueMessage插入消息而唤醒。时间到后就返回Msg。

疑问：$\color{red}{什么是同步屏障}$

5、Looper.loop()是如何阻塞的？MessageQueue.next()是如何阻塞的？
通过native方法：nativePollOnce()进行精准时间的阻塞。





#三、Looper
1、Looper是什么？
1、轮询器。（消息循环）
2、Looper以无限循环的形式去查找是否有新消息，有就处理消息，没有就一直等待着。

2、如何开启消息循环？
Looper.loop()。

3、Looper的构造方法
```
private Looper(boolean quitAllowed) {
    //1. 会创建消息队列: MessageQueue
    mQueue = new MessageQueue(quitAllowed);
    //2. 当前线程
    mThread = Thread.currentThread();
}
```

4、为线程创建Looper
```
//1. 在没有Looper的线程创建Handler会直接异常
new Thread("Thread#2"){
    @Override
    public void run(){
        Handler handler = new Handler();
    }
}.start();
```
>异常:
>java.lang.RuntimeException: Can’t create handler inside thread that has not called Looper.prepare()
```
//2. 用prepare为当前线程创建一个Looper
new Thread("Thread#2"){
    @Override
    public void run(){
        Looper.prepare();
        Handler handler = new Handler();
        //3. 开启消息循环
        Looper.loop();
    }
}.start();
```

5、主线程ActivityThread中的Looper的创建和获取
1）主线程中使用prepareMainLooper()创建Looper
2）getMainLooper能够在任何地方获取到主线程的Looper

6、Looper的两个退出方法？有什么区别？
1）Looper的退出有两个方法：quit和quitSafely
2）quit会直接退出Looper
3）quitSafely只会设置退出标记，在已有消息全部处理完毕后才安全退出
4）Looper退出后，Handler的发送的消息会失败，此时send返回false
5）子线程中如果手动创建了Looper，应该在所有事情完成后调用quit方法来终止消息循环

7、Looper.loop()的源码流程?
1）获取到Looper和消息队列
2）for无限循环，阻塞于消息队列的next方法
3）取出消息后调用msg.target.dispatchMessage(msg)进行消息分发

8、
```
//Looper.java
public static void loop() {
    //1. 获取Looper
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    //2. 获取消息队列
    final MessageQueue queue = me.mQueue;
    ......
    for (; ; ) {
        //3. 获取消息，如果没有消息则会一直阻塞
        Message msg = queue.next();
        /**=================================
         * 4. 如果消息获得为null，则退出循环
         * -Looper退出后，next就会返回null
         *=================================*/
        if (msg == null) {
            return;
        }
        ......
        /**==========================================================
         * 5. 处理消息
         *  -msg.target:是发送消息的Handler
         *  -最终在该Looper中执行了Handler的dispatchMessage()
         *  -成功将代码逻辑切换到指定的Looper(线程)中执行
         *========================================================*/
        msg.target.dispatchMessage(msg);
        ......
    }
}
```

9、Looper.loop()在什么情况下会退出？
1）next方法返回的msg == null
2）线程意外终止

10、Looper.quit/quitSafely的本质是什么？
让消息队列的next()返回null，依次来退出Looper.loop()

11、Looper.loop()方法执行时，如果内部的myLooper()获取不到Looper会出现什么结果?
```
throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
```

12、Android如何保证一个线程最多只能有一个Looper？如何保证只有一个MessageQueue
1）Looper的构造方法是private，不能直接构造。需要通过Looper.prepare()进行创建：
2)在Looper.prepare()中会判断sThreadLocal.get()是否为null，若不是，会抛出异常，从而保证了一个线程最多只能有一个Looper，也保证了只有一个MessageQueue。
```
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

13、Handler消息机制中，一个looper是如何区分多个Handler的？
1）msg的target持有一个发送此消息的Handler引用。
```
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```
2）Looper.loop()会阻塞于MessageQueue.next()
3）取出msg后，msg.target成员变量就是该msg对应的Handler
4）调用msg.target的disptachMessage()进行消息分发。

14、主线程是如何准备消息循环的？
```
//ActivityThread.java
public static void main(String[] args) {
    //1. 创建主线程的Looper和MessageQueue
    Looper.prepareMainLooper();
    ......
    //2. 开启消息循环
    Looper.loop();
}
/**=============================================
 * ActivityThread中需要Handler与消息队列进行交互
 * -内部定义一系列消息类型：主要有四大组件等
 * //ActivityThread.java
 *=============================================*/
private class H extends Handler {
    public static final int LAUNCH_ACTIVITY         = 100;
    public static final int PAUSE_ACTIVITY          = 101;
    public static final int PAUSE_ACTIVITY_FINISHING= 102;
    ......
}
```
>1、ActivityThread通过ApplicationThread和AMS进行IPC通信
>2、AMS完成请求的工作后会回调ApplicationThread中的Binder方法
>3、ApplicationThread会向Handler H发送消息
>4、H接收到消息后会将ApplicationThread的逻辑切换到ActivityThread中去执行

$\color{red}{注：这个题还不懂。}$

15、主线程Looper一直循环查消息为何没卡主线程？
1）线程的阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。
（2）主线程Looper从消息队列读取消息，当读完所有消息时，主线程阻塞。子线程往消息队列发送消息，并且往管道文件写数据，主线程即被唤醒，从管道文件读取数据，主线程被唤醒只是为了读取消息，当消息读取完毕，再次睡眠。因此loop的循环并不会对CPU性能有过多的消耗。





#四、ThreadLocal
1、ThreadLocal是什么？有什么用？
ThreadLocal是线程内部的数据存储类，可以在指定线程中存储数据，之后只有在指定线程中才可以读取到存储的数据

2、ThreadLocal的两个应用场景？
1）某些数据是以线程为作用域，并且不同线程具有不同的数据副本的时候。ThreadLocal可以轻松实现Looper在线程中的存取。
2）在复杂逻辑下的对象传递，通过ThreadLocal可以让对象成为线程内的全局对象，线程内部通过get就可以获取。

3、ThreadLocal的使用
```
mBooleanThreadLocal.set(true);
Log.d("ThreadLocal", "[Thread#main]" + mBooleanThreadLocal.get());

new Thread("Thread#1"){
    @Override
    public void run(){
        mBooleanThreadLocal.set(false);
        Log.d("ThreadLocal", "[Thread#1]" + mBooleanThreadLocal.get());
    }
}.start();

new Thread("Thread#2"){
    @Override
    public void run(){
        Log.d("ThreadLocal", "[Thread#2]" + mBooleanThreadLocal.get());
    }
}.start();
```
>1、最终main中输出true; Thread#1中输出false; Thread#2中输出null
>2、ThreadLocal内部会从各自线程中取出数组，再根据当前ThreadLocal的索引去查找出对应的value值。

4、ThreadLocal的set()源码分析
```
//ThreadLocal.java
public void set(T value) {
    //1. 获取当前线程
    Thread t = Thread.currentThread();
    //2. 获取当前线程对应的ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null)
        //3. map存在就进行存储
        map.set(this, value);
    else
        //4. 不存在就创建map并且存储
        createMap(t, value);
}
//ThreadLocal.java内部类: ThreadLocalMap
private void set(ThreadLocal<?> key, Object value) {
    //1. table为Entry数组
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    //2. 根据当前ThreadLocal获取到Hash key，并以此从table中查询出Entry
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //3. 如果Entry的ThreadLocal与当前的ThreadLocal相同，则用新值覆盖e的value
        if (k == key) {
            e.value = value;
            return;
        }
        //4. Entry没有ThreadLocal则把当前ThreadLocal置入，并存储value
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    //5. 没有查询到Entry，则新建Entry并且存储value
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
//ThreadLocal内部类ThreadLocalMap的静态内部类
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

5、ThreadLocal的get()源码分析
```
public T get() {
    //1. 获取当前线程对应的ThreadLocalMap
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //2. 取出map中的对应该ThreadLocal的Entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            //3. 获取到entry后返回其中的value
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //4. 没有ThreadLocalMap或者没有获取到ThreadLocal对应的Entry，返回规定数值
    return setInitialValue();
}
private T setInitialValue() {
    //1. value = null
    T value = initialValue();//返回null
    Thread t = Thread.currentThread();
    //2. 若不存在则新ThreadLocalMap, 在里面以threadlocal为key,value为值,存入entry
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
>1、当前线程对应了一个ThreadLocalMap
>2、当前线程的ThreadLocal对应一个Map中的Entry(存在table中)
>3、Entry中key会获取其对应的ThreadLocal, value就是存储的数值

6、ThreadLocal的原理
>1、thread.threadLocals就是当前线程thread中的ThreadLocalMap
>2、ThreadLocalMap中有一个table数组，元素是Entry。根据ThreadLocal(需要转换获取到Hash Key)能get到对应的Enrty。
>3、Entry中key为ThreadLocal, value就是存储的数值。

#五、内存泄漏
1、Handler的内存泄漏如何避免？
>1、采用静态内部类：static handler = xxx
>2、Activity结束时，调用handler.removeCallback()、然后handler设置为null
>3、如果使用到Context等引用，要使用弱引用

单独写一篇分析。

可以参考：
https://blog.csdn.net/qq_37321098/article/details/81535449?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param

https://blog.csdn.net/wsq_tomato/article/details/80301851?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param

参考：
1、[Handler消息机制(50题)](https://blog.csdn.net/feather_wch/article/details/81136078?utm_medium=distribute.pc_relevant_right.none-task-blog-BlogCommendFromMachineLearnPai2-4.channel_param_right&depth_1-utm_source=distribute.pc_relevant_right.none-task-blog-BlogCommendFromMachineLearnPai2-4.channel_param_right)
主要就是参考了他的博客，可以算是转载了吧，他的源码分析讲的挺清楚的。