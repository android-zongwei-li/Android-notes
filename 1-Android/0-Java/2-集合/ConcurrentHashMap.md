> version：2021/4/6
>
> review：2021/4/6
>



# 一、预备知识

HashMap、ReentrantLock、volatile、自旋锁、互斥锁、CAS、synchronized。

# 二、ConcurrentHashMap

## 1、简介

ConcurrentHashMap是Java中提供的一个线程安全的HashMap实现，它采用分段锁和CAS（Compare and Swap）操作等技术来实现高并发和线程安全。下面我们结合ConcurrentHashMap的内部源码解释**分段锁、CAS操作、扩容机制、近似计数**等技术如何实现的。

## 2、并发操作方法

ConcurrentHashMap提供了一些用于并发操作的方法，如putIfAbsent()、replace()、remove()等。这些方法可以在一个原子操作中完成检查和更新，从而避免多线程环境下的竞争条件。

例如，下面的代码展示了使用putIfAbsent()方法来实现一个线程安全的缓存：

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapTest {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();

        String key = "firstKey";
        String value = "firstValue";
        cache.putIfAbsent(key, value);
    }
}
```

1. 创建 ConcurrentHashMap 来存储缓存数据。
2. 使用putIfAbsent()可添加一个键值对，且只在键不存在时才添加键值对，从而避免覆盖已存在的值。

## 3、遍历ConcurrentHashMap

ConcurrentHashMap的遍历操作也是线程安全的。它提供了keySet、values和entrySet等方法，可以返回Map的键集、值集或键值对集。这些方法返回的集合是ConcurrentHashMap的视图，它们会反映ConcurrentHashMap的实时状态。也就是说，在遍历这些集合的过程中，其他线程对ConcurrentHashMap的修改操作是可见的。

例如，下面的代码展示了如何遍历ConcurrentHashMap：

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapTest {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();
        // 添加键值对
        cache.put("one", 1);
        cache.put("two", 2);
        cache.put("three", 3);
        // 使用 putIfAbsent 不会再次插入
        cache.putIfAbsent("three", 30);

        // 遍历ConcurrentHashMap
        // 这个遍历操作是线程安全的，即使在遍历过程中有其他线程修改ConcurrentHashMap，
        // 也不会抛出ConcurrentModificationException
        for (String k : cache.keySet()) {
            System.out.println("k = " + k + " , " + cache.get(k));
        }
    }
}

// 结果
k = one , 1
k = two , 2
k = three , 3
```

1. 创建一个ConcurrentHashMap实例
2. 使用put方法添加了一些键值对
3. 使用for-each循环遍历了整个ConcurrentHashMap。这个遍历操作是线程安全的，即使在遍历过程中有其他线程修改ConcurrentHashMap，也不会抛出ConcurrentModificationException。

## 4、扩展方法介绍

ConcurrentHashMap提供了一些并发编程中常用的扩展方法，如compute、merge等。这些方法可以在一个原子操作中完成复杂的更新逻辑，从而避免多线程环境下的竞争条件。

示例：使用`compute()`实现一个线程安全的计数器：

```java
import java.util.concurrent.ConcurrentHashMap;

public class Counter {
    private ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

    public void increment(String key) {
        map.compute(key, (k, v) -> (v == null) ? 1 : v + 1);
    }

    public int getCount(String key) {
        return map.getOrDefault(key, 0);
    }
}
```

1. 创建了一个Counter类，使用ConcurrentHashMap来存储计数数据。
2. 需要增加一个键的计数时，可以使用compute()方法，这个方法会在键存在时增加计数，否则初始化计数为1。

## 5、并发性能分析

ConcurrentHashMap采用了分段锁和CAS操作等技术，它在高并发环境下具有很好的性能。相比于同步的HashMap（如Hashtable或使用Collections.synchronizedMap包装的HashMap），**ConcurrentHashMap在读操作上几乎没有锁的开销，在写操作上也只需要锁定部分段**，因此并发性能更高。

