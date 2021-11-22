> version：2021/10/17
>
> review：2021/4/27--2021/10/17



目录

[TOC]



有时仅仅为了读写一个或者两个实例域就使用同步的话，显得开销过大；而 volatile 关键字为实例域的同步访问提供了免锁的机制。如果声明一个域为 volatile，那么编译器和虚拟机就知道该域是可能被另一个线程并发更新的。再讲到 volatile 关键字之前，我们需要了解一下内存模型的相关概念以及并发编程中的 3 个特性：原子性、可见性和有序性。

# 一、Java 内存模型

Java 中的堆内存用来存储对象实例，堆内存是被所有线程共享的运行时内存区域，因此，它存在内存可见性的问题。而局部变量、方法定义的参数则不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响。Java 内存模型定义了线程和主存之间的抽象关系：线程之间的共享变量存储在主存中，每个线程都有一个私有的本地内存，本地内存中存储了该线程共享变量的副本。需要注意的是本地内存是 Java 内存模型的一个抽象概念，其并不真实存在，它涵盖了缓存、写缓冲区、寄存器等区域。Java 内存模型控制线程之间的通信，它决定一个线程对主存共享变量的写入何时对另一个线程可见。Java 内存模型的抽象示意图如图所示：

![Java 内存模型的抽象示意图](images/image-20210426171853183.png)

线程 A 与线程 B 之间若要通信的话，必须要经历下面两个步骤：

（1）线程 A 把线程 A 本地内存中更新过的共享变量刷新到主存中去。

（2）线程 B 到主存中去读取线程 A 之前已更新过的共享变量。由此可见，如果我们执行下面的语句：

```
int i=3；
```

执行线程必须先在自己的工作线程中对变量 i 所在的缓存行进行赋值操作，然后再写入主存当中，而不是直接将数值 3 写入主存当中。

# 二、原子性、可见性和有序性

Java 语言对原子性、可见性以及有序性提供了哪些保证呢？首先我们要了解一下这 3 个特性。

## （1）原子性

​		对基本数据类型变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行完毕，要么就不执行。现在看一下下面的代码，如下所示：

```
x = 3； //语句 1
y = x； //语句 2
x++； //语句 3
```

在上面 3 个语句中，只有语句 1 是原子性操作，其他两个语句都不是原子性操作。语句 2 虽说很短，但它包含了两个操作，它先读取 x 的值，再将 x 的值写入**工作内存**。读取 x 的值以及将 x 的值写入工作内存这两个操作单拿出来都是原子性操作，但是合起来就不是原子性操作了。语句 3 包括 3 个操作：读取 x 的值、对 x 的值进行加 1、向工作内存写入新值。通过这 3 个语句我们得知，一个语句含有多个操作时，就不是原子性操作，只有简单地读取和赋值（将数字赋值给某个变量）才是原子性操作。java.util.concurrent.atomic 包中有很多类使用了很高效的机器级指令（而不是使用锁）来保证其他操作的原子性。例如 AtomicInteger 类提供了方法incrementAndGet 和 decrementAndGet，它们分别以原子方式将一个整数自增和自减。可以安全地使用 AtomicInteger 类作为共享计数器而无须同步。另外这个包还包含 AtomicBoolean、AtomicLong 和 AtomicReference 这些原子类，这仅供开发并发工具的系统程序员使用，应用程序员不应该使用这些类。

## （2）可见性

​		可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是一个线程修改的结果，另一个线程马上就能看到。当一个共享变量被 volatile 修饰时，它会保证修改的值立即被更新到主存，所以对其他线程是可见的。**当有其他线程需要读取该值时**，其他线程会去主存中读取新值。而**普通的共享变量不能保证可见性，因为普通共享变量被修改之后，并不会立即被写入主存，何时被写入主存也是不确定的**。当其他线程去读取该值时，此时主存中可能还是原来的旧值，这样就无法保证可见性。

## （3）有序性

​		Java 内存模型中允许编译器和处理器对指令进行**重排序**，虽然重排序过程不会影响到单线程执行的正确性，但是会影响到多线程并发执行的正确性。这时可以通过 volatile 来保证有序性，除了 volatile，也可以通过 synchronized 和 Lock 来保证有序性。我们知道，synchronized 和 Lock保证每个时刻只有一个线程执行同步代码，这相当于是让线程顺序执行同步代码，从而保证了有序性。

# 三、volatile

​		当一个共享变量被 volatile 修饰之后，其就具备了两个含义，一个是**线程修改了变量的值时，变量的新值对其他线程是立即可见**的。换句话说，就是不同线程对这个变量进行操作时具有可见性。另一个含义是**禁止使用指令重排序**。

