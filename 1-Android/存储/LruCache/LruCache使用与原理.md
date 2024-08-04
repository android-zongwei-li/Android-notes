> version：2023/05/13
>
> review：



[TOC]



# 前言

## 关键词 & 概念

> 本文会涉及到的一些概念，比如某个知识或技术点。

LruCache，

## 前置知识

> 若提前掌握了前置知识，会更容易掌握当前要介绍的知识或技术点。

LinkedHashMap

# 一、Android中的缓存策略

> Android的三级缓存，主要的就是内存缓存和硬盘缓存。这两种缓存机制的实现都应用到了LruCache算法。

一般来说，缓存策略主要包含缓存的添加、获取和删除这三类操作。如何添加和获取缓存这个比较好理解，那么为什么还要删除缓存呢？这是因为不管是内存缓存还是硬盘缓存，它们的缓存大小都是有限的。当缓存满了之后，再想添加缓存，这个时候就需要删除一些旧的缓存并添加新的缓存。

因此LRU(Least Recently Used)缓存算法便应运而生，LRU是近期最少使用的算法，它的核心思想是当缓存满时，会优先淘汰那些近期最少使用的缓存对象。采用LRU算法的缓存有两种：LruCache和DisLruCache，分别用于实现内存缓存和硬盘缓存，其核心思想都是LRU缓存算法。

# 二、LruCache的使用

LruCache  是 Android 3.1 所提供的一个缓存类，所以在Android中可以直接使用 LruCache 实现内存缓存。DisLruCache目前还不是Android SDK 的一部分，但Android官方文档推荐使用该算法来实现硬盘缓存。

## 1、LruCache 的介绍

LruCache 是用来实现内存缓存的一个类。LruCache是个泛型类，主要算法原理是把最近使用的对象用强引用（即我们平常使用的对象引用方式）存储在 LinkedHashMap 中。当缓存满时，把最近最少使用的对象从内存中移除，并提供了get和put方法来完成缓存的获取和添加操作。

## 2、LruCache 的使用

LruCache 的使用非常简单，下面以图片缓存为例。

### 示例1：

点击按钮时加载一张图片，先从缓存中获取，缓存没有时，从文件获取。

```java
class MainActivity : AppCompatActivity() {
    companion object {
        private const val TAG = "MainActivity"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.btnStartActivityCoroutines.setOnClickListener {
            startActivity(Intent(this, CoroutinesActivity::class.java))
        }

        binding.btnLoadBitmap.setOnClickListener {
            val startTime = System.currentTimeMillis()
            val bitmap = ImageLoader.loadDefaultBitmap(this)
            binding.ivPic.setImageBitmap(bitmap)
            val spentTime = System.currentTimeMillis() - startTime
            Log.i(TAG, "spentTime = $spentTime ms")
        }
    }

    private class ImageLoader {
        companion object {
            private const val TAG = "ImageLoader"

            var lruCacheManager: LruCacheManager? = null

            fun loadDefaultBitmap(context: Context): Bitmap {
                var result: Bitmap? = null
                if (lruCacheManager == null) {
                    lruCacheManager = LruCacheManager(context)
                }

                result = lruCacheManager?.getBitmap("defaultBitmap")
                Log.i(TAG, "loaded from LruCache, result = $result")

                if (result == null) {
                    result = context.resources.getDrawable(R.mipmap.ic_launcher).toBitmap()
                    Log.i(TAG, "load from file, result = $result")
                }

                return result.apply {
                    lruCacheManager?.putCache("defaultBitmap", result)
                }
            }
        }
    }
}
```

LruCache 工具类

