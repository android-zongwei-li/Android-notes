第10讲：深入理解 AQS 和 CAS 原理



AQS 全称是 Abstract Queued Synchronizer，一般翻译为同步器。它是一套实现多线程同步功能的框架，由大名鼎鼎的 Doug Lea 操刀设计并开发实现的。AQS 在源码中被广泛使用，尤其是在 JUC（Java Util Concurrent）中，比如 ReentrantLock、Semaphore、CountDownLatch、ThreadPoolExecutor。理解 AQS 对我们理解 JUC 中其他组件至关重要，并且在实际开发中也可以通过自定义 AQS 来实现各种需求场景。

注意：理解 AQS 需要一定的数据结构基础，尤其是双端队列，并对 Unsafe 有一定的了解。

ReentrantLock 和 AQS 的关系
本课时我们主要通过 ReentrantLock 来理解 AQS 内部的工作机制。首先从 ReentrantLock 的 lock() 方法开始：

![安卓1.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hNzeAGmpiAAFQGv9Fgr8055.png)

代码很简单，只是调用了一个 Sync 的 lock() 方法，这个 Sync 是什么呢？

![安卓2.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN0CAUaRQAAGpSG_ngyM130.png)


可以看出，Sync 是 ReentrantLock 中的一个内部类。ReentrantLock 并没有直接继承 AQS，而是通过内部类 Sync 来扩展 AQS 的功能，然后 ReentrantLock 中存有 Sync 的全局变量引用。

Sync 在 ReentrantLock 有两种实现：NonfairSync 和 FairSync，分别对应非公平锁和公平锁。以非公平锁为例，实现源码如下：

![安卓3.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN1SAJgt_AAFXxMZAJfA696.png)


可以看出在非公平锁中的 lock() 方法中，主要做了如下操作：

- 如果通过 CAS 设置变量 State（同步状态）成功，表示当前线程获取锁成功，则将当前线程设置为独占线程。
- 如果通过 CAS 设置变量 State（同步状态）失败，表示当前锁正在被其他线程持有，则进入 Acquire 方法进行后续处理。

acruire() 方法定义在 AQS 中，具体如下：

![安卓4.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN26AQVzmAACP7w8ZKD8422.png)


acquire() 方法是一个比较重要的方法，可以将其拆解为 3 个主要步骤：

1. tryAcquire() 方法主要目的是尝试获取锁；
2. addWaiter() 如果 tryAcquire() 尝试获取锁失败则调用 addWaiter 将当前线程添加到一个等待队列中；
3. acquireQueued 处理加入到队列中的节点，通过自旋去尝试获取锁，根据情况将线程挂起或者取消。

以上 3 个方法都被定义在 AQS 中，但其中 tryAcquire() 有点特殊，其实现如下：

![安卓5.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/CgoCgV6hN3eAU2WmAAB3xJkEMNE965.png)


默认情况下直接抛异常，因此它需要在子类中复写，也就是说真正的获取锁的逻辑由子类同步器自己实现。

ReentrantLock 中 tryAcquire 的实现（非公平锁）如下：

![安卓6.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN3-AC68pAANyZwCuYXA396.png)


解释说明：

- 获取当前线程，判断当前的锁的状态；
- 如果 state=0 表示当前是无锁状态，通过 cas 更新 state 状态的值，返回 true；
- 如果当前线程属于重入，则增加重入次数，返回 true；
- 上述情况都不满足，则获取锁失败返回 false。

最后用一张图表示 ReentrantLock.lock() 过程：

![安卓7.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN4qAAZadAADaSkC9FmM625.png)


从图中我们可以看出，在 ReentrantLock 执行 lock() 的过程中，大部分同步机制的核心逻辑都已经在 AQS 中实现，ReentrantLock 自身只要实现某些特定步骤下的方法即可，这种设计模式叫作模板模式。如果你做过 Android 开发对这一模式应该非常熟悉。Activity 的生命周期的执行流程都已经在 framework 中定义好了，子类 Activity 只要在相应的 onCreate、onPause 等生命周期方法中提供相应的实现即可。

> 注意：不只 ReentrantLock，JUC 包中其他组件例如 CountDownLatch、Semaphor 等都是通过一个内部类 Sync 来继承 AQS，然后在内部中通过操作 Sync 来实现同步。这种做法的好处是将线程控制的逻辑控制在 Sync 内部，而对外面向用户提供的接口是自定义锁，这种聚合关系能够很好的解耦两者所关注的逻辑。
>

# AQS 核心功能原理分析

首先看下 AQS 中几个关键的属性，如下所示：

![安卓8.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/CgoCgV6hN5SAetbZAACffASkCF4632.png)


代码中展示了 AQS 中两个比较重要的属性 Node 和 state。

## state 锁状态

state 表示当前锁状态。当 state = 0 时表示无锁状态；当 state>0 时，表示已经有线程获得了锁，也就是 state=1，如果同一个线程多次获得同步锁的时候，state 会递增，比如重入 5 次，那么 state=5。 而在释放锁的时候，同样需要释放 5 次直到 state=0，其他线程才有资格获得锁。

