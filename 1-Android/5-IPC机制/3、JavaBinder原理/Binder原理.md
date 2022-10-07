> version：2022/06/28
>
> review：



目录

[TOC]



# 关键词

Parcel。

# 一、前置知识

操作系统：进程间通信。Linux 进程的概念。

设计模式：代理模式。

Java基础：线程、线程池。



# 二、概述

## 1、 Binder是什么，有何作用？

Binder 是 Android 中用来实现进程间通信的一种机制。在整个 Android 系统中，Binder 被大量的使用。比如 Activity 的启动，系统服务的调用等，都用到了 Binder。

从Java层面来看，Binder 是实现了 IBinder 接口的一个类。通过 Binder，具体来说是通过aidl，应用可以很容易的实现跨进程通信，包括进程间对象的传递，方法的调用。

从Linux驱动层来看，它是/dev/binder目录下的一个binder驱动。

> TODO ：Linux角度，需要再学习。

## 2、为什么使用 Binder？

其实 Linux 中已经有了其他的一些 IPC 方式，比如管道、SystemV、Socket、共享内存、消息队列等。



2、 为什么要多进程？

在设备中，单个进程所分配的内存是有限的，根据机型不同，有36/48/64M等，当一个进程的内存不够使用时，就需要多个进程。

为了实现进程保活。

3、 什么时候需要用到进程间通信（使用场景）？

闹钟/打电话、WebView、图片加载库、推送、双进程守护

4、 进程间通信为什么要用到Binder机制？

不同进程之间存在进程隔离，不能直接通信。

内存划分：

![img](images/wpsA94F.tmp.jpg) 

比如有4G内存，用户空间分配3G，内核空间分配1G。用户空间之间是不能直接进行通信的，需要通过内核空间。（其实用户空间也不能对内核空间进行直接操作，必须通过“系统调用”进行通信，这样才能保证系统的安全性。）

5、 Android增加Binder的原因是什么？

Android特有IPC机制：Binder。

![img](images/wpsA950.tmp.jpg) 

性能方面 

在移动设备上（性能受限制的设备，比如要省电），广泛地使用跨进程通信对通信 

机制的性能有严格的要求，Binder相对于传统的Socket方式，更加高效。**Binder**数 

据拷贝只需要一次，而管道、消息队列、**Socket**都需要**2**次，共享内存方式一次内 

存拷贝都不需要，但实现方式又比较复杂。 

安全方面 

传统的进程通信方式对于通信双方的身份并没有做出严格的验证，比如Socket通信 

的IP地址是客户端手动填入，很容易进行伪造。然而，Binder机制从协议本身就支 

持对通信双方做身份校检，从而大大提升了安全性。

传统IPC传输数据：

![img](images/wpsA951.tmp.jpg) 

Binder传输数据：

原理：MMAP(memory map)内存映射。

![img](images/wpsA952.tmp.jpg) 



# 二、Binder原理

Binder通信采用C/S架构，从组件视角来说，包含Client、Server、ServiceManager 以及 Binder驱动，其中ServiceManager用于管理系统中的各种服务。架构图如下所示：

![image-20220628171553798](images/image-20220628171553798.png)

**Binder**通信的四个角色 

**Client**进程：使用服务的进程。 

**Server**进程：提供服务的进程。 

**ServiceManager**进程：ServiceManager的作用是将字符形式的Binder名字转化成 

Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder 

实体的引用。 

**Binder**驱动：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递， 

Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。 



**Binder**运行机制 

图中Client/Server/ServiceManage之间的相互通信都是基于Binder机制。既然基于 

Binder机制通信，那么同样也是C/S架构，则图中的3大步骤都有相应的Client端与 

Server端。 

注册服务**(addService)**：Server进程要先注册Service到ServiceManager。该过 

程：Server是客户端，ServiceManager是服务端。 

获取服务**(getService)**：Client进程使用某个Service前，须先向ServiceManager中 

获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。 

使用服务：Client根据得到的Service信息建立与Service所在的Server进程通信的通 

