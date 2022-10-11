> 知识路径：Android > Java > 集合
>
> version：2021/4/2
>
> review：2021/4/2
>
> 掌握程度：入门

## 一、预备知识

- 掌握HashMap源码；
- 掌握HashTable源码。

## 一、Java集合框架图

![image-20210402111534623](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210402111534623.png)

- List接口存储一组不唯一，有序（插入顺序）的对象。
- Set接口存储一组唯一，无序的对象。

## 二、HashMap与HashTable对比

HashMap是非synchronized的，性能更好，HashMap可以接受为null的key-value。

HashTable是线程安全的，比HashMap慢，HashTable不接受为null的key-value。

1、HashMap.java

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
}
```

2、HashTable.java

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

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
}
```





参考：

1、Android资料 >《Android核心知识点笔记V2020.03.30》

