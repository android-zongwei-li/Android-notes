> 知识路径：Android > Java > 集合
>
> version：2021/4/2
>
> review：2020/12/27---2021/4/2

## 一、数组、顺序表、链表简单介绍

1.1、首先得了解一下
[数组的优缺点](https://blog.csdn.net/qq_29224201/article/details/103130896)

1.2、为了实现数组的动态添加与删除，出现了顺序表

![顺序表结构](https://upload-images.jianshu.io/upload_images/9000209-cfb9a3edd3bb7f04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.3、为了避免数组添加、删除效率低等缺点，出现了链表

![链表结构](https://upload-images.jianshu.io/upload_images/9000209-64e2bbf090830ac7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.4、数组、顺序表、链表的发展

![数组、顺序表、链表的发展](https://upload-images.jianshu.io/upload_images/9000209-219863b0f4c068f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.5、顺序表与链表的对比

![顺序表与链表优缺点](https://upload-images.jianshu.io/upload_images/9000209-2c5b8467d60e09e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.6、Hash表的出现

![Hash表](https://upload-images.jianshu.io/upload_images/9000209-735f00b0d2869098.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 二、Hash表介绍

### 2.1、Hash结构/原理图简介

Java 7：

![Hash结构/原理图](https://upload-images.jianshu.io/upload_images/9000209-7867defb9ad69768.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Java 8：

![image-20210402103457516](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210402103457516.png)

### 2.2、Hash相关问题汇总

![Hash相关问题汇总](https://upload-images.jianshu.io/upload_images/9000209-f4839c58ef93780e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.3、Hash问题解析

##### 2.3.1、Hash表添加（put）元素的过程

![Hash表添加元素示意图](https://upload-images.jianshu.io/upload_images/9000209-7591f3d347617242.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在进行添加元素时，首先会对key进行hash(k)运算，得到一个int类型的hash值。然后再使用这个值求得一个index，这个index就是数组的下标。最后把元素添加到index下标对应的链表中，完成添加。

index计算方法：
hash & (n - 1) 等同于 ：
hash % n（n 为 数组.length）

##### 2.3.2、Hash运算（原理）示意图

![jdk1.7-Hash运算过程1](https://upload-images.jianshu.io/upload_images/9000209-6e6a1e2130b79ffa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![jdk1.7-Hash运算过程2](https://upload-images.jianshu.io/upload_images/9000209-074eb3401e6a8cc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

异或：相同为0。

不同的Java版本有不同的hash运算方法，最终的结果还取决于Key对象的hashCode()方法。

##### 2.3.3、数组与链表如何组织工作？

数组中的元素为Entry，Entry是一个链表的具体实现，通过next连接。

##### 2.3.4、int hash是什么？有什么用？

int hash = hash(key);
用于求出index下标：
index = hash & (n - 1);

### 2.4、HashMap碰撞与链表

##### 2.4.1 Hash碰撞

不同的对象算出来的index是相同的。

##### 2.4.2 jdk1.7-Hash碰撞解决

![Hash碰撞解决jdk1.7](https://upload-images.jianshu.io/upload_images/9000209-b974ee9b57b617da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2.4.3 jdk1.8-Hash碰撞解决

![Hash碰撞解决jdk1.8](https://upload-images.jianshu.io/upload_images/9000209-17bc864b328c94db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

节点查找优先级由O(n)提高到O(log(n))。

> **时间复杂度理解还不透彻。还不懂红黑树，学完数据结构红黑树以后，再来深入理解这块内容。**

## 三、HashMap相关问题

#### 3.1 如果 HashMap 的大小超过了负载因子（load factor）定义的容量，怎么办？

默认的负载因子大小为0.75，也就是说，当一个map填满了75%的数组index后，和其他集合类（如ArrayList等）一样，将会创建原来HashMap大小的两倍的index数组（也叫bucket数组），来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫做rehasing，因为它调用了hash()方法找到新的bucket（index）位置。

#### 3.2 为什么String、Integer这样的wrapper类适合作为键？

因为String是不可变的（final），而且已经重写了equals()和hashCode()方法。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。不可变性还有其他的优点，比如线程安全。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的话，那么请这么做吧。因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小一些，这样就能提高HashMap的性能。

