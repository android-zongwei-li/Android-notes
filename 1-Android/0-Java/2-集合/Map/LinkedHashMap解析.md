

包括LinkedHashMap 常用 api 的源码解析以及利用 LinkedHashMap 来实现 LRUCache 算法。

说明：

- LinkedHashMap 是一个双向链表，包含了指向链表头、尾的指针：head 和 tail，正是由于这个特性，所以 LinkedHashMap 是有序的，而 HashMap 是无序的。
- LinkedHashMap 存储的 Entey 对象和 HashMap 的不一样，因为是双向链表，所以肯定存在指向上个 Node 的指针 before 以及指向下一个 Node 的指针 after
- LinkedHashMap 还有一个特有的布尔变量 accessOrder，true 表示链表按访问顺序，false 表示按插入顺序，

什么是插入顺序和访问顺序？

如下示例：

```java
 LinkedHashMap<String, String> linkedHashMap = new LinkedHashMap<>(16, 0.75f, true);
 linkedHashMap.put("1", "a");
 linkedHashMap.put("2", "b");
 linkedHashMap.put("3", "c");
```

第3个参数就表示 accessOrder 的值为 true，我们现在什么也不做直接输出链表的值看看：

![图片](images/LinkedHashMap解析/640)

现在我们调用 linkedHashMap.get(“1”)，结果是什么呢？

![图片](images/LinkedHashMap解析/640-20240408145533507)

可以看出结果不一样了，key = “1” 的结点放到了链表的最后，这个就表示按访问顺序了，当 accessOrder的值为 true，那么get后会把这个Node放在链表的末尾，这就是利用LinkedHashMap实现LRUCache（Least Recently Used，最近最少使用）的核心思想了。而 accessOrder的值为false时并不会将该结点放到链表的末尾，后续会讲到为什么是会这样的。



# (n - 1) & hash

在 HashMap 中，会通过 (n - 1) & hash 来计算 table index，原理是：

`&` 通常表示按位与（bitwise AND）操作，而 `hash` 是一个哈希值，用于快速查找或数据结构的组织。`(n - 1) & hash` 这个表达式经常出现在一些哈希表（hash table）或位图（bitmap）的实现中，特别是在处理哈希冲突或确定元素在数组中的位置时。

具体来说，`(n - 1) & hash` 这个表达式通常用于将哈希值映射到一个更小的范围内，这个范围由 `n` 的值确定。这里的 `n` 通常是哈希表或数组的大小。减一（`n - 1`）操作是为了确保结果是一个非负整数，并且其二进制表示中的所有位都是1（如果 `n` 是2的幂的话）。

举个例子，如果 `n` 是8（即2的3次方），那么 `n - 1` 就是7，其二进制表示为 `0111`。如果 `hash` 的二进制表示中的任何一位与 `0111` 的对应位都为1，那么该位的结果在按位与操作中也会是1。这样，`(n - 1) & hash`的结果就确保了 `hash` 被映射到了一个0到7之间的值，这个值可以用作数组中的索引。

这种技术的一个主要优点是，当 `n` 是2的幂时，它可以将哈希值均匀地分布到整个范围内，从而减少了哈希冲突的可能性。这是因为任何与 `n - 1` 进行按位与操作的数都会保留其最低的几个位，而这些位通常包含哈希值中最重要的信息。

需要注意的是，这种映射方法并不保证没有哈希冲突（即不同的 `hash` 值可能映射到相同的索引），但它通常可以减少冲突的数量，从而提高哈希表的性能。

# Put 操作

LinkedHashMap 继承了HashMap，LinkedHashMap 并没有重写父类 HashMap 的 put 方法，只是重写了put操作中的部分方法，看代码：

## put 源码

```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; 
   			Node<K,V> p; 
   			int n, i;
        // 注释1
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;// 最开始时table为空，初始化 table
        if ((p = tab[i = (n - 1) & hash]) == null) // 注释2
            tab[i] = newNode(hash, key, value, null);// 注释3 LinkedHashMap 中重写此方法
        else {
            Node<K,V> e; 
          	K k;
            // 注释4 如果是相同的key则替换value的值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode) // 如果是红黑树 则执行红黑树的插入操作
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                // 注释5 表示hash冲突了，找到冲突结点所在链表的最后一个结点，把当前结点插入到最后一个结点的后面
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 注释6
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;// 注释7
                }
            }
            // 注释8 表示key相同 则更新并返回旧的值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e); // 注释9 LinkedHashMap 中重写此方法，作用就是将这个结点放到链表的尾部
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold) // 注释10
            resize();
        afterNodeInsertion(evict); // 注释11 LinkedHashMap 中重写此方法， LRU用来删除头结点的方法，
        return null;
    }
```