```java
class LruCacheManager(context: Context) {
    private val TAG = "LruCacheManager"

    var mMemoryCache: LruCache<String, Bitmap>? = null

    init {
        initMemoryCache(context)
    }

    private fun initMemoryCache(context: Context) {
        val am = context.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
        // 1、设置LruCache缓存的大小，一般为当前进程可用容量的1/8。
        val availMenInBytes = am.memoryClass * 1024 * 1024 / 8
        mMemoryCache = object : LruCache<String, Bitmap>(availMenInBytes) {
            // 2、重写sizeOf方法，计算出要缓存的每张图片的大小。
            override fun sizeOf(key: String?, bitmap: Bitmap?): Int {
                return getBitmapSize(bitmap)
            }
        }
    }

    // 根据android版本来计算bitmap的实际占用内存
    private fun getBitmapSize(bitmap: Bitmap?): Int {
        if (bitmap == null) {
            return 0
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            //API 19
            return bitmap.allocationByteCount
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB_MR1) {
            //API 12
            return bitmap.byteCount
        }
        // 在低版本中用一行的字节x高度
        return bitmap.rowBytes * bitmap.height
    }

    fun getBitmap(key: String?): Bitmap? {
        return mMemoryCache!![key]
    }

    fun putCache(key: String?, bitmap: Bitmap?) {
        if (getBitmap(key) == null) {
            if (key != null && bitmap != null) {
                try {
                    mMemoryCache!!.put(key, bitmap)
                } catch (e: Exception) {
                    Log.e(TAG, e.toString())
                }
            }
        }
    }
}
```

1、设置LruCache缓存的大小，一般为当前进程可用容量的1/8。
2、重写sizeOf方法，计算出要缓存的每张图片的大小。

**注意：**缓存的总容量和每个缓存对象的大小所用单位要一致。

`示例1` 运行结果：

```java
2023-05-15 00:18:17.385 23864-23864/com.lizw.core_apis I/ImageLoader: loaded from LruCache, result = null
2023-05-15 00:18:17.394 23864-23864/com.lizw.core_apis I/ImageLoader: load from file, result = android.graphics.Bitmap@fda4aac
2023-05-15 00:18:17.395 23864-23864/com.lizw.core_apis I/MainActivity: spentTime = 11 ms
2023-05-15 00:18:22.808 23864-23864/com.lizw.core_apis I/ImageLoader: loaded from LruCache, result = android.graphics.Bitmap@fda4aac
2023-05-15 00:18:22.808 23864-23864/com.lizw.core_apis I/MainActivity: spentTime = 1 ms
```

可以看到，我们首次点击后，通过文件加载了一张图片并显示出来，然后将图片通过LruCache缓存起来，之后通过LruCache获取。

#### 性能比较

加载同一种图片，从文件加载（不使用缓存）和从内存加载（使用缓存）时间相差有10ms，10倍多



# 二、LruCache的实现原理

LruCache的核心思想就是要维护一个缓存对象列表，其中对象列表的排列方式是按照访问顺序实现的，即一直没访问的对象，将放在队尾，即将被淘汰。而最近访问的对象将放在队头，最后被淘汰。

如下图所示：

![img](images/LruCache%E4%BD%BF%E7%94%A8%E4%B8%8E%E5%8E%9F%E7%90%86/p1.webp)

这个队列是由LinkedHashMap来维护的，LinkedHashMap是由数组+双向链表的数据结构来实现的。其中双向链表的结构可以实现访问顺序和插入顺序，使得LinkedHashMap中的<key,value>对按照一定顺序排列起来。

通过下面构造函数来指定LinkedHashMap中双向链表的结构是访问顺序还是插入顺序。

```java
public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

其中accessOrder设置为true则为访问顺序，为false，则为插入顺序。

以具体例子解释：当设置为true时

```swift
public static final void main(String[] args) {
        LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(0, 0.75f, true);
        map.put(0, 0);
        map.put(1, 1);
        map.put(2, 2);
        map.put(3, 3);
        map.put(4, 4);
        map.put(5, 5);
        map.put(6, 6);
        map.get(1);
        map.get(2);

        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
}
```

输出结果：

> 0:0
> 3:3
> 4:4
> 5:5
> 6:6
> 1:1
> 2:2

即最近访问的最后输出，那么这就正好满足了 LRU 缓存算法的思想。**可见LruCache巧妙实现，就是利用了LinkedHashMap的这种数据结构。**

下面我们在LruCache源码中具体看看，怎么应用LinkedHashMap来实现缓存的添加，获取和删除的。

```cpp
 public LruCache(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
    }
