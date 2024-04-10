> version：2021/4/2
>
> review：2020/12/27---2021/4/2



HashMap用到的数据结构

数组、单向链表、双向链表、红黑树





# Map 系列接口解析

在分析各个 Map 的实现原理之前，先来弄清楚各个 Map 要操作的对象，也就是 Map.Entry 接口及其子类。我们通过 put 放入的数据最终保存到 Entry 中，通过 get 获取的数据最终也是从各个 Entry 中拿到的。所以 Entry 是 Map 操作的最基本对象，了解各个 Entry 具有什么样的属性和方法就很有必要，会有助于我们后续理解 Map 对 Entry 的各种操作。

## Map

```java
public interface Map<K, V> {
    // Query Operations

  
    interface Entry<K, V> {
        K getKey();

        V getValue();
      
        V setValue(V value);

        boolean equals(Object o);
      
        int hashCode();

    ...还有几个 compare 相关的方法
    }
}
```

（1）hashCode() 是什么？有什么用？

先来看一个例子：现在有 100 个学生，每个学生有姓名（和其他特征），此时想要找到某个学生时，需要按照名称一个个去匹配。

有没有更快的办法呢？

当然，给每个学生一个 id 吧，然后按照 id 去给学生分组，当要找某个学生的时候，根据他的 id 先找到他所在的组，再找到他，这样显然更快。

在 Java 中，这个 id 就是 hashCode，下面看看 Java 中对 hashCode 是怎么定义的，我们在生成 hashCode 时应该遵循这些规则：

遵循这些约定是确保Java对象在哈希表等数据结构中正确工作的关键。如果你重写了 `equals()` 方法，那么通常也需要重写 `hashCode()` 方法，以确保这两个方法的行为是一致的。

> hashCode的一般约定是：
>
> 1、每当在Java应用程序的执行过程中对同一对象多次调用hashCode时，只要不修改对象上的equals比较中使用的信息，hashCode方法就必须始终返回相同的整数。从一个应用程序的一次执行到同一应用程序的另一次执行，该整数不需要保持一致。
>
> 2、如果根据equals（Object）方法，两个对象相等，那么对这两个对象中的每一个调用hashCode方法**必须产生相同的整数结果**。
>
> 3、如果根据equals（Object）方法，两个对象不相等，**并不要求**这两个对象中的每一个调用hashCode方法必须产生不同的整数结果。然而，程序员应该意识到，为不相等的对象生成不同的整数结果可能会提高哈希表的性能。
>
> Object#hashCode() 方法注释

所以，在遵循此预定的情况下，equals 和 hashCode 的关系如下：

equals 相等时，hashCode 必须相等；

equals 不相等时，hashCode 可以相等，也可以不同，当然不同最好。

hashCode 相同时，两个对象可能不一样，当出现这种情况时，就称为 hash 碰撞——两个不同对象，有相同 hashCode。哈希碰撞是不可避免的，因为哈希码的取值范围是有限的，而对象的数量可能是无限的。

hashCode 不同，equals 一定不相等。

作用：hashCode 是为了 Map 可以查询的更快；equals 是用来判断对象是否相等的。

（2）为什么有 setValue，怎么没有 setKey 呢

对于每个 Entry，其 Key 只有一个，并且都是通过构造方法传入，所以不需要 setKey 方法。

（3）和 Key 一样，每个 Entry 的 hashCode 也是只有一个，并且通过构造方法传入。

