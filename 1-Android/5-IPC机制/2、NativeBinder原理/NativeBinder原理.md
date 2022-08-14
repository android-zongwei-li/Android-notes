> version：2021/11/
>
> review：



目录

[TOC]



根据Android系统的分层，Binder机制“从上往下”可以分为 Java Binder、Native Binder、Kernel Binder。上层依赖于下层，本文Kernel Binder基于goldfish3.4。

# 一、前置知识

Linux和Binder的IPC通信原理、使用Binder的原因、学习Binder的原因。

## 1、Linux和Binder的IPC通信原理

进程通信简单模型如下：

![进程间通信简单模型](images/%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1%E7%AE%80%E5%8D%95%E6%A8%A1%E5%9E%8B.jpg)

![进程间通信](images/%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1-%E5%9B%BE2.jpg)

*  在下文中，左边的用户空间称为A进程，右边的称为B进程。

下面了解几个概念：

### 1.1、内核空间和用户空间

内核空间是 Linux 内核的运行空间，用户空间是用户程序的运行空间。为了保证内核空间的安全，操作系统从逻辑上将虚拟内存划分为内核空间和用户空间，它们是相互隔离的，以避免用户进程直接操作内核，内核也不会受用户空间crash的影响。内核空间的数据是可以共享的，而用户空间则不行。

### 1.2、进程隔离

进程隔离是指一个进程不能直接操作或者访问另一个进程，也就是进程A不能直接访问进程B的数据。

### 1.3、系统调用

用户空间借助系统调用来访问内核空间。系统调用是用户空间访问内核空间的唯一方式，这保证了所有资源的访问都是在内核的控制下进行的，避免了用户程序对系统资源的越权访问，提升了系统的安全性和稳定性。

进程A和进程B的用户空间可以通过如下系统函数和内核空间进行交互。

- copy_from_user：将用户空间的数据复制到内核空间。
- copy_to_user：将内核空间的数据复制到用户空间。

### 1.4、内存映射（注：还不是很理解）

由于应用程序不能直接操作设备硬件地址，所以操作系统提供了一种机制：内存映射（Memory Map），将设备地址映射到进程虚拟内存区。

### 1.5、Linux的IPC通信原理



### 1.6、Binder的通信原理

-



## 2、使用Binder的原因



## 3、学习Binder的原因





# 二、

# 相关问题

<font color='orange'>Q：Binder原理</font>



<font color='orange'>Q：Android中Binder机制。</font>



<font color='orange'>Q：Binder泄漏</font>



<font color='orange'>Q：Binder比起其他跨进程的通信方式好在哪？Binder的优势</font>



<font color='orange'>Q：Binder具体的实现原理，数据拷贝次数：代理模式&协议</font>



<font color='orange'>Q：简单讲讲Binder驱动吧？</font>



<font color='orange'>Q：为什么要用Binder</font>



<font color='orange'>Q：看过Binder驱动的源码，说说他的内存映射过程，说说客户端等待服务端处理返回的流程，如果要跨进程传递大内存数据你具体会怎么做？简单写一写吧。</font>



<font color='orange'>Q：Binder线程池的工作过程是什么样</font>

<font color='orange'>Q：Android Binder之应用层总结与分析</font>

<font color='orange'>Q：Binder怎么实现进程间通信的？mmap的模型？</font>

<font color='orange'>Q：Binder两个特点</font>

<font color='orange'>Q：进程间通信的方式，安卓中有哪些方式，为什么是基于Binder的，不用传统的操作系统进程间通信方式呢</font>

<font color='orange'>Q：Binder机制说一下。如何实现两个进程同时访问一个服务时线程同步问题。</font>

<font color='orange'>Q：Binder的通讯过程：具体server和client的通讯，为什么是一次拷贝</font>

<font color='orange'>Q：Binder通讯的流程</font>



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 脑图



# 参考

1、《Android进阶指北》