## put 实例分析

下面以实例来分析，为了方便说明，我这里的数组长度为 4，假设我们要put 3个元素，3个元素的key分别为 m，n和t；假设m的计算出来的hash值为5， n的计算出来的hash值为6，t的计算出来的hash值为9。

Key	m	n	t

hash	5	6	9

HashMap会进行分组，也就是 table 数组，然后 hash 值会被用来计算插入的对象在数组中的 index。

### 第一步：插入 key 为 m 的数据（m 的 hash = 5）：

走到注释1：由于是第一次插入数据所以注释1的条件成立，先要执行resize方法进行table的初始化，这个可以看HashMap源码分析。

走到注释2：p = tab[i = (n - 1) & hash]，我们之前假设了链表的初始长度等于4，所以 (n - 1) & hash = 3&5 = 1，即将key为m的数据插入到数组 table index = 1的位置。

走到注释3：tab[i] = newNode(hash, key, value, null)，LinkedHashMap 重写了newNode 方法，如下：

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMapEntry<K,V> p =
            new LinkedHashMapEntry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

 private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
        LinkedHashMapEntry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

LinkedHashMapEntry是LinkedHashMap中插入的Entry对象，包含了 before, after指针。

linkNodeLast 方法是将待插入结点p插入到链表的末尾。具体分析我通过画图来解释：

由于是第一次插入，所以 tail = null，即 last = null，然后让tail 指向待插入的结点 p，由于 last = null，所以 if 条件语句成立，即 head 也指向待插入的 结点 p, 如下图：

![图片](images/LinkedHashMap解析/640-20240410102417811)

然后将这个结点放到数组 tab index = 1 的位置（tab[i] = newNode(hash, key, value, null);），如下图：

![图片](images/LinkedHashMap解析/640-20240410102429675)

下一步走到注释10。

注释10：判断size是否大于阈值，这里的阈值 threshold = 4*0.75 = 3，然后走到注释11，注释11先不说，下面再说。

### 第二步：插入key 为 n 的数据（n 的 hash = 6）：

走到注释2 ：p = tab[i = (n - 1) & hash] ， (n - 1) & hash = 3&6 = 2，即插入结点的位置位于数组tab的索引等于2。由于数组索引等于2的位置没有元素，所以执行注释3。

注释3：tab[i] = newNode(hash, key, value, null) 即new一个结点放入索引等于2的位置。下面通过画图来解释 newNode 方法：

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMapEntry<K,V> p =
            new LinkedHashMapEntry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

 private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
        LinkedHashMapEntry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

首先：LinkedHashMapEntry<K,V> last = tail; 即让局部变量last指向链表的尾部，如下：

![图片](images/LinkedHashMap解析/640-20240410103412115)

然后: tail = p，让tail指向当前待插入的结点，如下：

![图片](images/LinkedHashMap解析/640-20240410103423172)

然后：if (last == null) 由于last指向了上一个结点 m，所以 if 条件不成立，走到下面的else 语句：p.before = last; last.after = p;如下图：

![图片](images/LinkedHashMap解析/640-20240410104015853)可以看出执行完上面的2步操作后，tail 指向链表的尾部，且形成了双向链表。

### 第三步：插入key 为 t 的数据（t 的 hash = 9）：

走到注释2：p = tab[i = (n - 1) & hash] ，(n - 1) & hash = 3&9 = 1，tab[1]之前插入了key 为 m 的结点，所以走到注释4：

注释4：if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) 判断当前的key和之前插入的key是否相等， 之前的key m 和 现在的 key t 不相等，所以走到注释5。

注释5：if ((e = p.next) == null) 由注释2可知p执行key为m的结点，p.next = null，所以条件成立，走到下面的 p.next = newNode(hash, key, value, null);

结果就是将 p 的 next 指针指向当前插入的结点。接着分析 newNode：