> 这里提到了重排序，那么什么是重排序呢？重排序通常是编译器或运行时环境为了优化程序性能而采取的对指令进行重新排序执行的一种手段。重排序分为两类：编译期重排序和运行期重排序，分别对应编译时和运行时环境。

下面我们来看一段代码，假设线程 1 先执行，线程 2 后执行，如下所示：

```
//线程 1
boolean stop = false；
while(!stop){
 //doSomething
}

//线程 2
stop = true；
```

很多开发人员在中断线程时可能会采用这种方式。但是这段代码不一定会将线程中断。虽说无法中断线程这个情况出现的概率很小，但是一旦发生这种情况就会造成死循环。为何有可能无法中断线程？在前面我提到每个线程在运行时都有私有的工作内存，因此线程 1 在运行时会将 stop 变量的值复制一份放在私有的工作内存中。当线程 2 更改了 stop 变量的值之后，线程 2 突然需要去做其他的操作，这时就无法将更改的 stop 变量写入主存当中，这样线程 1 就不会知道线程 2 对 stop 变量进行了更改，因此线程 1 就会一直循环下去。当 stop 用 volatile 修饰之后，那么情况就变得不同了，当线程 2 进行修改时，会强制将修改的值立即写入主存，并且会导致线程 1 的工作内存中变量 stop 的缓存行无效，这样线程 1 再次读取变量 stop 的值时就会去主存读取。

## 1、volatile 不保证原子性

我们知道 volatile 保证了操作的可见性，下面我们来分析 volatile 是否能保证对变量的操作是原子性的。现在先阅读以下代码：

```java
public class VolatileTest {
    public volatile int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) {
        final VolatileTest test = new VolatileTest();
        for (int i = 0; i < 10; i++) {
            new Thread() {
                public void run() {
                    for (int j = 0; j < 1000; j++)
                        test.increase();
                }
            }.start();
        }
        //如果有子线程就让出资源，保证所有子线程都执行完
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(test.inc);
    }
}
```

这段代码每次运行，结果都不一致。在前面已经提到过，自增操作是不具备原子性的，它包括读取变量的原始值、进行加 1、写入工作内存。也就是说，自增操作的 3 个子操作可能会分割开执行。假如某个时刻变量 inc 的值为 9，线程 1 对变量进行自增操作，线程 1 先读取了变 量 inc 的原始值，然后线程 1 被阻塞了。之后线程 2 对变量进行自增操作，线程 2 也去读取变量 inc 的原始值，然后进行加 1 操作，并把 10 写入工作内存，最后写入主存。随后线程 1 接着进行加 1 操作，因为线程 1 在此前已经读取了 inc 的值为 9，所以不会再去主存读取最新的数值，线程 1 对 inc 进行加 1 操作后 inc 的值为 10，然后将 10 写入工作内存，最后写入主存。两个线程分别对 inc 进行了一次自增操作后，inc 的值只增加了 1，因此自增操作不是原子性操作，volatile也无法保证对变量的操作是原子性的。

注：跟可见并不矛盾，可见是要读取该值时，才去主存获取，但此时该值已经取到了，接下来要进行 +1 和写入操作。

## 2、正确使用 volatile 关键字

synchronized 关键字可防止多个线程同时执行一段代码，这会很影响程序执行效率。 而 volatile 关键字在某些情况下的性能要优于 synchronized。但是 volatile 是无法替代 synchronized 的，因为 volatile 无法保证操作的原子性。通常来说，使用 volatile 必须具备以下两个条件：

（1）对变量的写操作不会依赖于当前值。

（2）该变量没有包含在具有其他变量的不变式中。

第一个条件就是不能是自增、自减等操作，上文已经提到 volatile 不保证原子性。关于第二个条件，我们来举一个例子，它包含了一个不变式：下界总是小于或等于上界，代码如下所示：

```java
public class NumberRange {
 private volatile int lower, upper；
 public int getLower() {
 return lower； 
 }
 public int getUpper() { 
 return upper；
 }
 public void setLower(int value) {
 // 1、4 > 5 false
 if (value ＞ upper) 
 throw new IllegalArgumentException(...)；
 lower = value；
 }
 public void setUpper(int value) {
 // 2、3 < 0 false
 if (value ＜ lower) 
 throw new IllegalArgumentException(...)；
 upper = value；
 } }
```