路，然后就可以直接与Service交互。该过程：Client是客户端，Server是服务端。

图中的Client，Server，Service Manager之间交互都是虚线表示，是由于它们彼此 

之间不是直接交互的，而是都通过与Binder驱动进行交互的，从而实现IPC通信 

（Interprocess Communication）方式。其中Binder驱动位于内核空间，Client， 

Server，Service Manager位于用户空间。Binder驱动和Service Manager可以看做 

是Android平台的基础架构，而Client和Server是Android的应用层，开发人员只需自 

定义实现Client、Server端，借助Android的基本平台架构便可以直接进行IPC通 

信。



**Binder**运行的实例解释 

首先我们看看我们的程序跨进程调用系统服务的简单示例，实现浮动窗口部分代 

码

```java
//获取WindowManager服务引用 WindowManager wm = (WindowManager) getSystemService(getApplicati on().WINDOW_SERVICE); 
//布局参数layoutParams相关设置略... 
View view = LayoutInflater.from(getApplication()).inflate(R.layo ut.float_layout, null); 
//添加view 
wm.addView(view, layoutParams);
```

注册服务**(addService)**： 

在Android开机启动过程中，Android会初始化系统的各种 Service，并将这些Service向ServiceManager注册（即让ServiceManager管理）。 这一步是系统完成的。 

获取服务**(getService)**： 

客户端想要得到具体的Service直接向ServiceManager要即可。客户端首先向ServiceManager查询得到具体的Service引用，通常是 Service 引用的代理对象，对数据进行一些处理操作。即第2行代码中，得到的wm是 WindowManager对象的引用。 

使用服务： 

通过这个引用向具体的服务端发送请求，服务端执行完成后就返回。即第6行调用WindowManager的addView函数，将触发远程调用，调用的是运行在 

systemServer进程中的WindowManager的addView函数。 

使用服务的具体执行过程

![image-20220628172402494](images/image-20220628172402494.png)

1. Client通过获得一个Server的代理接口，对Server进行调用。 

2. 代理接口中定义的方法与Server中定义的方法是一一对应的。 

3. Client调用某个代理接口中的方法时，代理接口的方法会将Client传递的参数打 包成Parcel对象。 

4. 代理接口将Parcel发送给内核中的Binder Driver。 

5. Server会读取Binder Driver中的请求数据，如果是发送给自己的，解包Parcel 对象，处理并将结果返回。 

6. 整个的调用过程是一个同步过程，在Server处理的时候，Client会Block住。因 此**Client**调用过程不应在主线程。 



三、其他知识点记录

一个进程只有一个主线程。

 

# 相关问题

**1、Android 中进程和线程的关系？ 区别？**

线程是 CPU 调度的**最小单元**，同时线程是一种**有限**的系统资源进程一般指一个执行单元，在 PC 和移动设备上一个程序或则一个应用一般来说，一个 App 程序**至少有一个**进程，一个进程**至少有一个**线程（包含与被包含的关系）， 通俗来讲就是，在App 这个工厂里面有一个进程，线程就是里面的生产线，但主线程（主生产线）只有一条，而子线程（副生产线）可以有多个

进程有自己独立的地址空间，而进程中的线程共享此地址空间，都可以**并发**执行

**2、如何开启多进程 ？ 应用是否可以开启 N 个进程 ？** 

在 AndroidMenifest 中给四大组件指定属性 android:process 开启多进程模式

在内存允许的条件下可以开启 N 个进程

**3、为何需要 IPC？多进程通信可能会出现的问题？**

所有运行在不同进程的四大组件（Activity、Service、Receiver、ContentProvider）共享数据都会失败，这是由于 Android 为每个应用分配了独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这会导致在不同的虚拟机中访问同一个类的对象会产生多份副本。比如常用例子（**通过开启多进程获取更大内存空间、两个或则多个应用之间共享数据、微信全家桶**） 

一般来说，使用多进程通信会造成如下几方面的问题