state 还有一个功能是实现锁的独占模式或者共享模式。

- 独占模式：只有一个线程能够持有同步锁。

比如在独占模式下，我们可以把 state 的初始值设置成 0，当某个线程申请锁对象时，需要判断 state 的值是不是 0，如果不是 0 的话意味着其他线程已经持有该锁，则本线程需要阻塞等待。

- 共享模式：可以有多个线程持有同步锁。

在共享模式下的道理也差不多，比如说某项操作我们允许 10 个线程同时进行，超过这个数量的线程就需要阻塞等待。那么只需要在线程申请对象时判断 state 的值是否小于 10。如果小于 10，就将 state 加 1 后继续同步语句的执行；如果等于 10，说明已经有 10 个线程在同时执行该操作，本线程需要阻塞等待。

## Node 双端队列节点

Node 是一个先进先出的双端队列，并且是等待队列，当多线程争用资源被阻塞时会进入此队列。这个队列是 AQS 实现多线程同步的核心。

从之前 ReentrantLock 图中可以看到，在 AQS 中有两个 Node 的指针，分别指向队列的 head 和 tail。

Node 的主要结构如下：

![安卓9.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/CgoCgV6hN56AWeUQAAI8w2N7emE340.png)


默认情况下，AQS 中的链表结构如下图所示：

![安卓10.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN6eAaiC5AAAjUQaQ1Js892.png)

## 获取锁失败后续流程分析

锁的意义就是使竞争到锁对象的线程执行同步代码，多个线程竞争锁时，竞争失败的线程需要被阻塞等待后续唤醒。那么 ReentrantLock 是如何实现让线程等待并唤醒的呢？

前面中我们提到在 ReentrantLock.lock() 阶段，在 acquire() 方法中会先后调用 tryAcquire、addWaiter、acquireQueued 这 3 个方法来处理。tryAcquire 在 ReentrantLock 中被复写并实现，如果返回 true 说明成功获取锁，就继续执行同步代码语句。可是如果 tryAcquire 返回 false，也就是当前锁对象被其他线程所持有，那么当前线程会被 AQS 如何处理呢？

## addWaiter

首先当前获取锁失败的线程会被添加到一个等待队列的末端，具体源码如下：

![安卓11.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN7CAKiQAAARQFTwNKQg227.png)


有两种情况会致使插入队列失败：

- tail 为空：说明队列从未初始化，因此需要调用 enq 方法在队列中插入一个空的 Node；
- compareAndSetTail 失败：说明插入过程中有线程修改了此队列，因此需要调用 enq 将当前 node 重新插入到队列末端。

经过 addWaiter 方法之后，此时线程以 Node 的方式被加入到队列的末端，但是线程还没有被执行阻塞操作，真正的阻塞操作是在下面的 acquireQueued 方法中判断执行。

## acquireQueued

在 acquireQueued 方法中并不会立即挂起该节点中的线程，因此在插入节点的过程中，之前持有锁的线程可能已经执行完毕并释放锁，所以这里使用自旋再次去尝试获取锁（不放过任何优化细节）。如果自旋操作还是没有获取到锁！那么就将该线程挂起（阻塞），该方法的源码如下：

![安卓12.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/CgoCgV6hN7qAXkEXAAQrReei3G0475.png)

可以看出在 shouldParkAfterFailedAcquire 方法中会判读当前线程是否应该被挂起，其代码如下：

![安卓13.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN8OAIK5GAAOTQNP0ki8128.png)

首先获取前驱节点的 waitStatus 值，Node 中的 waitStatus 一共有 5 种取值，分别代表的意义如下：

| waitStatue值   | 描述                                                         |
| :------------- | ------------------------------------------------------------ |
| CANCELLED (1)  | 当前线程因为超时或者中断被取消。这是一个终结态，也就是状态到此为止 |
| SIGNAL (-1)    | 当前线程的后继线程被阻塞或者即将被阻塞，当前线程释放锁或者取消后需要唤醒后继线程。这个状态一般都是后继线程来设置前驱节点的 |
| CONDITION (-2) | 当前线程在 condition 队列中                                  |
| PROPAGATE (-3) | 用于将唤醒后继线程传递下去，这个状态的引入是为了完善和增强共享锁的唤醒机制。在一个节点成为头节点之前，是不会跃迁为此状态的 |
| 0              | 表示无锁状态                                                 |

接下来根据 waitStatus 不同的值进行不同的操作，主要有以下几种情况：

- 如果 waitStatus 等于 SIGNAL，返回 true 将当前线程挂起，等待后续唤醒操作即可。
- 如果 waitStatus 大于 0 也就是 CANCLE 状态，会将此前驱节点从队列中删除，并在循环中逐步寻找下一个不是“CANCEL”状态的节点作为当前节点的前驱节点。
- 如果 waitStatus 既不是 SIGNAL 也不是 CANCEL，则将当前节点的前驱节点状态设置为 SIGNAL，这样做的好处是下一次执行 shouldParkAfterFailedAcquire 时可以直接返回 true，挂起线程。