```

从LruCache的构造函数中可以看到正是用了LinkedHashMap的访问顺序。

**put()方法**

```csharp
public final V put(K key, V value) {
         // 不可为空，否则抛出异常
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }
        V previous;
        synchronized (this) {
            // 插入的缓存对象值加1
            putCount++;
            // 增加已有缓存的大小
            size += safeSizeOf(key, value);
            // 向map中加入缓存对象
            previous = map.put(key, value);
            // 如果已有缓存对象，则缓存大小恢复到之前
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }
        // entryRemoved()是个空方法，可以自行实现
        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }
        // 调整缓存大小(关键方法)
        trimToSize(maxSize);
        return previous;
    }
```

可以看到put()方法并没有什么难点，重要的就是在添加过缓存对象后，调用 trimToSize()方法，来判断缓存是否已满，如果满了就要删除近期最少使用的算法。

**trimToSize()方法**

```csharp
 public void trimToSize(int maxSize) {
        //死循环
        while (true) {
            K key;
            V value;
            synchronized (this) {
                //如果map为空并且缓存size不等于0或者缓存size小于0，抛出异常
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }
                //如果缓存大小size小于最大缓存，或者map为空，不需要再删除缓存对象，跳出循环
                if (size <= maxSize || map.isEmpty()) {
                    break;
                }
                //迭代器获取第一个对象，即队尾的元素，近期最少访问的元素
                Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
                key = toEvict.getKey();
                value = toEvict.getValue();
                //删除该对象，并更新缓存大小
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }
            entryRemoved(true, key, value, null);
        }
    }
```

trimToSize()方法不断地删除LinkedHashMap中队尾的元素，即近期最少访问的，直到缓存大小小于最大值。

当调用LruCache的get()方法获取集合中的缓存对象时，就代表访问了一次该元素，将会更新队列，保持整个队列是按照访问顺序排序。这个更新过程就是在LinkedHashMap中的get()方法中完成的。

**LruCache 的 get()方法**

```kotlin
public final V get(K key) {
        //key为空抛出异常
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V mapValue;
        synchronized (this) {
            //获取对应的缓存对象
            //get()方法会实现将访问的元素更新到队列头部的功能
            mapValue = map.get(key);
            if (mapValue != null) {
                hitCount++;
                return mapValue;
            }
            missCount++;
        }
```

其中LinkedHashMap的get()方法如下：

```kotlin
public V get(Object key) {
        LinkedHashMapEntry<K,V> e = (LinkedHashMapEntry<K,V>)getEntry(key);
        if (e == null)
            return null;
        // 实现排序的关键方法
        e.recordAccess(this);
        return e.value;
    }

void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            // 判断是否是访问排序
            if (lm.accessOrder) {
                lm.modCount++;
                // 删除此元素
                remove();
                // 将此元素移动到队列的头部
                addBefore(lm.header);
            }
        }
```

**由此可见LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。当调用put()方法时，就会在集合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队头。**

以上便是LruCache实现的原理，理解了LinkedHashMap的数据结构就能理解整个原理。如果不懂，可以先看看LinkedHashMap的具体实现。

# 相关问题

<font color='orange'>Q：LruCache 的 Key 值用什么合适？</font>



<font color='orange'>Q：LruCache 的原理</font>



<font color='orange'>Q：LruCache怎么实现？为什么是o(1)的时间复杂度？</font>



# 总结



# 参考

[彻底解析Android缓存机制——LruCache](https://www.jianshu.com/p/b49a111147ee)