然而，ConcurrentHashMap并不是万能的。在数据量较小或并发访问较低的情况下，简单的HashMap可能会更快。此外，ConcurrentHashMap也不能保证所有操作的全局有序性。如果需要全局有序性，可以考虑使用同步的Map实现，或者使用锁和其他同步工具来协调并发操作。

## 6、局限性与适用场景

虽然ConcurrentHashMap在并发环境下提供了很好的性能，但它也有一些局限性。

1. ConcurrentHashMap的所有操作都是线程安全的，但如果你需要执行复合操作（例如，先检查一个键是否存在，然后根据结果进行更新操作），那么就需要额外的同步措施来保证这些操作的原子性。因为在两个操作之间，可能有其他线程修改了ConcurrentHashMap的状态。
2. ConcurrentHashMap的size方法和isEmpty方法返回的结果是近似的，它们可能不会立即反映其他线程的修改操作。这是因为为了提高性能，ConcurrentHashMap没有使用全局锁来同步这些方法。
3. 如果应用场景中读操作远多于写操作，那么使用Read-Write Locks可能会获得更好的性能。Read-Write Locks允许多个线程同时读，但只允许一个线程写，这对于读多写少的场景是非常有效的。

## 7、一些注意事项

Key、Value不能传入null。

get()、getOrDefault()可能会返回null（当Key没有对应Value时），所以在拆箱并赋值时要注意处理空异常。

```kotlin
        ConcurrentHashMap<String, Integer> map1 = new ConcurrentHashMap<>();
        // 报错：map1.get("x")为null，类型是Integer，此时拆箱赋值给int，会报错
        int x = map1.get("x");
```

## 8、复合操作方法





# 原理

## 底层数据结构

JDK1.7 的 ConcurrentHashMap 底层采用 **分段的数组+链表** 实现。

JDK1.8 采用的数据结构跟 HashMap1.8 的结构一样，数组+链表/红黑二叉树。

## 实现线程安全的方式

- 在 JDK1.7 的时候，ConcurrentHashMap 对整个桶数组进行了分割分段(Segment，分段锁)，每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
- 在 JDK1.8 的时候，ConcurrentHashMap 已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6 以后 synchronized 锁做了很多优化） 整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。

## ConcurrentHahMap加锁的方式

1. 没有发生hash冲突的时候，如果添加的元素的位置在数组中是空的话，那么就使用**CAS**的方式来加入元素，这里加锁的粒度是数组中的元素
2. 如果出现了hash冲突，添加的元素的位置在数组中已经有了值，那么又存在三种情况
   - key相同，那么直接用新的元素覆盖旧的元素
   - 如果数组中的元素是链表的形式，那么将新的元素挂载在链表尾部
   - 如果数组中的元素是红黑树的形式，那么将新的元素加入到红黑树

第二种情况使用的是**synchronized**加锁，锁住的对象就是数组中的元素，加锁的粒度和第一种情况相同

**得出结论**：ConcurrentHashMap的分段加锁机制，其实锁住的就是数组中的元素，当操作数组中不同的元素时，是不会产生竞争的

[ConcurrentHashMap分段加锁机制简析](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2FLO_YUN%2Farticle%2Fdetails%2F106358362)

## ConcurrentHahMap底层图解

- ConcurrentHashMap在jdk1.7是通过分段数组和链表实现的
- ConcurrentHashMap在jdk1.8之后是通过数组+链表+红黑树进行是进行实现的

**JDK1.7 的 ConcurrentHashMap**：

![img](images/ConcurrentHashMap/1)

首先将数据分为一段一段（这个“段”就是 Segment）的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**ConcurrentHashMap** **是由** **Segment** **数组结构和** **HashEntry** **数组结构组成**。

Segment 继承了 ReentrantLock，所以 Segment 是一种可重入锁，扮演锁的角色。HashEntry 用于存储键值对数据。