​		这种方式将 lower 和 upper 字段定义为 volatile 类型不能够充分实现类的线程安全。如果当两个线程在同一时间使用不一致的值执行 setLower 和 setUpper 的话，则会使范围处于不一致的状态。例如，如果初始状态是(0, 5)，在同一时间内，线程 A 调用 setLower(4)并且线程 B 调用 setUpper(3)，虽然这两个操作交叉存入的值是不符合条件的，但是这两个线程都会通过用于保护不变式的检查，使得最后的范围值是(4, 3)。因为执行注释 1 、2 时，用于判断的值是 5、0 。这显然是不对的，因此使用 volatile 无法实现setLower 和 setUpper 操作的原子性。

​		使用 volatile 有很多种场景，这里介绍其中的两种。

（1）状态标志

```java
volatile boolean shutdownRequested；
...
public void shutdown()
{ 
shutdownRequested = true；
 }
public void doWork() { 
 while (!shutdownRequested) { 
 ...
 } }
```

如果在另一个线程中调用 shutdown 方法，就需要执行某种同步来确保正确实现shutdownRequested 变量的可见性。这里可以使用 volatile，状态标志 shutdownRequested 并不依赖于程序内的任何其他状态，并且还能简化代码。因此，此处适合使用 volatile。

（2）双重检查模式（DCL）

```java
public class Singleton { 
 private volatile static Singleton instance = null； 
 public static Singleton getInstance() { 
 if (instance == null) { 
 synchronized(this) { 
 if (instance == null) { 
 instance = new Singleton()； 
 } 
 }
 } 
 return instance； 
 } 
}
```

getInstance 方法中对 Singleton 进行了两次判空，第一次是为了避免不必要的同步，第二次是只有在 Singleton 等于 null 的情况下才创建实例。在这里用到了 volatile 关键字会或多或少地影响性能，但考虑到程序的正确性，牺牲这点性能还是值得的。DCL 的优点是资源利用率高，第一次执行 getInstance 方法时单例对象才被实例化，效率高。其缺点是第一次加载时反应稍慢一些，在高并发环境下也有一定的缺陷（虽然发生的概率很小）。 

> 关于 DCL 和 volatile 的关系，在 `DCL单例和volatile的使用.md` 中做详细说明。



# 相关问题

<font color='orange'>Q：volatile关键字干了什么？（字节跳动）</font>

volatile变量，用来确保将变量的更新操作通知到其他线程。volatile可以保证变量的可见性。

<font color='orange'>Q：什么叫指令重排？（字节跳动）</font>

在Java代码编译的时候，可能会对代码顺序进行优化，在单线程中这样的优化不会存在问题，但是在多线程中，就可能引起线程安全问题。volatile关键字可以保证变量不会重排序。

Q：volatile关键字干了什么？（字节跳动）

<font color='orange'>Q：volatile能否保证线程安全？</font>

①volatile变量是一种同步机制；②volatile能够确保可见性。

> 参考：https://www.cnblogs.com/laipimei/p/11857786.html

<font color='orange'>Q：volatile和synchronize有什么区别？（B站小米京东）</font>

volatile用来修饰一个变量，确保变量更新后立刻通知到其他线程，也就是保证可见性。它没有使用锁。

synchronized可以用来修饰方法、代码块，确保线程同步访问一段代码。

> 还需要深入。

<font color='orange'>Q：在DCL上的作用是什么？</font>

可以避免实例化对象时的指令重排。在DCL new Instance()的时候，在JVM中对应三个步骤，分配内存空间、实例化对象、将引用指向内存空间。jvm为了优化代码可能会进行重排序，导致前面的顺序改变——先将引用指向内存、再实例化对象，从而出现对象实例不为null，但其实初始化并没有完成的情况，从而导致代码错误。解决办法就是使用volatile将对变量的重排序禁止。

> 关于 DCL 和 volatile 的关系，在 `DCL单例和volatile的使用.md` 中做详细说明。



# 总结

​		与锁相比，volatile 变量是一种非常简单但同时又非常脆弱的同步机制，它在某些情况下将提供优于锁的性能和伸缩性。如果严格遵循 volatile 的使用条件，即变量真正独立于其他变量和自己以前的值，在某些情况下可以使用 volatile 代替 synchronized 来简化代码。然而，使用 volatile的代码往往比使用锁的代码更加容易出错。在第 （5、） 小节中介绍了可以使用 volatile 代替synchronized 的最常见的两种用例，在其他情况下我们最好还是使用 synchronized。

​		本文知识点概括：

- Java 内存模型；
- 原子性、可见性、有序性
- volatile 可见性、有序性
- volatile 不保证原子性
- volatile 的正确使用
  1. 状态标志
  2. DCL 对 volatile 有序性的运用



## 【精益求精】我还能做（补充）些什么？

1、



# 参考

1、《Android核心知识点笔记V2020.03.30》

2、《Android进阶之光 第一版》