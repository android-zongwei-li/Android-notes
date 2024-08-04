> version：2021/4/2
>
> review：2021/4/2
>



## 一、Java集合框架图

- List接口存储一组不唯一，有序（插入顺序）的对象。
- Set接口存储一组唯一，无序的对象。

## 二、对比

HashMap是非synchronized的，性能更好，HashMap可以接受为null的key-value。

HashTable是线程安全的，比HashMap慢，HashTable不接受为null的key-value。

ConcurrentHashMap线程安全，性能好，因为使用 cas 加锁 和 数组元素加锁。而不是整个方法。ConcurrentHashMap不接受为null的key-value

# HashTable方法

在方法前面加上了 synchronized 锁，这样实现了线程安全，但是性能不好。

```kotlin
    public synchronized V get(Object key) {
        HashtableEntry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (HashtableEntry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }

public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        HashtableEntry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        HashtableEntry<K,V> entry = (HashtableEntry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```



# 问题

HashMap与HashTable对比

HashMap与ConcurrentHashMap对比

Hashtable为什么是线程安全的

Hashtable、ConcurrentHashMap区别和优缺点

| Map               | 线程安全                                                | key-value                   |      |
| ----------------- | ------------------------------------------------------- | --------------------------- | ---- |
| HashMap           | 线程不安全。                                            | 可以为null，key只能一个null |      |
| ConcurrentHashMap | 线程安全。1.8cas+synchronized，1.7ReentrantLock。性能好 | 都不可以为null              |      |
| HashTable         | 线程安全。每个方法上加synchronized。性能不好            | 都不可以为null              |      |
|                   |                                                         |                             |      |



# 参考

