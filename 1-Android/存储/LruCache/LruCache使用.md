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



# 二、原理分析



# 相关问题

> 这个模块收集和记录知识点可能涉及到的一些问题，比如在做笔记时的一些疑问，或者工作中常见的问题，或者面试相关的问题等。

<font color='orange'>Q：LruCache 的 Key 值用什么合适？</font>



<font color='orange'>Q：</font>

# 总结

1、



# 【精益求精】我还能做（补充）些什么？

> 这个模块主要是探索下是否有更多关联的知识点，或者当前知识点是否还有拓展或需要深入的点？

比如可以从下面几个方向进行思考：

1、相关技术

比如技术比较，是否有新的技术来替代等。

横向、纵向（原理）去探索

2、此技术在哪些项目或者三方库中用到了？

3、

# 脑图



# 参考

1、[彻底解析Android缓存机制——LruCache](https://www.jianshu.com/p/b49a111147ee)