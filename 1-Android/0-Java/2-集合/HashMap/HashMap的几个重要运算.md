> version：2022/04/
>
> review：



目录

[TOC]

1、tableSizeFor() 方法

返回一个大于给定int值的2次方值。

比如，传入 1，返回2。传入10，返回16。

```java
    /**
     * Returns a power of two size for the given target capacity.
     */    
	static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

解析：

首先，这个方法的目的是要得到一个大于给定int的2的n次方值。

因为任意一个int值，它的最高位都是1，那要得到一个最近的2的n次方值，只要将其低位全部变成1，然后再加1，就可以了。

比如，10

二进制是：1010

目标是得到：16，即10000

那使用，1111 + 0001 就可以了。

所以这个问题变成了如何将 1010 变成 1111

上述方法就是这样实现的（先不用管 cap -1 ）：

 n |= n >>> 1  --  》 将高2位都变成1。

n |= n >>> 2 ，将高4位都变成1

n |= n >>> 4 ，将高8位都变成1
n |= n >>> 8 ，将高16位都变成1
n |= n >>> 16 ， 将高位都变成1



2、

```java
    /**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

h >>> 16 的作用是将高16位，引入hash值的计算，从而降低冲突的几率。

为啥要引入高16位，是因为hash值，会被用来计算index，而index大多数情况下，都是小于2^16次方的，所以如果不引入高16位，那参与计算的总是低16位，这样hash碰撞的几率就增大了。

下面看下，如何用 hash 值算 index。

3、index计算

通过数组的length，hash值来计算。

```java
n = tab.length
tab[(n - 1) & hash]
```

length 的 取值永远是 2 的n次方。

index的取值就是，0 — length -1。

(n - 1) 的范围就是 0 — length -1，并且二进制有效位永远全是 1，因为n是2的n次方。高位都是0。

所以使用(n - 1) & hash就可以得到 hash值 对应的 index 值。



# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结



# 参考

1、[HashMap中hash(Object key)原理，为什么(hashcode ＞＞＞ 16)。](https://blog.csdn.net/qq_42034205/article/details/90384772?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-90384772-blog-105092989.pc_relevant_multi_platform_whitelistv1&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-90384772-blog-105092989.pc_relevant_multi_platform_whitelistv1&utm_relevant_index=1)

