> version：2021/4/6
>
> review：2021/4/6



### 一、预备知识

volatile、synchronized、数组

### 二、CopyOnWriteArrayList



### 三、总结

CopyOnWriteArrayList是线程安全容器。其内部持有的array数组使用volatile修饰保证了线程见的可见性；又通过synchronized对add()、remove()方法进行加锁，保证了线程安全、数据一致性。

本质上也是对数组操作的封装。

对数组的复制涉及到了两个重要方法：Arrays.copyOf、System.copyarray；



# 参考