## HashMap

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

		static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;

            return o instanceof Map.Entry<?, ?> e
                    && Objects.equals(key, e.getKey())
                    && Objects.equals(value, e.getValue());
        }
    }


    static final class TreeNode<K,V> extends LinkedHashMap.LinkedHashMapEntry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }

        /**
         * Ensures that the given root is the first node of its bin.
         */
        static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
            int n;
            if (root != null && tab != null && (n = tab.length) > 0) {
                int index = (n - 1) & root.hash;
                TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
                if (root != first) {
                    Node<K,V> rn;
                    tab[index] = root;
                    TreeNode<K,V> rp = root.prev;
                    if ((rn = root.next) != null)
                        ((TreeNode<K,V>)rn).prev = rp;
                    if (rp != null)
                        rp.next = rn;
                    if (first != null)
                        first.prev = root;
                    root.next = first;
                    root.prev = null;
                }
                assert checkInvariants(root);
            }
        }

        /**
         * Finds the node starting at root p with the given hash and key.
         * The kc argument caches comparableClassFor(key) upon first use
         * comparing keys.
         */
        final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
            TreeNode<K,V> p = this;
            do {
                int ph, dir; K pk;
                TreeNode<K,V> pl = p.left, pr = p.right, q;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.find(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
            return null;
        }

        /**
         * Calls find for root node.
         */
        final TreeNode<K,V> getTreeNode(int h, Object k) {
            return ((parent != null) ? root() : this).find(h, k, null);
        }

        /**
         * Tie-breaking utility for ordering insertions when equal
         * hashCodes and non-comparable. We don't require a total
         * order, just a consistent insertion rule to maintain
         * equivalence across rebalancings. Tie-breaking further than
         * necessary simplifies testing a bit.
         */
        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }

        /**
         * Forms tree of the nodes linked from this node.
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
        }

        /**
         * Returns a list of non-TreeNodes replacing those linked from
         * this node.
         */
        final Node<K,V> untreeify(HashMap<K,V> map) {
            Node<K,V> hd = null, tl = null;
            for (Node<K,V> q = this; q != null; q = q.next) {
                Node<K,V> p = map.replacementNode(q, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }

        /**
         * Tree version of putVal.
         */
        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }

        /**
         * Removes the given node, that must be present before this call.
         * This is messier than typical red-black deletion code because we
         * cannot swap the contents of an interior node with a leaf
         * successor that is pinned by "next" pointers that are accessible
         * independently during traversal. So instead we swap the tree
         * linkages. If the current tree appears to have too few nodes,
         * the bin is converted back to a plain bin. (The test triggers
         * somewhere between 2 and 6 nodes, depending on tree structure).
         */
        final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
                                  boolean movable) {
            int n;
            if (tab == null || (n = tab.length) == 0)
                return;
            int index = (n - 1) & hash;
            TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
            TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
            if (pred == null)
                tab[index] = first = succ;
            else
                pred.next = succ;
            if (succ != null)
                succ.prev = pred;
            if (first == null)
                return;
            if (root.parent != null)
                root = root.root();
            if (root == null
                || (movable
                    && (root.right == null
                        || (rl = root.left) == null
                        || rl.left == null))) {
                tab[index] = first.untreeify(map);  // too small
                return;
            }
            TreeNode<K,V> p = this, pl = left, pr = right, replacement;
            if (pl != null && pr != null) {
                TreeNode<K,V> s = pr, sl;
                while ((sl = s.left) != null) // find successor
                    s = sl;
                boolean c = s.red; s.red = p.red; p.red = c; // swap colors
                TreeNode<K,V> sr = s.right;
                TreeNode<K,V> pp = p.parent;
                if (s == pr) { // p was s's direct parent
                    p.parent = s;
                    s.right = p;
                }
                else {
                    TreeNode<K,V> sp = s.parent;
                    if ((p.parent = sp) != null) {
                        if (s == sp.left)
                            sp.left = p;
                        else
                            sp.right = p;
                    }
                    if ((s.right = pr) != null)
                        pr.parent = s;
                }
                p.left = null;
                if ((p.right = sr) != null)
                    sr.parent = p;
                if ((s.left = pl) != null)
                    pl.parent = s;
                if ((s.parent = pp) == null)
                    root = s;
                else if (p == pp.left)
                    pp.left = s;
                else
                    pp.right = s;
                if (sr != null)
                    replacement = sr;
                else
                    replacement = p;
            }
            else if (pl != null)
                replacement = pl;
            else if (pr != null)
                replacement = pr;
            else
                replacement = p;
            if (replacement != p) {
                TreeNode<K,V> pp = replacement.parent = p.parent;
                if (pp == null)
                    (root = replacement).red = false;
                else if (p == pp.left)
                    pp.left = replacement;
                else
                    pp.right = replacement;
                p.left = p.right = p.parent = null;
            }

            TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);

            if (replacement == p) {  // detach
                TreeNode<K,V> pp = p.parent;
                p.parent = null;
                if (pp != null) {
                    if (p == pp.left)
                        pp.left = null;
                    else if (p == pp.right)
                        pp.right = null;
                }
            }
            if (movable)
                moveRootToFront(tab, r);
        }

        /**
         * Splits nodes in a tree bin into lower and upper tree bins,
         * or untreeifies if now too small. Called only from resize;
         * see above discussion about split bits and indices.
         *
         * @param map the map
         * @param tab the table for recording bin heads
         * @param index the index of the table being split
         * @param bit the bit of hash to split on
         */
        final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            TreeNode<K,V> b = this;
            // Relink into lo and hi lists, preserving order
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }

            if (loHead != null) {
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }

        /* ------------------------------------------------------------ */
        // Red-black tree methods, all adapted from CLR

        static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                              TreeNode<K,V> p) {
            TreeNode<K,V> r, pp, rl;
            if (p != null && (r = p.right) != null) {
                if ((rl = p.right = r.left) != null)
                    rl.parent = p;
                if ((pp = r.parent = p.parent) == null)
                    (root = r).red = false;
                else if (pp.left == p)
                    pp.left = r;
                else
                    pp.right = r;
                r.left = p;
                p.parent = r;
            }
            return root;
        }

        static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                               TreeNode<K,V> p) {
            TreeNode<K,V> l, pp, lr;
            if (p != null && (l = p.left) != null) {
                if ((lr = p.left = l.right) != null)
                    lr.parent = p;
                if ((pp = l.parent = p.parent) == null)
                    (root = l).red = false;
                else if (pp.right == p)
                    pp.right = l;
                else
                    pp.left = l;
                l.right = p;
                p.parent = l;
            }
            return root;
        }

        static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                                    TreeNode<K,V> x) {
            x.red = true;
            for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
                if ((xp = x.parent) == null) {
                    x.red = false;
                    return x;
                }
                else if (!xp.red || (xpp = xp.parent) == null)
                    return root;
                if (xp == (xppl = xpp.left)) {
                    if ((xppr = xpp.right) != null && xppr.red) {
                        xppr.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.right) {
                            root = rotateLeft(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateRight(root, xpp);
                            }
                        }
                    }
                }
                else {
                    if (xppl != null && xppl.red) {
                        xppl.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.left) {
                            root = rotateRight(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateLeft(root, xpp);
                            }
                        }
                    }
                }
            }
        }

        static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                                   TreeNode<K,V> x) {
            for (TreeNode<K,V> xp, xpl, xpr;;) {
                if (x == null || x == root)
                    return root;
                else if ((xp = x.parent) == null) {
                    x.red = false;
                    return x;
                }
                else if (x.red) {
                    x.red = false;
                    return root;
                }
                else if ((xpl = xp.left) == x) {
                    if ((xpr = xp.right) != null && xpr.red) {
                        xpr.red = false;
                        xp.red = true;
                        root = rotateLeft(root, xp);
                        xpr = (xp = x.parent) == null ? null : xp.right;
                    }
                    if (xpr == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpr.left, sr = xpr.right;
                        if ((sr == null || !sr.red) &&
                            (sl == null || !sl.red)) {
                            xpr.red = true;
                            x = xp;
                        }
                        else {
                            if (sr == null || !sr.red) {
                                if (sl != null)
                                    sl.red = false;
                                xpr.red = true;
                                root = rotateRight(root, xpr);
                                xpr = (xp = x.parent) == null ?
                                    null : xp.right;
                            }
                            if (xpr != null) {
                                xpr.red = (xp == null) ? false : xp.red;
                                if ((sr = xpr.right) != null)
                                    sr.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateLeft(root, xp);
                            }
                            x = root;
                        }
                    }
                }
                else { // symmetric
                    if (xpl != null && xpl.red) {
                        xpl.red = false;
                        xp.red = true;
                        root = rotateRight(root, xp);
                        xpl = (xp = x.parent) == null ? null : xp.left;
                    }
                    if (xpl == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpl.left, sr = xpl.right;
                        if ((sl == null || !sl.red) &&
                            (sr == null || !sr.red)) {
                            xpl.red = true;
                            x = xp;
                        }
                        else {
                            if (sl == null || !sl.red) {
                                if (sr != null)
                                    sr.red = false;
                                xpl.red = true;
                                root = rotateLeft(root, xpl);
                                xpl = (xp = x.parent) == null ?
                                    null : xp.left;
                            }
                            if (xpl != null) {
                                xpl.red = (xp == null) ? false : xp.red;
                                if ((sl = xpl.left) != null)
                                    sl.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateRight(root, xp);
                            }
                            x = root;
                        }
                    }
                }
            }
        }

        /**
         * Recursive invariant check
         */
        static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
            TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
                tb = t.prev, tn = (TreeNode<K,V>)t.next;
            if (tb != null && tb.next != t)
                return false;
            if (tn != null && tn.prev != t)
                return false;
            if (tp != null && t != tp.left && t != tp.right)
                return false;
            if (tl != null && (tl.parent != t || tl.hash > t.hash))
                return false;
            if (tr != null && (tr.parent != t || tr.hash < t.hash))
                return false;
            if (t.red && tl != null && tl.red && tr != null && tr.red)
                return false;
            if (tl != null && !checkInvariants(tl))
                return false;
            if (tr != null && !checkInvariants(tr))
                return false;
            return true;
        }
    }
  
}
```

Node<K,V> 显然是单链表的节点。



## LinkedHashMap

```kotlin
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
  
    static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    } 
  
  
}
```





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