```kotlin
 private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
        LinkedHashMapEntry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

首先：LinkedHashMapEntry<K,V> last = tail 即让last执行尾结点 tail，尾指针 tail 指向待插入的结点p，如下图：

![图片](images/LinkedHashMap解析/640-20240410104108235)

然后：if (last == null) 由于last指向上一个结点，即key为n的结点，所以if 条件不成立，走到else中p.before = last; last.after = p;

![图片](images/LinkedHashMap解析/640-20240410111237157)

然后p.next = newNode(hash, key, value, null);，p 执向的是key为m的结点。最终图如下：

![图片](images/LinkedHashMap解析/640-20240410111246240)

插入3个数据后的示意图如上，为了研究扩容，再插入一个数据r，假设 r 的 hash = 7；

### 插入key 为 r 的数据（r 的 hash = 7）

根据之前的公式算出key为r的结点在数组table中的index， (n - 1) & hash = 3 & 7 = 3，所以将该结点插入到table数组的最后。具体的分析就不讲了，最终的图如下：

![图片](images/LinkedHashMap解析/640-20240410111705617)

插入第4个数据后，tail 指针指向最后插入的key 为 r 的结点。这时候走到注释10。if (++size > threshold) 其中size++后等于4 大于 threshold （threshold= 3），所以要进行扩容。即走到 resize 方法。

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
      。。。。。。
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) { // 注释1
                    oldTab[j] = null;
                    if (e.next == null)  // 注释2
                        newTab[e.hash & (newCap - 1)] = e;  // 注释3
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { //  // 注释4 preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;// 注释5
                            if ((e.hash & oldCap) == 0) {  // 注释6
                                if (loTail == null) // 注释7
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;  // 注释8
                            }
                            else { // 注释9
                                if (hiTail == null)  // 注释10
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;  // 注释11
                            }
                        } while ((e = next) != null);  // 注释12
                        if (loTail != null) { // 注释13
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) { // 注释14
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

省略号的部分不重要 就是计算 新数组的容量和阈值，新数组的容量和阈值等于老数组的2倍，即新的table容量等于8，新的阈值threshold = 6；下面就是将老数组中的数据移到新的数组中。具体操作就是遍历老数组中的数据，然后移到新数组中相应的位置。

走到注释1：if ((e = oldTab[j]) != null) 由于老数组中index为0的位置未插入数据，所以条件不成立，++j 后继续循环，依然走到注释1，由于老数组中index=1中插入的结点key = m，即条件成立，走到注释2。

注释2 ：if (e.next == null) 由之前的结构图可知，e.next 指向的是 key=t的结点，即if 条件不成立，走到注释4。

注释4：这里面就是定义了一些变量 ：loHead， loTail， hiHead， hiTail 以及 next，接着就是do…while循环了，走到注释5。

注释5：next = e.next; 即变量next指向key为t的结点，如下图：

![图片](images/LinkedHashMap解析/640-20240410113059844)

注释6：if ((e.hash & oldCap) == 0)，oldCap 就是老数组的容量。即等于 4，oldCap应该是 oldCapacity 的省略写法。e 指向的结点 m 的hash值等于 5， 5&4 = 4 所以if条件不成立，走到注释10。

注释10：if (hiTail == null) hiHead 是之前定义的变量赋值为 null，所以条件成立，执行 hiHead = e; 接着执行注释11，hiTail = e;如下图：

![图片](images/LinkedHashMap解析/640-20240410113434698)

接着执行注释12：while ((e = next) != null)， e 赋值为 key 为 t 的结点，所以不等于 null。如下图：

![图片](images/LinkedHashMap解析/640-20240410113539660)

即while条件满足，继续执行do while ，走到注释5 next = e.next; 由于 e 此时没有了后续结点了，所以 e.next = null，即 next = null；接着到 注释6。

注释6：if ((e.hash & oldCap) == 0) e 指向结点的 key = t，hash值为 9 ，9&4 = 0，条件成立，走到注释7。

注释7：if (loTail == null) loTail 是之前定义的局部变量且未被赋值，所以条件成立，执行loHead = e; 和注释8的 loTail = e，如下图：

![图片](images/LinkedHashMap解析/640-20240410113720871)

接着执行注释12：while ((e = next) != null) next等于null 所以 e = null，while 条件不成立。执行到注释13。

注释13：if (loTail != null) 条件成立：执行 loTail.next = null; newTab[j] = loHead; 执行的结果就是将 key 为 t 的node放到新数组index = 1的位置。接着执行注释14。

注释14：if (hiTail != null) hiTail 指向的结点的 key 为 m，所以，条件成立；执行 hiTail.next = null; newTab[j + oldCap] = hiHead; 执行的结果就是 将 key 为 m 的结点放到新数组。所以等于 j + oldCap = 1+4 = 5 的位置。

接着 j++ j =2，老数组索引等于 2 的位置存放的是 key = n 的结点，所以 注释1 if ((e = oldTab[j]) != null) 条件成立， e 指向key=n的结点，此结点没有后继结点（结点的next=null），所以注释2，if (e.next == null)成立， 执行注释3。

注释3：newTab[e.hash & (newCap - 1)] = e; n 的hash值为6 ，newCap 新数组的容量为8，所以 e.hash&newCap= 6&7 = 6，即将 key = n 的结点放到新数组索引值为6的位置。

同理将 key = r ，hash = 7 的结点插入的位置为 7&7 = 7；最终扩容后的结果如下图：

![图片](images/LinkedHashMap解析/640-20240410142719003)

现在put 和 resize 扩容操作基本讲完了。接下来讲get方法，我们这里的 accessOrder = true，LinkedHashMap<String, String> linkedHashMap = new LinkedHashMap<>(4, 0.75f, true); 就是前面讲到的按访问顺序来的，即通过get或者put Map中已有的 key 的操作，会将相应的结点移动到链表尾部。来看看源码是怎么实现的。

# get操作

我们还是用插入了4条数据的Map来分析，但是没扩容，为的是好理解，即插入 key 分别为 m (hash = 5) 、n(hash = 6)、t (hash = 9)、 r(hash = 7) 如下图。

![图片](images/LinkedHashMap解析/640-20240410143208866)

我们直接get一个最复杂的即get(t) 。get代码如下：

```java
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

