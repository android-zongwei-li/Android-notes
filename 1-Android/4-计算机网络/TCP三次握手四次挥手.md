

# 前言

- 计算机网络基础中，`TCP`协议建立连接 & 释放连接时的三次握手、四次挥手十分重要
- 今天carson将图文解析`TCP`链接的三次握手 & 四次挥手，包学包会包易懂！

------

# 1. TCP建立连接：三次握手

### 1.1 示意图

![img](https:////upload-images.jianshu.io/upload_images/944365-895493e20637d2b0.png?imageMogr2/auto-orient/strip|imageView2/2/w/880/format/webp)

### 1.2 流程解析

![img](https:////upload-images.jianshu.io/upload_images/944365-d148731fa16316be.png?imageMogr2/auto-orient/strip|imageView2/2/w/1020/format/webp)

**成功进行TCP的三次握手后，就建立起一条TCP连接，即可传送应用层数据**。需要注意的是：

1. 因 `TCP`提供的是全双工通信，故通信双方的应用进程在任何时候都能发送数据
2. 三次握手期间，任何1次未收到对面的回复，则都会重发

### 1.3 特别说明：为什么TCP建立连接需三次握手？

- 结论
   防止服务器端因接收了**早已失效的连接请求报文**，从而一直等待客户端请求，最终导致**形成死锁、浪费资源**
- 具体描述

![img](https:////upload-images.jianshu.io/upload_images/944365-1551b53e24060636.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/857/format/webp)

具体描述

> SYN洪泛攻击：
>
> - 从上可看出：服务端的TCP资源分配时刻 = 完成第二次握手时；而客户端的TCP资源分配时刻 = 完成第三次握手时
> - 这就使得服务器易于受到`SYN`洪泛攻击，即同时多个客户端发起连接请求，从而需进行多个请求的TCP连接资源分配

------

# 2. TCP释放连接：四次挥手

### 2.1 示意图

![img](https:////upload-images.jianshu.io/upload_images/944365-6162a7db50ebb9d3.png?imageMogr2/auto-orient/strip|imageView2/2/w/950/format/webp)

### 2.2 流程解析

![img](https:////upload-images.jianshu.io/upload_images/944365-91b079843a9e8235.png?imageMogr2/auto-orient/strip|imageView2/2/w/1020/format/webp)

示意图

### 2.3 特别说明

#### 说明1：为什么TCP释放连接需四次挥手？

- 结论
   为了保证通信双方都能通知对方 需释放 & 断开连接

> 即释放连接后，都无法接收 / 发送消息给对方

- 具体描述

![img](https:////upload-images.jianshu.io/upload_images/944365-345dbc590bbcb19d.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/816/format/webp)

示意图

#### （延伸疑问）说明2：为什么客户端关闭连接前要等待2MSL时间？

> 1. 即 `TIME - WAIT` 状态的作用是什么；
> 2. `MSL` = 最长报文段寿命（`Maximum Segment Lifetime`）

- 原因1：为了保证客户端发送的最后1个连接释放确认报文 能到达服务器，从而使得服务器能正常释放连接

![img](https:////upload-images.jianshu.io/upload_images/944365-1cfff0282bdac472.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/880/format/webp)

示意图

- 原因2：防止 上文提到的早已失效的连接请求报文 出现在本连接中
   客户端发送了最后1个连接释放请求确认报文后，再经过2`MSL`时间，则可使本连接持续时间内所产生的所有报文段都从网络中消失。

> 即 在下1个新的连接中就不会出现早已失效的连接请求报文

# 3. 总结

- 本文全面讲解了 计算机网络中`TCP`协议最重要的三次握手 & 四次挥手



# 参考

[计算机网络：图文解析TCP的三次握手、四次挥手](https://www.jianshu.com/p/9253c34215a7)