```scala
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个 ConcurrentHashMap 里包含一个 Segment 数组，Segment 的个数一旦**初始化就不能改变**。 Segment 数组的大小默认是 16，也就是说默认可以同时支持 16 个线程并发写。

Segment 的结构和 HashMap 类似，是一种数组和链表结构，一个 Segment 包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素，每个 Segment 守护着一个 HashEntry 数组里的元素，当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment 的锁。也就是说，对同一 Segment 的并发写入会被阻塞，不同 Segment 的写入是可以并发执行的。

**JDK1.8 的 ConcurrentHashMap**：

![img](images/ConcurrentHashMap/2)

Java 8 几乎完全重写了 ConcurrentHashMap。取消了 Segment 分段锁，采用 Node + CAS + synchronized 来保证并发安全。数据结构跟 HashMap 1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）。

Java 8 中，锁粒度更细，**synchronized 只锁定当前链表或红黑二叉树的首节点**，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升。

## ConcurrentHashMap的key和value为什么不可以为null？

ConcurrentHashMap 的 key 和 value 不能为 null 主要是为了避免二义性。null 是一个特殊的值，表示没有对象或没有引用。如果用 null 作为键，那么就无法区分这个键是否存在于 ConcurrentHashMap 中，还是根本没有这个键。同样，如果用 null 作为值，那么就无法区分这个值是否是真正存储在 ConcurrentHashMap 中的，还是因为找不到键对应的值而返回的。

拿 get 方法取值来说，返回的结果为 null 存在两种情况：

- 值没有在集合中 ；
- 值本身就是 null。

## ConcurrentHashMap能保证符合操作的原子性吗？

ConcurrentHashMap 是线程安全的，意味着它可以保证多个线程同时对它进行读写操作时，不会出现数据不一致的情况，也不会导致 JDK1.7 及之前版本的 HashMap 多线程操作导致死循环问题。但是，这并不意味着它可以保证所有的复合操作都是原子性的，一定不要搞混了！

复合操作是指由多个基本操作(如put、get、remove、containsKey等)组成的操作，例如先判断某个键是否存在containsKey(key)，然后根据结果进行插入或更新put(key, value)。这种操作在执行过程中可能会被其他线程打断，导致结果不符合预期。

例如，有两个线程 A 和 B 同时对 ConcurrentHashMap 进行复合操作，如下：

```arduino
// 线程 A
if (!map.containsKey(key)) {
map.put(key, value);
}
// 线程 B
if (!map.containsKey(key)) {
map.put(key, anotherValue);
}
```

如果线程 A 和 B 的执行顺序是这样：

1. 线程 A 判断 map 中不存在 key
2. 线程 B 判断 map 中不存在 key
3. 线程 B 将 (key, anotherValue) 插入 map
4. 线程 A 将 (key, value) 插入 map

那么最终的结果是 (key, value)，而不是预期的 (key, anotherValue)。这就是复合操作的非原子性导致的问题。

**那如何保证 ConcurrentHashMap 复合操作的原子性呢？**

ConcurrentHashMap 提供了一些原子性的复合操作，如 putIfAbsent、compute、computeIfAbsent 、computeIfPresent、merge等。这些方法都可以接受一个函数作为参数，根据给定的 key 和 value 来计算一个新的 value，并且将其更新到 map 中。

上面的代码可以改写为：

```arduino
// 线程 A
map.putIfAbsent(key, value);
// 线程 B
map.putIfAbsent(key, anotherValue);
```

或者：

```arduino
// 线程 A
map.computeIfAbsent(key, k -> value);
// 线程 B
map.computeIfAbsent(key, k -> anotherValue);
```

虽然这种情况也能加锁同步，但不建议使用加锁的同步机制，这违背了使用 ConcurrentHashMap 的初衷。在使用 ConcurrentHashMap 的时候，尽量使用这些原子性的复合操作方法来保证原子性。



# 参考

[深入理解Java中的ConcurrentHashMap：原理与实践](https://juejin.cn/post/7360269869399605302?searchId=202407010937301909B67C2269C7FD2D8F)

[ConcurrentHashMap](https://juejin.cn/post/7351682240747749413?searchId=202407010937301909B67C2269C7FD2D8F)