getNode代码是在父类HashMap中实现的：

```java
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; 
  			Node<K,V> first, e; 
  			int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) { // 注1
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k)))) // 注2
                return first;
            if ((e = first.next) != null) { // 注3
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k)))) // 注4
                        return e;
                } while ((e = e.next) != null); // 注5
            }
        }
        return null;
    }
```

走到注1：if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) first = tab[(n - 1) & hash]，t 的 hash 值为9 ， n 为数组的长度等于 4，3&9 = 1，first 等于数组中索引等于1的结点，即 key = m 的结点；


走到注2：if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) 由于我们get key = t的数据，所以注2不成立，走到注3。

注3：if ((e = first.next) != null) first.next 指向 key 为m 结点的后继结点即 key = t 的结点。走到注释4。

注4：if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) 条件成立返回找到的结点 e，return e;接着执行LinkedHashMap中get方法的剩下代码：if (accessOrder) afterNodeAccess(e); 由于accessOrder等于true，所以执行afterNodeAccess方法。如下：

```java
// 将结点e移到链表的尾部
 void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMapEntry<K,V> last;
        // 判断 accessOrder 和 当前结点 e 是否是链表的尾结点
        if (accessOrder && (last = tail) != e) { // 注1
            LinkedHashMapEntry<K,V> p =
                (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after; // 注2
            p.after = null;
            if (b == null) // 注3
                head = a;
            else
                b.after = a;
            if (a != null) // 注4
                a.before = b;
            else
                last = b;
            if (last == null)// 注5
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p; // 注6
            ++modCount;
        }
    }
```

执行完注1和注2后的图如下：

![图片](images/LinkedHashMap解析/640-20240410160554019)

接着执行 p.after = null; 如下图：

![图片](images/LinkedHashMap解析/640-20240410161238787)

红色的X号表示断开连接。

走到注释3：if (b == null) b 指向 key = n的结点，所以条件不成立，执行 b.after = a; 如下图：

![图片](images/LinkedHashMap解析/640-20240410162150134)

我把断开连接的那条线去掉了，这样更清晰。接着执行注释4：if (a != null)， a 执行的key= r的结点，所以条件成立，执行 a.before = b; 如下图：

![图片](images/LinkedHashMap解析/640-20240410162237170)

接着执行注释5：if (last == null) last 指向尾结点即key = r的结点，所以条件不成立，执行p.before = last; last.after = p; 执行后如下图：

![图片](images/LinkedHashMap解析/640-20240410162335473)

最后执行：tail = p; 即将该结点设为尾结点，如下图：

![图片](images/LinkedHashMap解析/640-20240410162351002)

这样的话， key = t 的结点就移到链表的末尾啦。刚才讲了 put 的 key 在map中已存在的话也会导致该结点会移到链表的尾部，在代码的哪里体现的呢？其实是在 putVal 方法中体现的，如下：

