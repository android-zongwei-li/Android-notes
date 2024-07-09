# 前言

- `HTTP`网络通信协议在任何的开发工作中都非常重要
- 今天，我将献上一份`HTTP`的说明指南，希望你们会喜欢

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-38bddd4279238551.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

# 储备知识

讲解`HTTP`协议前，先了解一些基础的计算机网络相关知识

### 1.1 计算机网络体系结构

- 定义
   计算机网络的各层 + 其协议的集合
- 作用
   定义该计算机网络的所能完成的功能
- 结构介绍
   计算机网络体系结构分为3种：`OSI`体系结构、`TCP` / `IP`体系结构、五层体系结构

> - `OSI`体系结构：概念清楚 & 理念完整，但复杂 & 不实用
> - `TCP` / `IP`体系结构：含了一系列构成互联网基础的网络协议，是`Internet`的核心协议 & 被广泛应用于局域网 和 广域网
> - 五层体系结构：融合了`OSI` 与 `TCP` / `IP`的体系结构，目的是为了学习 & 讲解计算机原理

![img](https:////upload-images.jianshu.io/upload_images/944365-b3deda7c3f3b92c6.png?imageMogr2/auto-orient/strip|imageView2/2/w/620/format/webp)

示意图

- ```
  TCP
  ```

   / 

  ```
  IP
  ```

  的体系结构详细介绍

  由于 

  ```
  TCP
  ```

   / 

  ```
  IP
  ```

  体系结构较为广泛，故主要讲解

  ![img](https:////upload-images.jianshu.io/upload_images/944365-73d7d56b1f54d945.png?imageMogr2/auto-orient/strip|imageView2/2/w/780/format/webp)


### 1.2 HTTP 协议通信的基础模型

- `HTTP`协议传输信息的基础：`TCP/IP`协议模型

  ![img](https:////upload-images.jianshu.io/upload_images/944365-492f1cf3981cb078.png?imageMogr2/auto-orient/strip|imageView2/2/w/620/format/webp)

  示意图

  

- `HTTP`协议 属于 最高层的应用层

# 2. 简介

下面，将简单介绍一下 `HTTP`

![img](https:////upload-images.jianshu.io/upload_images/944365-a6c6e24b7bb7bfc7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 3. 工作方式

- `HTTP`协议采用 **请求 / 响应** 的工作方式
- 具体工作流程如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-28e4020dd64411b4.png?imageMogr2/auto-orient/strip|imageView2/2/w/280/format/webp)

示意图

------

# 4. HTTP报文详解

- `HTTP`在 应用层 交互数据的方式 = 报文
- `HTTP`的报文分为：请求报文 & 响应报文

> 分别用于 发送请求 & 响应请求时

- 下面，将详细介绍这2种报文

### 4.1 请求报文

### 4.1.1 报文结构

- `HTTP`的请求报文由 **请求行、请求头 & 请求体** 组成，如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-76f625b54c1039be.png?imageMogr2/auto-orient/strip|imageView2/2/w/580/format/webp)

示意图

- 下面，将详细介绍每个组成部分

### 4.1.2 结构详细介绍

##### 组成1：请求行

- 作用
   声明 请求方法 、主机域名、资源路径 & 协议版本
- 结构
   请求行的组成 = 请求方法 + 请求路径 + 协议版本

> 注：空格不能省

![img](https:////upload-images.jianshu.io/upload_images/944365-ee23b153f6d654c6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

请求行的组成

- 组成介绍

![img](https:////upload-images.jianshu.io/upload_images/944365-332ccda4eb8625bf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

> 此处特意说明GET、PSOT方法的区别：

![img](https:////upload-images.jianshu.io/upload_images/944365-acce2f2323fd28a6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1100/format/webp)

示意图

- 示例
   设：请求报文采用`GET`方法、 `URL`地址 = [http://www.tsinghua.edu.cn/chn/yxsz/index.htm](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.tsinghua.edu.cn%2Fchn%2Fyxsz%2Findex.htm)；、`HTTP1.1`版本

则 请求行是：`GET /chn/yxsz/index.htm HTTP/1.1`

##### 组成2：请求头

- 作用：声明 客户端、服务器 / 报文的部分信息
- 使用方式：采用**”header（字段名）：value（值）“**的方式
- 常用请求头
   **1. 请求和响应报文的通用Header**

![img](https:////upload-images.jianshu.io/upload_images/944365-254eb5a7db3d3fe5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

请求和响应报文的通用Header

**2. 常见请求Header**

![img](https:////upload-images.jianshu.io/upload_images/944365-22f107afd0839c1a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

常见请求Header

- 举例：
   (URL地址：[http://www.tsinghua.edu.cn/chn/yxsz/index.htm](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.tsinghua.edu.cn%2Fchn%2Fyxsz%2Findex.htm)）
   Host：[www.tsinghua.edu.cn](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.tsinghua.edu.cn)   (表示主机域名）
   User - Agent：Mozilla/5.0     (表示用户代理是使用Netscape浏览器）

##### 组成3：请求体

- 作用：存放 需发送给服务器的数据信息

> 可选部分，如 `GET请求`就无请求数据

- 使用方式：共3种

![img](https:////upload-images.jianshu.io/upload_images/944365-6a361cc6960eb113.png?imageMogr2/auto-orient/strip|imageView2/2/w/860/format/webp)

示意图

**至此，关于请求报文的请求行、请求头、请求体 均讲解完毕。**

### 4.1.3 总结

- 关于 请求报文的总结如下

![img](https:////upload-images.jianshu.io/upload_images/944365-63e390481e92aebe.png?imageMogr2/auto-orient/strip|imageView2/2/w/873/format/webp)

示意图

- 请求报文示例

![img](https:////upload-images.jianshu.io/upload_images/944365-806653dd8c7df0e7.png?imageMogr2/auto-orient/strip|imageView2/2/w/480/format/webp)

示意图

### 4.2 HTTP响应报文

### 4.2.1 报文结构

- `HTTP`的响应报文包括：状态行、响应头 & 响应体

![img](https:////upload-images.jianshu.io/upload_images/944365-e74ef9116a1b5df8.png?imageMogr2/auto-orient/strip|imageView2/2/w/580/format/webp)

示意图

- 其中，响应头、响应体 与请求报文的请求头、请求体类似
- 这2种报文最大的不同在于 状态行 & 请求行

下面，将详细介绍每个组成部分

### 4.2.2 结构详细介绍

### 组成1：状态行

- 作用
   声明 协议版本，状态码，状态码描述
- 组成
   状态行有协议版本、状态码 &状态信息组成

> 其中，空格不能省

![img](https:////upload-images.jianshu.io/upload_images/944365-834e3a1b316f265c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

状态行组成

- 具体介绍

  ![img](https:////upload-images.jianshu.io/upload_images/944365-bd27e7365b0b2855.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  示意图

- 状态行 示例
   `HTTP/1.1 202 Accepted`(接受)、`HTTP/1.1 404 Not Found`(找不到)

### 组成2：响应头

- 作用：声明客户端、服务器 / 报文的部分信息
- 使用方式：采用**”header（字段名）：value（值）“**的方式
- 常用请求头
   **1. 请求和响应报文的通用Header**

![img](https:////upload-images.jianshu.io/upload_images/944365-254eb5a7db3d3fe5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

请求和响应报文的通用Header

**2. 常见响应Header**

![img](https:////upload-images.jianshu.io/upload_images/944365-a9f0a212c4ea72b9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

常见响应Header

### 组成3：响应体

- 作用：存放需返回给客户端的数据信息
- 使用方式：和请求体是一致的，同样分为：任意类型的数据交换格式、键值对形式和分部分形式

![img](https:////upload-images.jianshu.io/upload_images/944365-9e829a9edda9ceb0.png?imageMogr2/auto-orient/strip|imageView2/2/w/860/format/webp)

示意图

### 4.2.3 响应报文 总结

![img](https:////upload-images.jianshu.io/upload_images/944365-cc24a9b8effcbd42.png?imageMogr2/auto-orient/strip|imageView2/2/w/832/format/webp)

示意图

### 4.3 总结

下面，简单总结两种报文结构

![img](https:////upload-images.jianshu.io/upload_images/944365-57aec599cfef3f0f.png?imageMogr2/auto-orient/strip|imageView2/2/w/840/format/webp)

示意图

------

# 5. 额外知识

下面将讲解一些关于`HTTP`的额外知识：

- `HTTP1.1` 与 `HTTP1.0` 的区别
- `HTTP` 与 `HTTPS`的区别
- `HTTP` 处理长连接的方式

### 5.1 HTTP1.1 与 HTTP1.0的区别

`Http1.1` 比 `Http1.0` 多了以下优点：

- 引入持久连接，即 在同一个`TCP`的连接中可传送多个`HTTP`请求 & 响应
- 多个请求 & 响应可同时进行、可重叠
- 引入更加多的请求头 & 响应头

> 如 与身份认证、状态管理 & `Cache`缓存等机制相关的、`HTTP1.0`无`host`字段

### 5.2 HTTP 与HTTPS的区别

![img](https:////upload-images.jianshu.io/upload_images/944365-820f955afd57185f.png?imageMogr2/auto-orient/strip|imageView2/2/w/712/format/webp)



### 5.3 HTTP处理长连接的方式

![img](https:////upload-images.jianshu.io/upload_images/944365-1ae8c95acbb1b364.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/899/format/webp)



# 参考

[计算机网络：这是一份全面& 详细 HTTP知识讲解](https://www.jianshu.com/p/a6d086a3997d)