**静态成员和单例模式完全失效**：独立的虚拟机造成

**线程同步机制完全实效**：独立的虚拟机造成

**SharedPreferences 的可靠性下降**：这是因为 Sp 不支持两个进程并发进行读写，有一定几率导致数据丢失

**Application 会多次创建**：Android 系统在创建新的进程会分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，自然也会创建新的Application

**4、Android 中 IPC 方式、各种方式优缺点，为什么选择 Binder？** 

![image-20221007163638713](images/Binder原理/image-20221007163638713.png)

与 Linux 上传统的 IPC 机制，比如 System V，Socket 相比，Binder 好在哪呢？

**传输效率高、可操作性强**：传输效率主要影响因素是内存拷贝的次数，拷贝次数越少，传输速率越高。从 Android

进程架构角度分析：对于消息队列、Socket 和管道来说，数据先从发送方的缓存区拷贝到内核开辟的缓存区中，再从内核缓存区拷贝到接收方的缓存区，一共两次拷贝，如图：

![image-20221007163721243](images/Binder原理/image-20221007163721243.png)

而对于 Binder 来说，数据从发送方的缓存区拷贝到内核的缓存区，而接收方的缓存区与内核的缓存区是映射到同一块物理地址的，节省了一次数据拷贝的过程，如图：

![image-20221007163831944](images/Binder原理/image-20221007163831944.png)

由于共享内存操作复杂，综合来看，Binder 的传输效率是最好的。

**实现 C/S 架构方便**：Linux 的众 IPC 方式除了 Socket 以外都不是基于 C/S 架构，而 Socket 主要用于网络间的通信且传输效率较低。Binder 基于 C/S 架构 ，Server 端与Client 端相对独立，稳定性较好。

**安全性高**：传统 Linux IPC 的接收方无法获得对方进程可靠的 UID/PID，从而无法鉴别对方身份；而 Binder 机制为每个进程分配了 UID/PID 且在 Binder 通信时会根据UID/PID 进行有效性检测。

**5、Binder 机制的作用和原理？**

Linux 系统将一个进程分为**用户空间**和**内核空间**。对于进程之间来说，用户空间的数据不可共享，内核空间的数据可共享，为了保证安全性和独立性，一个进程不能直接操作或者访问另一个进程，即 Android 的进程是相互独立、隔离的，这就需要跨进程之间的数据通信方式

![image-20221007164053183](images/Binder原理/image-20221007164053183.png)

一次完整的 Binder IPC 通信过程通常是这样：

o 首先 Binder 驱动在内核空间创建一个数据接收缓存区；

o 接着在内核空间开辟一块内核缓存区，建立内核缓存区和内核中数据接收缓存区之间的映射关系，以及内核中数据接收缓存区和接收进程用户空间地址的映射关系；

o 发送方进程通过系统调用 copyfromuser() 将数据 copy 到内核中的内核缓存区，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

![image-20221007164317933](images/Binder原理/image-20221007164317933.png)

**6、Binder 框架中 ServiceManager 的作用？**

o **Binder 框架** 是基于 C/S 架构的。由一系列的组件组成，包括 Client、Server、ServiceManager、Binder 驱动，其中Client、Server、ServiceManager 运行在用户空间，Binder 驱动运行在内核空间

![image-20221007164450733](images/Binder原理/image-20221007164450733.png)

**Server&Client**：服务器&客户端。在 Binder 驱动和ServiceManager 提供的基础设施上，进行 Client、Server 之间的通信。

**ServiceManager**（如同 DNS 域名服务器）服务的管理者，将 Binder 名字转换为 Client 中对该 Binder的引用，使得 Client 可以通过 Binder 名字获得Server 中 Binder 实体的引用。

**Binder 驱动**（如同路由器）：负责进程之间 binder通信的建立，传递，计数管理以及数据的传递交互等底层支持。

![image-20221007164616283](images/Binder原理/image-20221007164616283.png)



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 脑图



# 参考