```java
...
// 注释8 表示key相同 则更新并返回旧的值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);// 注释9 LinkedHashMap 中重写此方法，作用就是将这个结点放到链表的尾部
                return oldValue;
            }
            ...
```

最后分析下是怎么实现LRUCache 的，LRU 表示的是最近最少使用，我们通过给LinkedHashMap 是在 accessOrder = true 后，就保证了 最近使用的会放到链表的尾部，那么我们可以猜想到达到删除条件的时候肯定删除的是链表头部的结点，是不是这样的呢？还有是不是设置了 accessOrder = true 就可以实现LRUCache了呢？

我们通过源码来给出强有力的答案吧。

什么时候会触发删除结点的条件呢？我们想一想，猜测肯定是 put 数据的时候，对不对，看源码。我们可以看到 put 方法的最后会调用 afterNodeInsertion(evict); 这个方法在 LinkedHashMap 中实现了，如下：

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMapEntry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key; // first 指向的是头结点，即删除链表头的结点
            removeNode(hash(key), key, null, false, true);
        }
    }
```

其中evict默认是true , 而 removeEldestEntry 默认返回的是false，所以要想实现LRUCache不光要设置 accessOrder = true，还要重写此方法。重写如下：

```java
private MyLRUCache() {
        map = new LinkedHashMap<String, String>(6, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, String> entry) {
                //当map的size 大于3的时候就删除表头结点
                return map.size() > 3;
            }
        };
    }
```

然后就是看真正的删除结点的算法啦，removeNode 是在 HashMap 中实现的，我这里只摘出部分代码：

```java
// node 表示的就是待删除的头结点 index 表示头结点在table数组中的索引值
tab[index] = node.next; // 注1 
 afterNodeRemoval(node); // 注2
```

执行完注1后的图如下：

![图片](images/LinkedHashMap解析/640-20240410163038883)

接着执行 afterNodeRemoval 如下：

```java
 LinkedHashMapEntry<K,V> p =
            (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after; // 注1
        p.before = p.after = null; // 注2
        if (b == null)// 注3
            head = a;
        else
            b.after = a;
        if (a == null)// 注4
            tail = b;
        else
            a.before = b;
```

执行完注1后，如下图：

![图片](images/LinkedHashMap解析/640-20240410163308874)

由于删除的是头结点，所以 b = p.before = null， a 指向 key = n 的结点，注2：p.before = p.after = null; 执行完如下图，即断开p.after的那条链接。

![图片](images/LinkedHashMap解析/640-20240410164308882)

注3：if (b == null) 条件成立，执行 head = a; head指向结点a，如下图：

![图片](images/LinkedHashMap解析/640-20240410164316478)

注4：if (a == null) 条件不成立，执行 a.before = b; 断开结点a执行结点p的连接，如下：

![图片](images/LinkedHashMap解析/640-20240410164415327)

这样整个删除头结点的操作就执行完了。这也是实现LRUCache的实现原理了。

# 总结

1. LinkedHashMap 是有序的，是双向链表
2. LinkedHashMap 如果 accessOrder 设置为true，那么它是按访问顺序的。
3. 利用 LinkedHashMap 实现LRUCache 必须同时满足 accessOrder = true 且重写 removeEldestEntry 方法

最后附上自定义的LRUCache 代码：

```java
public class MyLRUCache {
    private static volatile MyLRUCache mInstance;
    private LinkedHashMap<String, String> map;

    private MyLRUCache() {
        map = new LinkedHashMap<String, String>(6, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, String> entry) {
                //当map的size 大于3的时候就删除表头结点
                return map.size() > 3;
            }
        };
    }

    public static MyLRUCache getInstance() {
        if (mInstance == null) {
            synchronized (MyLRUCache.class) {
                if (mInstance == null) {
                    mInstance = new MyLRUCache();
                }
            }
        }
        return mInstance;
    }

    public void put(String key, String value) {
        map.put(key, value);
    }

    public String get(String key) {
        return map.get(key);
    }

    public void traverse() {
        Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();
        while (iterator.hasNext()) {
            Map.Entry<String, String> entry = iterator.next();
            System.out.println("key = " + entry.getKey() + ", value = " + entry.getValue());
        }      
    }
}
```

# 参考

1、[图解LinkedHashMap](https://mp.weixin.qq.com/s/tM27YqkHnDchvWHBQ6iTGQ)