代码再回到 acquireQueued 中，如果 shouldParkAfterFailedAcquire 返回 true 表示线程需要被挂起，那么会继续调用 parkAndCheckInterrupt 方法执行真正的阻塞线程代码，具体如下：

![安卓14.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN86AVsTxAAFr5n_07eo859.png)


这个方法比较简单，只是调用了 LockSupport 中的 park 方法。在 LockSupport.park() 方法中调用了 Unsafe API 来执行底层 native 方法将线程挂起，代码到这已经到了操作系统的层面，没有必要再深入分析。

至此，获取锁的大体流程已经分析完毕，总结一下整个过程如下：

- AQS 的模板方法 acquire 通过调用子类自定义实现的 tryAcquire 获取锁；
- 如果获取锁失败，通过 addWaiter 方法将线程构造成 Node 节点插入到同步队列队尾；
- 在 acquirQueued 方法中以自旋的方法尝试获取锁，如果失败则判断是否需要将当前线程阻塞，如果需要阻塞则最终执行 LockSupport(Unsafe) 中的 native API 来实现线程阻塞。

## 释放锁流程分析

在上面加锁阶段被阻塞的线程需要被唤醒过后才可以重新执行。那具体 AQS 是何时尝试唤醒等待队列中被阻塞的线程呢？

同加锁过程一样，释放锁需要从 ReentrantLock.unlock() 方法开始：

![安卓15.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN9mAdr9LAAFZOXdF9Gs514.png)

可以看出，首先调用 tryRelease 方法尝试释放锁，如果成功最终会调用 AQS 中的 unparkSuccessor 方法来实现释放锁的操作。unparkSuccessor 的具体实现如下：

![安卓16.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/CgoCgV6hN-GAFxFeAALtBh7FKak896.png)


解释说明：

首先获取当前节点（实际上传入的是 head 节点）的状态，如果 head 节点的下一个节点是 null，或者下一个节点的状态为 CANCEL，则从等待队列的尾部开始遍历，直到寻找第一个 waitStatus 小于 0 的节点。

如果最终遍历到的节点不为 null，再调用 LockSupport.unpark 方法，调用底层方法唤醒线程。 至此，线程被唤醒的时机也分析完毕。

## 不得不说的 CAS

不管是在加锁还是释放锁阶段，多次提到了一种通用的操作：compareAndSetXXX。这种操作最终会调用 Unsafe 中的 API 进行 CAS 操作。

CAS 全称是 Compare And Swap，译为比较和替换，是一种通过硬件实现并发安全的常用技术，底层通过利用 CPU 的 CAS 指令对缓存加锁或总线加锁的方式来实现多处理器之间的原子操作。

它的实现过程主要有 3 个操作数：内存值 V，旧的预期值 E，要修改的新值 U，当且仅当预期值 E和内存值 V 相同时，才将内存值 V 修改为 U，否则什么都不做。

CAS 底层会根据操作系统和处理器的不同来选择对应的调用代码，以 Windows 和 X86 处理器为例，如果是多处理器，通过带 lock 前缀的 cmpxchg 指令对缓存加锁或总线加锁的方式来实现多处理器之间的原子操作；如果是单处理器，通过 cmpxchg 指令完成原子操作。

## 自定义 AQS

理解了 AQS 的设计思路，接下来我们就可以通过自定义 AQS 来实现自己的同步实现机制。

![安卓17.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN-6AKAxBAAKlvS870jk917.png)

代码中的 MyLock 就是一个最简单的独占锁，通过使用 MyLock 也能实现同 synchronized 和 ReentrantLock 相同的功能。比如如下代码：

![安卓18.png](images/未学-第10讲：深入理解 AQS 和 CAS 原理/Ciqah16hN_iAfYSJAAMCMZKMFfk271.png)


最终打印的 count 值为 20000，说明两个线程之间是线程安全的同步操作。

# 总结

总体来说，AQS 是一套框架，在框架内部已经封装好了大部分同步需要的逻辑，在 AQS 内部维护了一个状态指示器 state 和一个等待队列 Node，而通过 state 的操作又分为两种：独占式和共享式，这就导致 AQS 有两种不同的实现：独占锁（ReentrantLock 等）和分享锁（CountDownLatch、读写锁等）。本课时主要从独占锁的角度深入分析了 AQS 的加锁和释放锁的流程。

理解 AQS 的原理对理解 JUC 包中其他组件实现的基础有帮助，并且理解其原理才能更好的扩展其功能。上层开发人员可以基于此框架基础上进行扩展实现适合不同场景、不同功能的锁。其中几个有可能需要子类同步器实现的方法如下。

- lock()。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回 true，失败则返回 false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回 true，失败则返回 false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回 true，否则返回 false。