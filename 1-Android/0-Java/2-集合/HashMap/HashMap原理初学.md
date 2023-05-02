> version：2021/4/2
>
> review：2020/12/27---2021/4/2



HashMap用到的数据结构

数组、单向链表、双向链表、红黑树



### 构造方法以及属性解析



```java
// 初始值为null
transient Node<K,V>[] table;

static class Node<K,V> implements Map.Entry<K,V> {
	// 根据 Key 计算出的 hash 值
	final int hash;
	final K key;
	V value;
	Node<K,V> next;
        
	// 方法先省略...
}

transient Set<Map.Entry<K,V>> entrySet;

// 表示 HashMap 中存了多少个键值对
transient int size;

// 表示 HashMap修改过几次，put、remove是在修改，get不是修改，
// 这个跟异常有关ConcurrentModificationException
transient int modCount;

// 阈值，默认为0
int threshold;

// 默认加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 加载因子，跟扩容有关
final float loadFactor;

public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

// initialCapacity 指的是初始容量，容量是指 table 数组的容量。
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```



### put 方法解析

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 根据 Key 对象算出一个key的 hash 值
static final int hash(Object key) {
    int h;
    // 先计算出 key 的原始 hashCode，再进行扰动（右移、异或运算）
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

Q：为什么不直接使用 key 的 hashCode() 方法返回的值作为 hash值。



> 异或运算：相同为0，不同为1。
>
> a = 3;b = 5;
>
> a^b = 0011 ^ 0101 = 0110



## 一、数组、顺序表、链表简单介绍

1.1、首先得了解一下
[数组的优缺点](https://blog.csdn.net/qq_29224201/article/details/103130896)

1.2、为了实现数组的动态添加与删除，出现了顺序表

![顺序表结构](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/1.webp)

1.3、为了避免数组添加、删除效率低等缺点，出现了链表

![链表结构](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p2.webp)

1.4、数组、顺序表、链表的发展

![数组、顺序表、链表的发展](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p3.webp)

1.5、顺序表与链表的对比

![顺序表与链表优缺点](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p4.webp)

1.6、Hash表的出现

![Hash表](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p5.webp)

## 二、Hash表介绍

### 2.1、Hash结构/原理图简介

Java 7：

![Hash结构/原理图](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p6.webp)

Java 8：



### 2.2、Hash相关问题汇总

![Hash相关问题汇总](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p7.webp)

### 2.3、Hash问题解析

##### 2.3.1、Hash表添加（put）元素的过程

![Hash表添加元素示意图](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p8.webp)

在进行添加元素时，首先会对key进行hash(k)运算，得到一个int类型的hash值。然后再使用这个值求得一个index，这个index就是数组的下标。最后把元素添加到index下标对应的链表中，完成添加。

index计算方法：
hash & (n - 1) 等同于 ：
hash % n（n 为 数组.length）

##### 2.3.2、Hash运算（原理）示意图

![jdk1.7-Hash运算过程1](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p9.webp)

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

![Hash碰撞解决jdk1.7](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p10.webp)

##### 2.4.3 jdk1.8-Hash碰撞解决

![Hash碰撞解决jdk1.8](images/HashMap%E5%8E%9F%E7%90%86%E5%88%9D%E5%AD%A6/p11.webp)

节点查找优先级由O(n)提高到O(log(n))。

> **时间复杂度理解还不透彻。还不懂红黑树，学完数据结构红黑树以后，再来深入理解这块内容。**

## 三、自测问题

#### 3.1 如果 HashMap 的大小超过了负载因子（load factor）定义的容量，怎么办？

默认的负载因子大小为0.75，也就是说，当一个map填满了75%的数组index后，和其他集合类（如ArrayList等）一样，将会创建原来HashMap大小的两倍的index数组（也叫bucket数组），来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫做rehasing，因为它调用了hash()方法找到新的bucket（index）位置。

#### 3.2 为什么String、Integer这样的wrapper类适合作为键？

因为String是不可变的（final），而且已经重写了equals()和hashCode()方法。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。不可变性还有其他的优点，比如线程安全。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的话，那么请这么做吧。因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小一些，这样就能提高HashMap的性能。

### 原理

1、Hashmap的原理

### 扩容

### 红黑树

### 并发

### 对比



|                                                           |
| --------------------------------------------------------- |
| hash算法，hash怎么散列的                                  |
| 遍历Hashmap的原理?                                        |
| 如果Hashmap，key不一样,但是hashcode一样会怎么样?          |
| Hashmap的哈希散列实现？                                   |
| HashMap的底层原理是什么？线程安全么？                     |
| HashMap中put是如何实现的？                                |
| 什么是哈希碰撞？怎么解决?                                 |
| Hashmap扩容的条件，扩容时到底干了什么（位运算过程）       |
| HashMap中什么时候需要进行扩容，扩容resize()是如何实现的？ |
| Hashmap红黑树的时间效率为什么是logn，怎么算出来的？       |
| Hashmap为什么要链表转红黑树，为什么不一开始就用红黑树     |
| Hashmap在1.7和1.8中的区别是啥？链表环是怎么形成的？       |
| Hashmap和hashset区别                                      |
| HashSet和Hashmap的关系                                    |
| treemap,Hashmap应用场景                                   |
| LinkedHashmap跟Hashmap的差别是什么？ 底层原理是什么？     |
| HashMap和HashTable的区别（小米）                          |
| hashmapconcurrenthashmap原理（美团）                      |
| arraylist和hashmap的区别，为什么取数快？（字节跳动）      |
| SparseArray和Hashmap区别                                  |
| Hashmap是线程安全的吗？Hashmap怎么实现线程安全？          |
| Hashmap高并发场景会怎样                                   |
| 怎样设计实现一个高效的线程安全的Hashmap。                 |
| LRU实现（146）参考LinkedHashMap实现的3个方法              |



## 参考

1、[被面试官狂问HashMap：万般无奈只好肝了这套HashMap底层原理源码教程，受益匪浅！](https://www.bilibili.com/video/BV1JY4y1Y7NW/?spm_id_from=333.337.search-card.all.click&vd_source=d053f150135c9a2d7b7534c6bbd407d9)

2、
