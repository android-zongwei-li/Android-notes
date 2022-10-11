> 知识路径：Android > Java > 集合
>
> version：2021/4/6
>
> review：2021/4/6
>
> 掌握程度：了解



### 一、预备知识

HashMap、ReentrantLock、volatile、自旋锁、互斥锁、CAS、synchronized。

### 二、ConcurrentHashMap

#### 2.1、Base 1.7

ConcurrentHashMap的最外层是一个Segment数组。

![image-20210406104910394](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406104910394.png)

![image-20210406104931606](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406104931606.png)

![image-20210406105101451](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406105101451.png)

![image-20210406105613957](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406105613957.png)

![image-20210406105649787](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406105649787.png)



#### 2.2、Base 1.8

![image-20210406105903639](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406105903639.png)

![image-20210406105943454](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406105943454.png)

![image-20210406110026560](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406110026560.png)

ConcurrentHashMap.java

```java
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                        else if (f instanceof ReservationNode)
                            throw new IllegalStateException("Recursive update");
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```





参考：

1、《Android核心知识点笔记V2020.03.30》