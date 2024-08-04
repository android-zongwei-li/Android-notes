> version：2021/10/27
>
> review：2021/4/27--2021/10/17



目录

[TOC]

# 一、volatile优化DCL

​		我们第一次写的单例模式是下面这样的：

```java
public class Singleton {
    private static Singleton instance = null;
    public static Singleton getInstance() {
        if(null == instance) {                    // line A
            instance = new Singleton();        // line B
        }
        return instance;
    }
}
```

假设这样的场景：两个线程并发调用 Singleton.getInstance()，假设线程一先判断 instance是否为 null，即代码中 line A 进入到 line B 的位置。刚刚判断完毕后，JVM将 CPU 资源切换给线程二，由于线程一还没执行line B，所以instance仍然为空，因此线程二执行了new Singleton()操作。片刻之后，线程一被重新唤醒，它执行的仍然是new Singleton()操作，这样问题就来了，new出了两个instance，这还能叫单例吗？

​		紧接着，我们再做单例模式的第二次尝试：

```java
public class Singleton {
    private static Singleton instance = null;
    public synchronized static Singleton getInstance() {
        if(null == instance) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

比起第一段代码仅仅在方法中多了一个synchronized修饰符，现在可以保证不会出线程问题了。但是这里有个很大（至少耗时比例上很大）的性能问题。除了第一次调用时是执行了Singleton的构造函数之外，以后的每一次调用都是直接返回instance对象。返回对象这个操作耗时是很小的，绝大部分的耗时都用在synchronized修饰符的同步准备上，因此从性能上来说很不划算。

　　继续把代码改成下面这样：

```java
public class Singleton {
    private static Singleton instance = null;
    public static Singleton getInstance() {
        synchronized (Singleton.class) {
            if(null == instance) {
                instance = new Singleton();
            }
        }
        return instance;
    }
}
```

基本上，把synchronized移动到代码内部是没有什么意义的，每次调用getInstance()还是要进行同步。同步本身没有问题，但是我们只希望在第一次创建instance实例的时候进行同步，因此有了下面的写法——**双重锁定检查（DCL,Double Check Lock）**。

```java
public class Singleton {
    private static Singleton instance = null;
    public  static Singleton getInstance() {
        if(null == instance) {    // 线程二检测到instance不为空
            synchronized (Singleton.class) {
                if(null == instance) {
                    instance = new Singleton();    // 线程一被指令重排，先执行了赋值，但还没执行完构造函数（即未完成初始化）
                }
            }
        }
        return instance;    // 后面线程二执行时将引发：对象尚未初始化错误
    }
}
```

看样子已经达到了要求，除了第一次创建对象之外，其它的访问在第一个if中就返回了，因此不会走到同步块中，已经完美了吗？

　　如上代码段中的注释：假设线程一执行到instance = new Singleton()这句，这里看起来是一句话，但实际上其被编译后在JVM执行的对应汇编代码就发现，这句话被编译成8条汇编指令，大致做了三件事情：

　　1）给instance实例分配内存；

　　2）初始化instance的构造器；

　　3）将instance引用指向分配的内存空间（注意到这步时instance就非null了）

　　**如果指令按照顺序执行倒也无妨，但JVM为了优化指令，提高程序运行效率，允许指令重排序**。如此，在程序真正运行时以上指令执行顺序可能是这样的：

　　a）给instance实例分配内存；

　　b）将instance引用指向分配的内存空间，此时instance不为null；

　　c）初始化instance的构造器；

　　这时候，当线程一执行b）完毕，在执行c）之前，被切换到线程二上，这时候instance判断为非空，此时线程二直接来到return instance语句，拿走instance然后调用，接着就顺理成章地报错（**对象尚未初始化**）。

​		**具体来说就是synchronized虽然保证了线程的原子性（即synchronized块中的语句要么全部执行（并不保证一次全部执行完），要么一条也不执行），但单条语句编译后形成的指令并不是一个原子操作（即可能该条语句的部分指令未得到执行，就被切换到另一个线程了）**。

　　根据以上分析可知，**解决这个问题的方法是：禁止指令重排序优化，即使用volatile变量**。

```java
public class Singleton {
    private volatile static Singleton instance = null;
    public static Singleton getInstance() {
        if(null == instance) {
            synchronized (Singleton.class) {
                if(null == instance) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

​		将变量instance使用volatile修饰即可实现单例模式的线程安全。



# 相关问题

<font color='orange'>Q：volatile在DCL上的作用是什么？</font>

可以避免实例化对象时的指令重排。在DCL new Instance()的时候，在JVM中对应三个步骤，分配内存空间、实例化对象、将引用指向内存空间。jvm为了优化代码可能会进行重排序，导致前面的顺序改变——先将引用指向内存、再实例化对象，从而出现对象实例不为null，但其实初始化并没有完成的情况，从而导致代码错误。解决办法就是使用volatile将对变量的重排序禁止。



# 参考

1、[关于volatile解决DCL(双重检查)问题的看法](https://blog.csdn.net/ACreazyCoder/article/details/80982578)