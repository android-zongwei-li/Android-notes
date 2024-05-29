# 前言

- `Glide`，该功能非常强大 `Android`  图片加载开源框架 相信大家并不陌生

  ![img](https:////upload-images.jianshu.io/upload_images/944365-e2ba626030d121a5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1065/format/webp)

  Github截图

  

- 正由于他的功能强大，所以它的源码非常复杂，这导致很多人望而却步

- 本人尝试将 `Glide` 的功能进行分解，并单独针对每个功能进行源码分析，从而降低`Glide`源码的复杂度。

> 接下来，我将推出一系列关于 `Glide`的功能源码分析，有兴趣可以继续关注

- 今天，我将主要针对**`Glide`的图片缓存功能** 进行流程 & 源码分析 ，希望你们会喜欢。

> 由于文章较长，希望读者先收藏 & 预留足够时间进行查看。

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-4ea27e4368926a3f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

目录

------

# 1. Glide缓存机制简介

### 1.1 缓存的图片资源

`Glide` 需要缓存的 图片资源 分为两类：

- 原始图片（`Source`） ：即图片源的图片初始大小 & 分辨率
- 转换后的图片(`Result`) ：经过 尺寸缩放 和 大小压缩等处理后的图片

> 当使用 `Glide`加载图片时，Glide默认 根据 `View`视图对图片进行压缩 & 转换，而不显示原始图（这也是`Glide`加载速度高于`Picasso`的原因）

### 1.2 缓存机制设计

- `Glide`的缓存功能设计成 **二级缓存**：内存缓存 & 硬盘缓存

> 并不是三级缓存，**因为 从网络加载 不属于缓存**

- 缓存读取顺序：内存缓存 --> 磁盘缓存 --> 网络

> 1. 内存缓存 默认开启
> 2. `Glide`中，内存缓存 & 磁盘缓存相互不影响，独立配置

- 二级缓存的作用不同：

  1. 内存缓存：防止应用 重复将图片数据 读取到内存当中

  > 只 缓存转换过后的图片

  1. 硬盘缓存：防止应用 重复从网络或其他地方重复下载和读取数据

  > 可缓存原始图片 & 缓存转换过后的图片，用户自行设置

`Glide`的缓存机制使得 `Glide`具备非常好的图片缓存效果，从而使得具备较高的图片加载效率。

> 如，在 `RecyclerView` 上下滑动，而`RecyclerView`中只要是`Glide`加载过的图片，都可以直接从内存中读取 & 展示，从而不需要重复从 网络或硬盘上读取，提高图片加载效率。

# 2. Glide  缓存功能介绍

- `Glide` 的缓存功能分为：内存缓存 & 磁盘缓存
- 具体介绍如下

### 2.1 内存缓存

- 作用：防止应用 重复将图片数据 读取到内存当中

> 只 缓存转换过后的图片，而并非原始图片

- 具体使用
   默认情况下，`Glide`自动开启 内存缓存



```csharp
// 默认开启内存缓存，用户不需要作任何设置
Glide.with(this)
     .load(url)
     .into(imageView);

// 可通过 API 禁用 内存缓存功能
Glide.with(this)
     .load(url)
     .skipMemoryCache(true) // 禁用 内存缓存
     .into(imageView);
```

- 实现原理
   `Glide`的内存缓存实现是基于：`LruCache` 算法（`Least Recently Used`） & 弱引用机制

> 1. `LruCache`算法原理：将 最近使用的对象 **用强引用的方式** 存储在`LinkedHashMap`中 ；当缓存满时 ，**将最近最少使用的对象从内存中移除**
> 2. 弱引用：弱引用的对象具备更短生命周期，因为 **当`JVM`进行垃圾回收时，一旦发现弱引用对象，都会进行回收（无论内存充足否）

### 2.2  磁盘缓存

- 作用：防止应用 重复从网络或其他地方重复下载和读取数据

> 可缓存原始图片 & 缓存转换过后的图片，用户自行设置

- 具体使用

```csharp
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);

// 缓存参数说明
// DiskCacheStrategy.NONE：不缓存任何图片，即禁用磁盘缓存
// DiskCacheStrategy.ALL ：缓存原始图片 & 转换后的图片
// DiskCacheStrategy.SOURCE：只缓存原始图片（原来的全分辨率的图像，即不缓存转换后的图片）
// DiskCacheStrategy.RESULT：（默认）只缓存转换后的图片（即最终的图像：降低分辨率后 / 或者转换后 ，不缓存原始图片
```

- 实现原理
   使用`Glide` 自定义的`DiskLruCache`算法

> 1. 该算法基于 `Lru` 算法中的`DiskLruCache`算法，具体应用在磁盘缓存的需求场景中
> 2. 该算法被封装到`Glide`自定义的工具类中（该工具类基于`Android` 提供的`DiskLruCache`工具类

# 3. Glide 缓存流程 解析

- `Glide`整个缓存流程 从 **加载图片请求** 开始，其中过程 有本文最关注的 内存缓存的读取 & 写入、磁盘缓存的读取 & 写入
- 具体如下

![img](https:////upload-images.jianshu.io/upload_images/944365-f3c273d8a5d4389d.png?imageMogr2/auto-orient/strip|imageView2/2/w/595/format/webp)

示意图

下面，我将根据 `Glide`缓存流程中的每个步骤 进行源码分析。

------

# 4. 缓存流程 源码分析

### 步骤1：生成缓存Key

- `Glide` 实现内存 & 磁盘缓存 是根据 **图片的缓存Key** 进行唯一标识

> 即根据 图片的缓存Key 去缓存区找 对应的缓存图片

- 生成缓存 `Key` 的代码发生在`Engine`类的 `load()`中

> 该代码在上一篇文章：[Android：手把手带你深入图片加载库Glide源码分析](https://www.jianshu.com/p/216df89bf59c)当中已分析过，只是当时忽略了缓存相关的内容，现在仅贴出缓存相关的代码



```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();

        final String id = fetcher.getId();
        // 获得了一个id字符串，即需加载图片的唯一标识
        // 如，若图片的来源是网络，那么该id = 这张图片的url地址

        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),transcoder, loadProvider.getSourceEncoder());
        // Glide的缓存Key生成规则复杂：根据10多个参数生成
        // 将该id 和 signature、width、height等10个参数一起传入到缓存Key的工厂方法里，最终创建出一个EngineKey对象
        // 创建原理：通过重写equals() 和 hashCode()，保证只有传入EngineKey的所有参数都相同情况下才认为是同一个EngineKey对象
        // 该EngineKey 即Glide中图片的缓存Key

        ...
}
```

至此，`Glide`的图片缓存 `Key` 生成完毕。

------

### 步骤2：创建缓存对象 `LruResourceCache`

- `LruResourceCache`对象是在创建 `Glide` 对象时创建的
- 而 创建 `Glide` 对象则是在上篇文章[Android：手把手带你深入图片加载库Glide源码分析](https://www.jianshu.com/p/216df89bf59c)讲解 `Glide` 图片加载功能时 第2步`load()`中`loadGeneric()` 创建 `ModelLoader`对象时创建的
- 请看源码分析



```java
<-- 第2步load（）中的loadGeneric（）-->
    private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {
        ...

        ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
        // 创建第1个ModelLoader对象；作用：加载图片
        // Glide会根据load()方法传入不同类型参数，得到不同的ModelLoader对象
        // 此处传入参数是String.class，因此得到的是StreamStringLoader对象（实现了ModelLoader接口）
        // Glide.buildStreamModelLoader（）分析 ->>分析1


<--分析1：Glide.buildStreamModelLoader（） -->
public class Glide {

    public static <T, Y> ModelLoader<T, Y> buildModelLoader(Class<T> modelClass, Class<Y> resourceClass,
            Context context) {
         if (modelClass == null) {
            if (Log.isLoggable(TAG, Log.DEBUG)) {
                Log.d(TAG, "Unable to load null model, setting placeholder only");
            }
            return null;
        }
        return Glide.get(context).getLoaderFactory().buildModelLoader(modelClass, resourceClass);
        // 创建ModelLoader对象时，调用Glide.get() 创建Glide对象-->分析2
    }

<--分析2：Glide.get() -->
// 作用：采用单例模式创建Glide对象
    public static Glide get(Context context) {

        // 实现单例功能
        if (glide == null) {
            synchronized (Glide.class) {
                if (glide == null) {
                    Context applicationContext = context.getApplicationContext();
                    List<GlideModule> modules = new ManifestParser(applicationContext).parse();
                    GlideBuilder builder = new GlideBuilder(applicationContext);
                    for (GlideModule module : modules) {
                        module.applyOptions(applicationContext, builder);
                    }
                    glide = builder.createGlide();
                    // 通过建造者模式创建Glide对象 ->>分析3
                    for (GlideModule module : modules) {
                        module.registerComponents(applicationContext, glide);
                    }
                }
            }
        }
        return glide;
    }
}
       
<--分析3：builder.createGlide() -->
// 作用：创建Glide对象
public class GlideBuilder {
    ...

    Glide createGlide() {
        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        if (bitmapPool == null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
                int size = calculator.getBitmapPoolSize();
                bitmapPool = new LruBitmapPool(size);
            } else {
                bitmapPool = new BitmapPoolAdapter();
            }
        }

        if (memoryCache == null) {
            memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
            // 创建一个LruResourceCache对象 并 赋值到memoryCache对象
            // 该LruResourceCache对象 = Glide实现内存缓存的LruCache对象

        }
        
        return new Glide(engine, memoryCache, bitmapPool, context, decodeFormat);
    }
}
```

至此，创建好了缓存对象`LruResourceCache`

### 步骤3：从 内存缓存 中获取缓存图片

- `Glide` 在图片加载前就会从 内存缓存 中获取缓存图片
- 读取内存缓存代码 是在`Engine`类的`load()`中

> 即上面讲解的生成缓存 `Key` 的地方

- 源码分析



```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    ...    

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();

        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());
         // 上面讲解的生成图片缓存Key


        EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
        // 调用loadFromCache()获取内存缓存中的缓存图片

        if (cached != null) {
            cb.onResourceReady(cached);
        }
        // 若获取到，就直接调用cb.onResourceReady()进行回调

        EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
        if (active != null) {
            cb.onResourceReady(active);
        }
        // 若没获取到，就继续调用loadFromActiveResources()获取缓存图片
        // 获取到也直接回调

        // 若上述两个方法都没有获取到缓存图片，就开启一个新的线程准备加载图片
        // 即从上文提到的 Glide最基础功能：图片加载
        EngineJob current = jobs.get(key);
            return new LoadStatus(cb, current);
        }

        EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
        DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
                transcoder, diskCacheProvider, diskCacheStrategy, priority);
        EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
        jobs.put(key, engineJob);
        engineJob.addCallback(cb);
        engineJob.start(runnable);

        return new LoadStatus(cb, engineJob);
    }

    ...
}
```

即：

- `Glide` 将 内存缓存 划分为两块：一块使用了`LruCache`算法 机制；另一块使用了弱引用 机制
- 当 获取 内存缓存 时，会通过两个方法分别从上述两块区域进行缓存获取

1. `loadFromCache()`：从 使用了 `LruCache`算法机制的内存缓存获取 缓存
2. `loadFromActiveResources()`：从 使用了 弱引用机制的内存缓存获取 缓存

源码分析如下：



```php
// 这2个方法属于  Engine 类
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    private final MemoryCache cache;
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...

<-- 方法1：loadFromCache() -->
// 原理：使用了 LruCache算法
    private EngineResource<?> loadFromCache(Key key, boolean isMemoryCacheable) {
        if (!isMemoryCacheable) {
            return null;
        // 若isMemoryCacheable = false就返回null，即内存缓存被禁用
        // 即 内存缓存是否禁用的API skipMemoryCache() - 请回看内存缓存的具体使用
        // 若设置skipMemoryCache(true)，此处的isMemoryCacheable就等于false，最终返回Null，表示内存缓存已被禁用
        }

        EngineResource<?> cached = getEngineResourceFromCache(key);
        // 获取图片缓存 ->>分析4

        // 从分析4回来看这里：
        if (cached != null) {
            cached.acquire();
            activeResources.put(key, new ResourceWeakReference(key, cached, getReferenceQueue()));
            // 将获取到的缓存图片存储到activeResources当中
            // activeResources = 一个弱引用的HashMap：用于缓存正在使用中的图片
            // 好处：保护这些图片不会被LruCache算法回收掉。 ->>方法2

        }
        return cached;
    }

<<- 分析4：getEngineResourceFromCache（） ->>
// 作用：获取图片缓存
// 具体过程：根据缓存Key 从cache中 取值 
// 注：此处的cache对象 = 在构建Glide对象时创建的LruResourceCache对象，即说明使用的是LruCache算法
    private EngineResource<?> getEngineResourceFromCache(Key key) {
        Resource<?> cached = cache.remove(key);
        // 当从LruResourceCache中获取到缓存图片后，会将它从缓存中移除->>回到方法1原处
        final EngineResource result;
        if (cached == null) {
            result = null;
        } else if (cached instanceof EngineResource) {
            result = (EngineResource) cached;
        } else {
            result = new EngineResource(cached, true /*isCacheable*/);
        }
        return result;
    }

<-- 方法2：loadFromActiveResources() -->
// 原理：使用了 弱引用机制
// 具体过程：当在方法1中无法获取内存缓存中的缓存图片时，就会从activeResources中取值
// activeResources = 一个弱引用的HashMap：用于缓存正在使用中的图片
    private EngineResource<?> loadFromActiveResources(Key key, boolean isMemoryCacheable) {
        if (!isMemoryCacheable) {
            return null;
        }
        EngineResource<?> active = null;
        WeakReference<EngineResource<?>> activeRef = activeResources.get(key);
        if (activeRef != null) {
            active = activeRef.get();
            if (active != null) {
                active.acquire();
            } else {
                activeResources.remove(key);
            }
        }
        return active;
    }

    ...
}
```

若上述两个方法都没获取到缓存图片时（即内存缓存里没有该图片的缓存），就开启新线程加载图片。

------

- 至此，获取内存缓存 的步骤讲解完毕。
- 总结

![img](https:////upload-images.jianshu.io/upload_images/944365-6c96c5ce38a8814c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

### 步骤4：开启 加载图片 线程

- 若无法从 内存缓存 里 获得缓存的图片，`Glide`就会开启 加载图片的线程
- 但在该线程开启后，`Glide`并不会马上去网络 加载图片，**而是采取采用`Glide`的第2级缓存：磁盘缓存** 去获取缓存图片

> 从 上篇文章[Android：手把手带你深入图片加载库Glide源码分析](https://www.jianshu.com/p/216df89bf59c)：`Glide` 在第3步 `into()`中开启图片线程 `run()`里的 `decode()`开始（上文的分析13）



```kotlin
private Resource<?> decode() throws Exception {

// 在执行 加载图片 线程时（即加载图片时），分两种情况：
// 情况1：从磁盘缓存当中读取图片（默认情况下Glide会优先从缓存当中读取，没有才会去网络源读取图片）
// 情况2：不从磁盘缓存中读取图片

// 情况1：从磁盘缓存中读取缓存图片
    if (isDecodingFromCache()) {
    // 取决于在使用API时是否开启，若采用DiskCacheStrategy.NONE，即不缓存任何图片，即禁用磁盘缓存
        return decodeFromCache();
        // 读取磁盘缓存的入口就是这里，此处主要讲解 ->>直接看步骤4的分析9
    } else {

    // 情况2：不从磁盘缓存中读取图片        
    // 即上文讨论的从网络读取图片，此处不作过多描述
        return decodeFromSource();
    }
}
```

------

### 步骤5：从 磁盘缓存 中获取缓存图片

若无法从 内存缓存 里 获得缓存的图片，`Glide`就会采用第2级缓存：磁盘缓存 去获取缓存图片



```kotlin
<--分析9：decodeFromCache()  -->
private Resource<?> decodeFromCache() throws Exception {
    Resource<?> result = null;

        result = decodeJob.decodeResultFromCache();
        // 获取磁盘缓存时，会先获取 转换过后图片 的缓存
        // 即在使用磁盘缓存时设置的模式，如果设置成DiskCacheStrategy.RESULT 或DiskCacheStrategy.ALL就会有该缓存
        // 下面来分析decodeResultFromCache() ->>分析10

    }
    if (result == null) {
        result = decodeJob.decodeSourceFromCache();
        // 如果获取不到 转换过后图片 的缓存，就获取 原始图片 的缓存
        // 即在使用磁盘缓存时设置的模式，如果设置成DiskCacheStrategy.SOURCE 或DiskCacheStrategy.ALL就会有该缓存
        // 下面来分析decodeSourceFromCache() ->>分析12
    }
    return result;
}


<--分析10：decodeFromCache()  -->
public Resource<Z> decodeResultFromCache() throws Exception {
    if (!diskCacheStrategy.cacheResult()) {
        return null;
    }

    Resource<T> transformed = loadFromCache(resultKey);
    // 1. 根据完整的缓存Key（由10个参数共同组成，包括width、height等）获取缓存图片
    // ->>分析11

    Resource<Z> result = transcode(transformed);
    return result;
    // 2. 直接将获取到的图片 数据解码 并 返回
    // 因为图片已经转换过了，所以不需要再作处理
    // 回到分析9原处
}


<--分析11：decodeFromCache()  -->
private Resource<T> loadFromCache(Key key) throws IOException {
    File cacheFile = diskCacheProvider.getDiskCache().get(key);

    // 1. 调用getDiskCache()获取Glide自己编写的DiskLruCache工具类实例
    // 2. 调用上述实例的get() 并 传入完整的缓存Key，最终得到硬盘缓存的文件

    if (cacheFile == null) {
        return null;
        // 如果文件为空就返回null
    }
    Resource<T> result = null;
    try {
        result = loadProvider.getCacheDecoder().decode(cacheFile, width, height);
            } finally {
        if (result == null) {
            diskCacheProvider.getDiskCache().delete(key);
        }
    }
    return result;
    // 如果文件不为空，则将它解码成Resource对象后返回
    // 回到分析10原处
}



<--分析12：decodeFromCache()  -->
public Resource<Z> decodeSourceFromCache() throws Exception {
    if (!diskCacheStrategy.cacheSource()) {
        return null;
    }

    Resource<T> decoded = loadFromCache(resultKey.getOriginalKey());
    // 1. 根据缓存Key的OriginalKey来获取缓存图片
    // 相比完整的缓存Key，OriginalKey只使用了id和signature两个参数，而忽略了大部分的参数
    // 而signature参数大多数情况下用不到，所以基本是由id（也就是图片url）来决定的Original缓存Key
    // 关于loadFromCache（）同分析11，只是传入的缓存Key不一样

    return transformEncodeAndTranscode(decoded);
    // 2. 先将图片数据 转换 再 解码，最终返回
    
}
```

- 至此，硬盘缓存读取的源码分析完毕。
- 总结

![img](https:////upload-images.jianshu.io/upload_images/944365-d6db80468725f1d5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

## 步骤6：从网络获取 图片资源

- 在 `Glide`两级缓存机制里都没有该图片缓存时，只能去源头（如网络）去加载图片了
- 但从网络加载图片前，需要先获取该图片的网络资源

> 1. 此处先忽略该过程
>
> # 2. 若有兴趣的同学请看#该过程在请看文章

------

## 步骤7：写入 磁盘缓存

- `Glide`将图片写入 磁盘缓存的时机：获取图片资源后 、图片加载完成前

- 写入磁盘缓存又分为：将原始图片 写入 或 将转换后的图片写入磁盘缓存

> 从 上篇文章[Android：手把手带你深入图片加载库Glide源码分析](https://www.jianshu.com/p/216df89bf59c)：`Glide` 在第3步 `into（）`中执行图片线程 `run（）`里的 `decode（）`开始（上文的分析13）
>  此处重新贴出代码



```kotlin
private Resource<?> decode() throws Exception {

// 在执行 加载图片 线程时（即加载图片时），分两种情况：
// 情况1：从磁盘缓存当中读取图片（默认情况下Glide会优先从缓存当中读取，没有才会去网络源读取图片）
// 情况2：不从磁盘缓存中读取图片

// 情况1：从磁盘缓存中读取缓存图片
    if (isDecodingFromCache()) {
        return decodeFromCache();
        // 读取磁盘缓存的入口就是这里，上面已经讲解
    } else {

    // 情况2：不从磁盘缓存中读取图片
        // 即上文讨论的从网络读取图片，不采用缓存
        // 写入磁盘缓存就是在 此处 写入的 ->>分析13
        return decodeFromSource();
    }
}


<--分析13：decodeFromSource()  -->
public Resource<Z> decodeFromSource() throws Exception {
    Resource<T> decoded = decodeSource();
    // 解析图片
    // 写入原始图片 磁盘缓存的入口 ->>分析14

     // 从分析16回来看这里
    return transformEncodeAndTranscode(decoded);
    // 对图片进行转码
    // 写入 转换后图片 磁盘缓存的入口 ->>分析17
}


<--分析14：decodeSource()  -->
private Resource<T> decodeSource() throws Exception {
    Resource<T> decoded = null;
    try {

        final A data = fetcher.loadData(priority);
        // 读取图片数据
        if (isCancelled) {
            return null;
        }
        decoded = decodeFromSourceData(data);
        // 对图片进行解码 ->>分析15
    } finally {
        fetcher.cleanup();
    }
    return decoded;
}

<--分析15：decodeFromSourceData()  -->
private Resource<T> decodeFromSourceData(A data) throws IOException {
    final Resource<T> decoded;
    // 判断是否允许缓存原始图片
    // 即在使用 硬盘缓存API时，是否采用DiskCacheStrategy.ALL 或 DiskCacheStrategy.SOURCE
    if (diskCacheStrategy.cacheSource()) {
        decoded = cacheAndDecodeSourceData(data);
        // 若允许缓存原始图片，则调用cacheAndDecodeSourceData()进行原始图片的缓存 ->>分析16

    } else {
        long startTime = LogTime.getLogTime();
        decoded = loadProvider.getSourceDecoder().decode(data, width, height);
    }
    return decoded;
}

<--分析16：cacheAndDecodeSourceData   -->
private Resource<T> cacheAndDecodeSourceData(A data) throws IOException {
   
    ...
    diskCacheProvider.getDiskCache().put(resultKey.getOriginalKey(), writer);
    // 1. 调用getDiskCache()获取DiskLruCache实例
    // 2. 调用put()写入硬盘缓存
    // 注：原始图片的缓存Key是用的getOriginalKey()，即只有id & signature两个参数
    // 请回到分析13

}

<--分析17：transformEncodeAndTranscode（） -->
private Resource<Z> transformEncodeAndTranscode(Resource<T> decoded) {
    
    Resource<T> transformed = transform(decoded);
    // 1. 对图片进行转换

    writeTransformedToCache(transformed);
    // 2. 将 转换过后的图片 写入到硬盘缓存中 -->分析18

    Resource<Z> result = transcode(transformed);
    return result;
}

<-- 分析18：TransformedToCache（） -->
private void writeTransformedToCache(Resource<T> transformed) {
    if (transformed == null || !diskCacheStrategy.cacheResult()) {
        return;
    }

    diskCacheProvider.getDiskCache().put(resultKey, writer);
    // 1. 调用getDiskCache()获取DiskLruCache实例
    // 2. 调用put()写入硬盘缓存
    // 注：转换后图片的缓存Key是用的完整的resultKey，即含10多个参数
}
```

- 至此，硬盘缓存的写入分析完毕。
- 总结

![img](https:////upload-images.jianshu.io/upload_images/944365-c0e9ed77f3fe7773.png?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp)

示意图

------

## 步骤9：写入 内存缓存

- `Glide` 将图片写入 内存缓存的时机：图片加载完成后 、图片显示出来前
- 写入 内存缓存 的具体地方：上篇文章中当图片加载完成后，会在`EngineJob`中通过`Handler`发送一条消息将执行逻辑切回到主线程当中，从而执行`handleResultOnMainThread()`里



```java
class EngineJob implements EngineRunnable.EngineRunnableManager {

    private final EngineResourceFactory engineResourceFactory;
    ...

    private void handleResultOnMainThread() {
        ...

        // 关注1：写入 弱引用缓存
        engineResource = engineResourceFactory.build(resource, isCacheable);
        listener.onEngineJobComplete(key, engineResource);

        // 关注2：写入 LruCache算法的缓存
        engineResource.acquire();

        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                engineResource.acquire();
                cb.onResourceReady(engineResource);
            }
        }
        engineResource.release();
    }
```

写入 内存缓存分为：写入 弱引用缓存 & `LruCache`算法的缓存

> 1. 内存缓存分为：一块使用了  `LruCache`算法机制的区域 & 一块使用了 弱引用机制的缓存
> 2. 内存缓存只缓存 转换后的图片

### 关注1：写入 弱引用缓存



```java
class EngineJob implements EngineRunnable.EngineRunnableManager {

    private final EngineResourceFactory engineResourceFactory;
    ...

    private void handleResultOnMainThread() {
        ...

        // 写入 弱引用缓存
        engineResource = engineResourceFactory.build(resource, isCacheable);
        // 创建一个包含图片资源resource的EngineResource对象

        listener.onEngineJobComplete(key, engineResource);
        // 将上述创建的EngineResource对象传入到Engine.onEngineJobComplete() ->>分析6


        // 写入LruCache算法的缓存（先忽略）
        engineResource.acquire();

        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                engineResource.acquire();
                cb.onResourceReady(engineResource);
            }
        }
        engineResource.release();
    }


<<- 分析6：onEngineJobComplete()（） ->>
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    ...    

    @Override
    public void onEngineJobComplete(Key key, EngineResource<?> resource) {
        Util.assertMainThread();

        if (resource != null) {
            resource.setResourceListener(key, this);
            if (resource.isCacheable()) {
                activeResources.put(key, new ResourceWeakReference(key, resource, getReferenceQueue()));
                // 将 传进来的EngineResource对象 添加到activeResources（）中
                // 即写入了弱引用 内存缓存
            }
        }
        jobs.remove(key);
    }

    ...
}
```

### 关注2：写入 LruCache算法 缓存



```java
class EngineJob implements EngineRunnable.EngineRunnableManager {

    private final EngineResourceFactory engineResourceFactory;
    ...

    private void handleResultOnMainThread() {
        ...

        // 写入 弱引用缓存（忽略）
        engineResource = engineResourceFactory.build(resource, isCacheable);
        listener.onEngineJobComplete(key, engineResource);

        // 写入 LruCache算法的缓存
        engineResource.acquire();
        // 标记1

        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                engineResource.acquire();
                // 标记2
                cb.onResourceReady(engineResource);
            }
        }
        engineResource.release();
        // 标记3
    }
```

写入 `LruCache`算法 内存缓存的原理：包含图片资源`resource`的`EngineResource`对象的一个引用机制：

- 用 一个 `acquired` 变量 记录图片被引用的次数
- 加载图片时：调用 `acquire()` ，变量加1

> 上述代码的标记1、标记2 & 下面`acquire()`源码



```csharp
<-- 分析7：acquire() -->
    void acquire() {
        if (isRecycled) {
            throw new IllegalStateException("Cannot acquire a recycled resource");
        }
        if (!Looper.getMainLooper().equals(Looper.myLooper())) {
            throw new IllegalThreadStateException("Must call acquire on the main thread");
        }
        ++acquired;
        // 当调用acquire()时，acquired变量 +1
    }
```

- 不加载图片时，调用 `release()` 时，变量减1

> 上述代码的标记3 & 下面`release()`源码



```java
<-- 分析8：release()  -->
    void release() {
        if (acquired <= 0) {
            throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
        }
        if (!Looper.getMainLooper().equals(Looper.myLooper())) {
            throw new IllegalThreadStateException("Must call release on the main thread");
        }
        if (--acquired == 0) {
            listener.onResourceReleased(key, this);
            // 当调用acquire()时，acquired变量 -1
            // 若acquired变量 = 0，即说明图片已经不再被使用
            // 调用listener.onResourceReleased()释放资源
            // 该listener = Engine对象，Engine.onResourceReleased()->>分析9
        }
    }
}

<-- 分析9：onResourceReleased（）  -->

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    private final MemoryCache cache;
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...    

    @Override
    public void onResourceReleased(Key cacheKey, EngineResource resource) {
        Util.assertMainThread();
        activeResources.remove(cacheKey);
        // 步骤1：将缓存图片从activeResources弱引用缓存中移除

        if (resource.isCacheable()) {
            cache.put(cacheKey, resource);
            // 步骤2：将该图片缓存放在LruResourceCache缓存中
        } else {
            resourceRecycler.recycle(resource);
        }
    }

    ...
}
```

所以：

- 当 `acquired` 变量 >0 时，说明图片正在使用，即该图片缓存继续存放到`activeResources`弱引用缓存中
- 当 `acquired`变量 = 0，即说明图片已经不再被使用，就将该图片的缓存Key从 `activeResources`弱引用缓存中移除，并存放到`LruResourceCache`缓存中

------

至此，实现了：

- 正在使用中的图片 采用 弱引用 的内存缓存
- 不在使用中的图片 采用 `LruCache`算法 的内存缓存

### 总结

![img](https:////upload-images.jianshu.io/upload_images/944365-a71e7bdc643fe611.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

### 步骤10：显示图片

- 在将图片 写入 内存缓存 & 磁盘缓存后，图片最终显示出来
- 在下次加载时，将通过二级缓存 从而提高图片加载效率

至此，`Glide` 的图片缓存流程解析完毕。

------

# 5. 汇总

- 用一张图将整个`Glide` 的图片缓存流程 汇总

![img](https:////upload-images.jianshu.io/upload_images/944365-af49cbe681c8c2be.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- 关于内存缓存 的总结
  1. 读取 内存缓存 时，先从`LruCache`算法机制的内存缓存读取，再从弱引用机制的 内存缓存 读取
  2. 写入 内存缓存 时，先写入 弱引用机制 的内存缓存，等到图片不再被使用时，再写入到 `LruCache`算法机制的内存缓存
- 关于磁盘缓存 的总结
  1. 读取 磁盘缓存 时，先读取 转换后图片 的缓存，再读取 原始图片 的缓存

> 是否读取 取决于 `Glide`使用API的设置

1. 写入 磁盘缓存 时，先写入 原始图片 的内存缓存，再写入的内存缓存

> 是否写入 取决于 `Glide`使用API的设置

------

# 6. 额外注意：为什么你的Glide缓存功能不起作用？

### a. 背景

- `Glide`实现内存 & 磁盘缓存是根据 图片的缓存`Key`进行唯一标识
- 开发者为了降低成本 & 安全，往往会将图片存放在云服务器上

> 如 七牛云 等等。

- 为了保护 客户的图片资源，图片云服务器 会在图片`Url`地址的基础上再加一个token参数



```cpp
http://url.com/image.jpg?token=a6cvva6b02c670b0a
```

- `Glide`加载该图片时，会使用加了`token`参数的图片`Url`地址 作为
   `id`参数，从而生成 缓存Key

### b. 问题

- 作为身份认证的`token`参数可能会发生变化，并不是一成不变
- 若 `token`参数变了，则图片`Url`跟着变，则生成缓存key的所需id参数发生变化，即 **缓存Key也会跟着变化**
- 这导致同一张图片，但因为`token`参数变化，而导致缓存Key发生变化，从而使得 `Glide`的缓存功能失效

> 缓存Key发生变化，即同一个图片的当前缓存key 和 之前写入缓存的key不相同，这意味着 在读取缓存时 无法根据当前缓存key 找到之前的缓存，从而使得失效

### c. 解决方案

[Android图片加载的那些事：为什么你的Glide 缓存没有起作用？](https://www.jianshu.com/p/3c2a8471361e)

------

# 7. 总结

- 本文主要对**`Glide`的图片缓存功能** 进行流程 & 源码分析





# Android图片加载的那些事：为什么你的Glide缓存没有起作用？

# 前言

- `Glide`，该功能非常强大 `Android`  图片加载开源框架 相信大家并不陌生

  ![img](https:////upload-images.jianshu.io/upload_images/944365-e2ba626030d121a5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1065/format/webp)

  Github截图

  

- 正由于他的功能强大，所以它的源码非常复杂，这导致很多人望而却步

- 本人尝试将 `Glide` 的功能进行分解，并单独针对每个功能进行源码分析，从而降低`Glide`源码的复杂度。

> 接下来，我将推出一系列关于 `Glide`的功能源码分析，有兴趣可以继续关注

- 今天，我将主要讲解在使用`Glide`缓存功能时的问题：为什么Glide 的缓存无起作用，希望你们会喜欢。

> 请先阅读文章：[Android：深入剖析图片加载库Glide缓存功能(源码分析）](https://www.jianshu.com/p/e70d16275e9a)

------

# 1. 背景

- `Glide`实现内存 & 磁盘缓存是根据 图片的缓存`Key`进行唯一标识
- 开发者为了降低成本 & 安全，往往会将图片存放在云服务器上

> 如 七牛云 等等。

- 为了保护 客户的图片资源，图片云服务器 会在图片`Url`地址的基础上再加一个token参数



```cpp
http://url.com/image.jpg?token=a6cvva6b02c670b0a
```

- `Glide`加载该图片时，会使用加了`token`参数的图片`Url`地址 作为
   `id`参数，从而生成 缓存Key

------

# 2. 问题

- 作为身份认证的`token`参数可能会发生变化，并不是一成不变
- 若 `token`参数变了，则图片`Url`跟着变，则生成缓存key的所需id参数发生变化，即 **缓存Key也会跟着变化**
- 这导致同一张图片，但因为`token`参数变化，而导致缓存Key发生变化，从而使得 `Glide`的缓存功能失效

> 缓存Key发生变化，即同一个图片的当前缓存key 和 之前写入缓存的key不相同，这意味着 在读取缓存时 无法根据当前缓存key 找到之前的缓存，从而使得失效

------

# 3. 解决方案

### 3.1 原理

在 生成缓存`Key` 的id参数 前，将 带有`token`参数的图片`Url`地址 去掉 `token`参数，从而根据 初始的图片`Url`地址 生成缓存`Key`的id参数

> 实现了一个图片的缓存`Key`的id参数始终唯一 ，即等于 图片`Url`地址

### 3.2 储备知识：生成缓存Key的id参数的逻辑

生成缓存`Key`的`id`参数的逻辑为：直接将图片的 `URL` 地址作为缓存Key的`id`参数

> 回看文章：[Android：手把手带你深入图片加载库Glide源码分析](https://www.jianshu.com/p/216df89bf59c)生成缓存`Key`的代码



```java
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();

        final String id = fetcher.getId();
        // 获得了一个id字符串，即需加载图片的唯一标识
        // 如，若图片的来源是网络，那么该id = 这张图片的url地址
        // fetcher = HttpUrlFetcher的实例，即调用HttpUrlFetcher.getid（）->>分析19

        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),transcoder, loadProvider.getSourceEncoder());
        // 将该id 和 signature、width、height等10个参数一起传入到缓存Key的工厂方法里，最终创建出一个EngineKey对象
        // 创建原理：通过重写equals() 和 hashCode()，保证只有传入EngineKey的所有参数都相同情况下才认为是同一个EngineKey对象
       // 该EngineKey 即Glide中的缓存Key

        ...
}


<-- 分析19：getId() -->
public class HttpUrlFetcher implements DataFetcher<InputStream> {
    ...

    private final GlideUrl glideUrl;
    // GlideUrl = 在上篇文章讲解 图片加载 第2步load()中传入图片url地址时，Glide在内部把图片url地址包装成一个GlideUrl对象

    @Override
    public String getId() {
        return glideUrl.getCacheKey();
        // ->>分析20
}

<-- 分析20：getCacheKey()  -->

public class GlideUrl {

    private final URL url;
    private final String stringUrl;
    ...

    // GlideUrl构造函数
     public GlideUrl(URL url) {
        this(url, Headers.DEFAULT);
    }

    public GlideUrl(String url) {
        this(url, Headers.DEFAULT);
    }


    public String getCacheKey() {
        return stringUrl != null ? stringUrl : url.toString();
        // 在生成GlideUrl对象时：
        // 若传入的是URL字符串（即图片地址），就直接返回该字符串（大多数是这种情况）
        // 若传入的是URL对象，那么就返回这个对象toString()后的结果。

    }

    ...
}
```

### 3.3 实现方案

即 **我们只需重写`getCacheKey()` & 将 带有token参数的图片`Url`地址 去掉 token参数 即可。**



```dart
/**
  * 代码实现：创建一个GlideUrl类的子类 & 重写getCacheKey()
  **/
    // 1. 继承GlideUrl 
    public class mGlideUrl extends GlideUrl {

        private String mUrl;

        // 构造函数里 传入 带有token参数的图片Url地址 
        public MyGlideUrl(String url) {
            super(url);
            mUrl = url;
        }

        // 2. 重写getCacheKey()
        @Override
        public String getCacheKey() {
            return mUrl.replace(deleteToken(), "");
            // 通过 deleteToken() 从 带有token参数的图片Url地址中 去掉 token参数
            // 最终返回一个没有token参数、初始的图片URL地址
            // ->>分析1
        }

        // 分析1：deleteToken()
        private String deleteToken() {
            String tokenParam = "";
            int tokenKeyIndex = mUrl.indexOf("?token=") >= 0 ? mUrl.indexOf("?token=") : mUrl.indexOf("&token=");
            if (tokenKeyIndex != -1) {
                int nextAndIndex = mUrl.indexOf("&", tokenKeyIndex + 1);
                if (nextAndIndex != -1) {
                    tokenParam = mUrl.substring(tokenKeyIndex + 1, nextAndIndex + 1);
                } else {
                    tokenParam = mUrl.substring(tokenKeyIndex);
                }
            }
            return tokenParam;
        }

    }

/**
  * 使用缓存时：需要在load()中传入自定义的 mGlideUrl对象
  **/

    Glide.with(this)
         .load(new mGlideUrl(url))
         .into(imageView);

    // 注：a. 若像之前直接传入图片的url地址，那么在内部还是会使用原始的GlideUrl类
    //    b. 即直接将传入传入图片的url地址作为缓存key的Id参数，而没有对token参数作任何处理
```

------

# 4. 总结

本文主要对**`Glide`的图片缓存功能**的使用问题进行讲解







今天我们就先从缓存这一块内容开始入手吧。不过今天文章中的源码都建在上一篇源码分析的基础之上，还没有看过上一篇文章的朋友，建议先去阅读 Android图片加载框架最全解析（二），从源码的角度理解Glide的执行流程 。

Glide缓存简介
Glide的缓存设计可以说是非常先进的，考虑的场景也很周全。在缓存这一功能上，Glide又将它分成了两个模块，一个是内存缓存，一个是硬盘缓存。

这两个缓存模块的作用各不相同，内存缓存的主要作用是防止应用重复将图片数据读取到内存当中，而硬盘缓存的主要作用是防止应用重复从网络或其他地方重复下载和读取数据。

内存缓存和硬盘缓存的相互结合才构成了Glide极佳的图片缓存效果，那么接下来我们就分别来分析一下这两种缓存的使用方法以及它们的实现原理。

缓存Key
既然是缓存功能，就必然会有用于进行缓存的Key。那么Glide的缓存Key是怎么生成的呢？我不得不说，Glide的缓存Key生成规则非常繁琐，决定缓存Key的参数竟然有10个之多。不过繁琐归繁琐，至少逻辑还是比较简单的，我们先来看一下Glide缓存Key的生成逻辑。

生成缓存Key的代码在Engine类的load()方法当中，这部分代码我们在上一篇文章当中已经分析过了，只不过当时忽略了缓存相关的内容，那么我们现在重新来看一下：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();
    
        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());
    
        ...
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
可以看到，这里在第11行调用了fetcher.getId()方法获得了一个id字符串，这个字符串也就是我们要加载的图片的唯一标识，比如说如果是一张网络上的图片的话，那么这个id就是这张图片的url地址。

接下来在第12行，将这个id连同着signature、width、height等等10个参数一起传入到EngineKeyFactory的buildKey()方法当中，从而构建出了一个EngineKey对象，这个EngineKey也就是Glide中的缓存Key了。

可见，决定缓存Key的条件非常多，即使你用override()方法改变了一下图片的width或者height，也会生成一个完全不同的缓存Key。

EngineKey类的源码大家有兴趣可以自己去看一下，其实主要就是重写了equals()和hashCode()方法，保证只有传入EngineKey的所有参数都相同的情况下才认为是同一个EngineKey对象，我就不在这里将源码贴出来了。

内存缓存
有了缓存Key，接下来就可以开始进行缓存了，那么我们先从内存缓存看起。

首先你要知道，默认情况下，Glide自动就是开启内存缓存的。也就是说，当我们使用Glide加载了一张图片之后，这张图片就会被缓存到内存当中，只要在它还没从内存中被清除之前，下次使用Glide再加载这张图片都会直接从内存当中读取，而不用重新从网络或硬盘上读取了，这样无疑就可以大幅度提升图片的加载效率。比方说你在一个RecyclerView当中反复上下滑动，RecyclerView中只要是Glide加载过的图片都可以直接从内存当中迅速读取并展示出来，从而大大提升了用户体验。

而Glide最为人性化的是，你甚至不需要编写任何额外的代码就能自动享受到这个极为便利的内存缓存功能，因为Glide默认就已经将它开启了。

那么既然已经默认开启了这个功能，还有什么可讲的用法呢？只有一点，如果你有什么特殊的原因需要禁用内存缓存功能，Glide对此提供了接口：

Glide.with(this)
     .load(url)
     .skipMemoryCache(true)
     .into(imageView);
1
2
3
4
可以看到，只需要调用skipMemoryCache()方法并传入true，就表示禁用掉Glide的内存缓存功能。

没错，关于Glide内存缓存的用法就只有这么多，可以说是相当简单。但是我们不可能只停留在这么简单的层面上，接下来就让我们就通过阅读源码来分析一下Glide的内存缓存功能是如何实现的。

其实说到内存缓存的实现，非常容易就让人想到LruCache算法（Least Recently Used），也叫近期最少使用算法。它的主要算法原理就是把最近使用的对象用强引用存储在LinkedHashMap中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。LruCache的用法也比较简单，我在 Android高效加载大图、多图解决方案，有效避免程序OOM 这篇文章当中有提到过它的用法，感兴趣的朋友可以去参考一下。

那么不必多说，Glide内存缓存的实现自然也是使用的LruCache算法。不过除了LruCache算法之外，Glide还结合了一种弱引用的机制，共同完成了内存缓存功能，下面就让我们来通过源码分析一下。

首先回忆一下，在上一篇文章的第二步load()方法中，我们当时分析到了在loadGeneric()方法中会调用Glide.buildStreamModelLoader()方法来获取一个ModelLoader对象。当时没有再跟进到这个方法的里面再去分析，那么我们现在来看下它的源码：

public class Glide {

    public static <T, Y> ModelLoader<T, Y> buildModelLoader(Class<T> modelClass, Class<Y> resourceClass,
            Context context) {
         if (modelClass == null) {
            if (Log.isLoggable(TAG, Log.DEBUG)) {
                Log.d(TAG, "Unable to load null model, setting placeholder only");
            }
            return null;
        }
        return Glide.get(context).getLoaderFactory().buildModelLoader(modelClass, resourceClass);
    }
    
    public static Glide get(Context context) {
        if (glide == null) {
            synchronized (Glide.class) {
                if (glide == null) {
                    Context applicationContext = context.getApplicationContext();
                    List<GlideModule> modules = new ManifestParser(applicationContext).parse();
                    GlideBuilder builder = new GlideBuilder(applicationContext);
                    for (GlideModule module : modules) {
                        module.applyOptions(applicationContext, builder);
                    }
                    glide = builder.createGlide();
                    for (GlideModule module : modules) {
                        module.registerComponents(applicationContext, glide);
                    }
                }
            }
        }
        return glide;
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
这里我们还是只看关键，在第11行去构建ModelLoader对象的时候，先调用了一个Glide.get()方法，而这个方法就是关键。我们可以看到，get()方法中实现的是一个单例功能，而创建Glide对象则是在第24行调用GlideBuilder的createGlide()方法来创建的，那么我们跟到这个方法当中：

public class GlideBuilder {
    ...

    Glide createGlide() {
        if (sourceService == null) {
            final int cores = Math.max(1, Runtime.getRuntime().availableProcessors());
            sourceService = new FifoPriorityThreadPoolExecutor(cores);
        }
        if (diskCacheService == null) {
            diskCacheService = new FifoPriorityThreadPoolExecutor(1);
        }
        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        if (bitmapPool == null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
                int size = calculator.getBitmapPoolSize();
                bitmapPool = new LruBitmapPool(size);
            } else {
                bitmapPool = new BitmapPoolAdapter();
            }
        }
        if (memoryCache == null) {
            memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
        }
        if (diskCacheFactory == null) {
            diskCacheFactory = new InternalCacheDiskCacheFactory(context);
        }
        if (engine == null) {
            engine = new Engine(memoryCache, diskCacheFactory, diskCacheService, sourceService);
        }
        if (decodeFormat == null) {
            decodeFormat = DecodeFormat.DEFAULT;
        }
        return new Glide(engine, memoryCache, bitmapPool, context, decodeFormat);
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
这里也就是构建Glide对象的地方了。那么观察第22行，你会发现这里new出了一个LruResourceCache，并把它赋值到了memoryCache这个对象上面。你没有猜错，这个就是Glide实现内存缓存所使用的LruCache对象了。不过我这里并不打算展开来讲LruCache算法的具体实现，如果你感兴趣的话可以自己研究一下它的源码。

现在创建好了LruResourceCache对象只能说是把准备工作做好了，接下来我们就一步步研究Glide中的内存缓存到底是如何实现的。

刚才在Engine的load()方法中我们已经看到了生成缓存Key的代码，而内存缓存的代码其实也是在这里实现的，那么我们重新来看一下Engine类load()方法的完整源码：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    ...    

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();
    
        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());
    
        EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
        if (cached != null) {
            cb.onResourceReady(cached);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from cache", startTime, key);
            }
            return null;
        }
    
        EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
        if (active != null) {
            cb.onResourceReady(active);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from active resources", startTime, key);
            }
            return null;
        }
    
        EngineJob current = jobs.get(key);
        if (current != null) {
            current.addCallback(cb);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Added to existing load", startTime, key);
            }
            return new LoadStatus(cb, current);
        }
    
        EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
        DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
                transcoder, diskCacheProvider, diskCacheStrategy, priority);
        EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
        jobs.put(key, engineJob);
        engineJob.addCallback(cb);
        engineJob.start(runnable);
    
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Started new load", startTime, key);
        }
        return new LoadStatus(cb, engineJob);
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
可以看到，这里在第17行调用了loadFromCache()方法来获取缓存图片，如果获取到就直接调用cb.onResourceReady()方法进行回调。如果没有获取到，则会在第26行调用loadFromActiveResources()方法来获取缓存图片，获取到的话也直接进行回调。只有在两个方法都没有获取到缓存的情况下，才会继续向下执行，从而开启线程来加载图片。

也就是说，Glide的图片加载过程中会调用两个方法来获取内存缓存，loadFromCache()和loadFromActiveResources()。这两个方法中一个使用的就是LruCache算法，另一个使用的就是弱引用。我们来看一下它们的源码：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    private final MemoryCache cache;
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...
    
    private EngineResource<?> loadFromCache(Key key, boolean isMemoryCacheable) {
        if (!isMemoryCacheable) {
            return null;
        }
        EngineResource<?> cached = getEngineResourceFromCache(key);
        if (cached != null) {
            cached.acquire();
            activeResources.put(key, new ResourceWeakReference(key, cached, getReferenceQueue()));
        }
        return cached;
    }
    
    private EngineResource<?> getEngineResourceFromCache(Key key) {
        Resource<?> cached = cache.remove(key);
        final EngineResource result;
        if (cached == null) {
            result = null;
        } else if (cached instanceof EngineResource) {
            result = (EngineResource) cached;
        } else {
            result = new EngineResource(cached, true /*isCacheable*/);
        }
        return result;
    }
    
    private EngineResource<?> loadFromActiveResources(Key key, boolean isMemoryCacheable) {
        if (!isMemoryCacheable) {
            return null;
        }
        EngineResource<?> active = null;
        WeakReference<EngineResource<?>> activeRef = activeResources.get(key);
        if (activeRef != null) {
            active = activeRef.get();
            if (active != null) {
                active.acquire();
            } else {
                activeResources.remove(key);
            }
        }
        return active;
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
在loadFromCache()方法的一开始，首先就判断了isMemoryCacheable是不是false，如果是false的话就直接返回null。这是什么意思呢？其实很简单，我们刚刚不是学了一个skipMemoryCache()方法吗？如果在这个方法中传入true，那么这里的isMemoryCacheable就会是false，表示内存缓存已被禁用。

我们继续住下看，接着调用了getEngineResourceFromCache()方法来获取缓存。在这个方法中，会使用缓存Key来从cache当中取值，而这里的cache对象就是在构建Glide对象时创建的LruResourceCache，那么说明这里其实使用的就是LruCache算法了。

但是呢，观察第22行，当我们从LruResourceCache中获取到缓存图片之后会将它从缓存中移除，然后在第16行将这个缓存图片存储到activeResources当中。activeResources就是一个弱引用的HashMap，用来缓存正在使用中的图片，我们可以看到，loadFromActiveResources()方法就是从activeResources这个HashMap当中取值的。使用activeResources来缓存正在使用中的图片，可以保护这些图片不会被LruCache算法回收掉。

好的，从内存缓存中读取数据的逻辑大概就是这些了。概括一下来说，就是如果能从内存缓存当中读取到要加载的图片，那么就直接进行回调，如果读取不到的话，才会开启线程执行后面的图片加载逻辑。

现在我们已经搞明白了内存缓存读取的原理，接下来的问题就是内存缓存是在哪里写入的呢？这里我们又要回顾一下上一篇文章中的内容了。还记不记得我们之前分析过，当图片加载完成之后，会在EngineJob当中通过Handler发送一条消息将执行逻辑切回到主线程当中，从而执行handleResultOnMainThread()方法。那么我们现在重新来看一下这个方法，代码如下所示：

class EngineJob implements EngineRunnable.EngineRunnableManager {

    private final EngineResourceFactory engineResourceFactory;
    ...
    
    private void handleResultOnMainThread() {
        if (isCancelled) {
            resource.recycle();
            return;
        } else if (cbs.isEmpty()) {
            throw new IllegalStateException("Received a resource without any callbacks to notify");
        }
        engineResource = engineResourceFactory.build(resource, isCacheable);
        hasResource = true;
        engineResource.acquire();
        listener.onEngineJobComplete(key, engineResource);
        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                engineResource.acquire();
                cb.onResourceReady(engineResource);
            }
        }
        engineResource.release();
    }
    
    static class EngineResourceFactory {
        public <R> EngineResource<R> build(Resource<R> resource, boolean isMemoryCacheable) {
            return new EngineResource<R>(resource, isMemoryCacheable);
        }
    }
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
在第13行，这里通过EngineResourceFactory构建出了一个包含图片资源的EngineResource对象，然后会在第16行将这个对象回调到Engine的onEngineJobComplete()方法当中，如下所示：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    ...    

    @Override
    public void onEngineJobComplete(Key key, EngineResource<?> resource) {
        Util.assertMainThread();
        // A null resource indicates that the load failed, usually due to an exception.
        if (resource != null) {
            resource.setResourceListener(key, this);
            if (resource.isCacheable()) {
                activeResources.put(key, new ResourceWeakReference(key, resource, getReferenceQueue()));
            }
        }
        jobs.remove(key);
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
现在就非常明显了，可以看到，在第13行，回调过来的EngineResource被put到了activeResources当中，也就是在这里写入的缓存。

那么这只是弱引用缓存，还有另外一种LruCache缓存是在哪里写入的呢？这就要介绍一下EngineResource中的一个引用机制了。观察刚才的handleResultOnMainThread()方法，在第15行和第19行有调用EngineResource的acquire()方法，在第23行有调用它的release()方法。其实，EngineResource是用一个acquired变量用来记录图片被引用的次数，调用acquire()方法会让变量加1，调用release()方法会让变量减1，代码如下所示：

class EngineResource<Z> implements Resource<Z> {

    private int acquired;
    ...
    
    void acquire() {
        if (isRecycled) {
            throw new IllegalStateException("Cannot acquire a recycled resource");
        }
        if (!Looper.getMainLooper().equals(Looper.myLooper())) {
            throw new IllegalThreadStateException("Must call acquire on the main thread");
        }
        ++acquired;
    }
    
    void release() {
        if (acquired <= 0) {
            throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
        }
        if (!Looper.getMainLooper().equals(Looper.myLooper())) {
            throw new IllegalThreadStateException("Must call release on the main thread");
        }
        if (--acquired == 0) {
            listener.onResourceReleased(key, this);
        }
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
也就是说，当acquired变量大于0的时候，说明图片正在使用中，也就应该放到activeResources弱引用缓存当中。而经过release()之后，如果acquired变量等于0了，说明图片已经不再被使用了，那么此时会在第24行调用listener的onResourceReleased()方法来释放资源，这个listener就是Engine对象，我们来看下它的onResourceReleased()方法：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    private final MemoryCache cache;
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...    
    
    @Override
    public void onResourceReleased(Key cacheKey, EngineResource resource) {
        Util.assertMainThread();
        activeResources.remove(cacheKey);
        if (resource.isCacheable()) {
            cache.put(cacheKey, resource);
        } else {
            resourceRecycler.recycle(resource);
        }
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
可以看到，这里首先会将缓存图片从activeResources中移除，然后再将它put到LruResourceCache当中。这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。

这就是Glide内存缓存的实现原理。

硬盘缓存
接下来我们开始学习硬盘缓存方面的内容。

不知道你还记不记得，在本系列的第一篇文章中我们就使用过硬盘缓存的功能了。当时为了禁止Glide对图片进行硬盘缓存而使用了如下代码：

Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
1
2
3
4
调用diskCacheStrategy()方法并传入DiskCacheStrategy.NONE，就可以禁用掉Glide的硬盘缓存功能了。

这个diskCacheStrategy()方法基本上就是Glide硬盘缓存功能的一切，它可以接收四种参数：

DiskCacheStrategy.NONE： 表示不缓存任何内容。
DiskCacheStrategy.SOURCE： 表示只缓存原始图片。
DiskCacheStrategy.RESULT： 表示只缓存转换过后的图片（默认选项）。
DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。
上面四种参数的解释本身并没有什么难理解的地方，但是有一个概念大家需要了解，就是当我们使用Glide去加载一张图片的时候，Glide默认并不会将原始图片展示出来，而是会对图片进行压缩和转换（我们会在后面学习这方面的内容）。总之就是经过种种一系列操作之后得到的图片，就叫转换过后的图片。而Glide默认情况下在硬盘缓存的就是转换过后的图片，我们通过调用diskCacheStrategy()方法则可以改变这一默认行为。

好的，关于Glide硬盘缓存的用法也就只有这么多，那么接下来还是老套路，我们通过阅读源码来分析一下，Glide的硬盘缓存功能是如何实现的。

首先，和内存缓存类似，硬盘缓存的实现也是使用的LruCache算法，而且Google还提供了一个现成的工具类DiskLruCache。我之前也专门写过一篇文章对这个DiskLruCache工具进行了比较全面的分析，感兴趣的朋友可以参考一下 Android DiskLruCache完全解析，硬盘缓存的最佳方案 。当然，Glide是使用的自己编写的DiskLruCache工具类，但是基本的实现原理都是差不多的。

接下来我们看一下Glide是在哪里读取硬盘缓存的。这里又需要回忆一下上篇文章中的内容了，Glide开启线程来加载图片后会执行EngineRunnable的run()方法，run()方法中又会调用一个decode()方法，那么我们重新再来看一下这个decode()方法的源码：

private Resource<?> decode() throws Exception {
    if (isDecodingFromCache()) {
        return decodeFromCache();
    } else {
        return decodeFromSource();
    }
}
1
2
3
4
5
6
7
可以看到，这里会分为两种情况，一种是调用decodeFromCache()方法从硬盘缓存当中读取图片，一种是调用decodeFromSource()来读取原始图片。默认情况下Glide会优先从缓存当中读取，只有缓存中不存在要读取的图片时，才会去读取原始图片。那么我们现在来看一下decodeFromCache()方法的源码，如下所示：

private Resource<?> decodeFromCache() throws Exception {
    Resource<?> result = null;
    try {
        result = decodeJob.decodeResultFromCache();
    } catch (Exception e) {
        if (Log.isLoggable(TAG, Log.DEBUG)) {
            Log.d(TAG, "Exception decoding result from cache: " + e);
        }
    }
    if (result == null) {
        result = decodeJob.decodeSourceFromCache();
    }
    return result;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
可以看到，这里会先去调用DecodeJob的decodeResultFromCache()方法来获取缓存，如果获取不到，会再调用decodeSourceFromCache()方法获取缓存，这两个方法的区别其实就是DiskCacheStrategy.RESULT和DiskCacheStrategy.SOURCE这两个参数的区别，相信不需要我再做什么解释吧。

那么我们来看一下这两个方法的源码吧，如下所示：

public Resource<Z> decodeResultFromCache() throws Exception {
    if (!diskCacheStrategy.cacheResult()) {
        return null;
    }
    long startTime = LogTime.getLogTime();
    Resource<T> transformed = loadFromCache(resultKey);
    startTime = LogTime.getLogTime();
    Resource<Z> result = transcode(transformed);
    return result;
}

public Resource<Z> decodeSourceFromCache() throws Exception {
    if (!diskCacheStrategy.cacheSource()) {
        return null;
    }
    long startTime = LogTime.getLogTime();
    Resource<T> decoded = loadFromCache(resultKey.getOriginalKey());
    return transformEncodeAndTranscode(decoded);
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
可以看到，它们都是调用了loadFromCache()方法从缓存当中读取数据，如果是decodeResultFromCache()方法就直接将数据解码并返回，如果是decodeSourceFromCache()方法，还要调用一下transformEncodeAndTranscode()方法先将数据转换一下再解码并返回。

然而我们注意到，这两个方法中在调用loadFromCache()方法时传入的参数却不一样，一个传入的是resultKey，另外一个却又调用了resultKey的getOriginalKey()方法。这个其实非常好理解，刚才我们已经解释过了，Glide的缓存Key是由10个参数共同组成的，包括图片的width、height等等。但如果我们是缓存的原始图片，其实并不需要这么多的参数，因为不用对图片做任何的变化。那么我们来看一下getOriginalKey()方法的源码：

public Key getOriginalKey() {
    if (originalKey == null) {
        originalKey = new OriginalKey(id, signature);
    }
    return originalKey;
}
1
2
3
4
5
6
可以看到，这里其实就是忽略了绝大部分的参数，只使用了id和signature这两个参数来构成缓存Key。而signature参数绝大多数情况下都是用不到的，因此基本上可以说就是由id（也就是图片url）来决定的Original缓存Key。

搞明白了这两种缓存Key的区别，那么接下来我们看一下loadFromCache()方法的源码吧：

private Resource<T> loadFromCache(Key key) throws IOException {
    File cacheFile = diskCacheProvider.getDiskCache().get(key);
    if (cacheFile == null) {
        return null;
    }
    Resource<T> result = null;
    try {
        result = loadProvider.getCacheDecoder().decode(cacheFile, width, height);
    } finally {
        if (result == null) {
            diskCacheProvider.getDiskCache().delete(key);
        }
    }
    return result;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
这个方法的逻辑非常简单，调用getDiskCache()方法获取到的就是Glide自己编写的DiskLruCache工具类的实例，然后调用它的get()方法并把缓存Key传入，就能得到硬盘缓存的文件了。如果文件为空就返回null，如果文件不为空则将它解码成Resource对象后返回即可。

这样我们就将硬盘缓存读取的源码分析完了，那么硬盘缓存又是在哪里写入的呢？趁热打铁我们赶快继续分析下去。

刚才已经分析过了，在没有缓存的情况下，会调用decodeFromSource()方法来读取原始图片。那么我们来看下这个方法：

public Resource<Z> decodeFromSource() throws Exception {
    Resource<T> decoded = decodeSource();
    return transformEncodeAndTranscode(decoded);
}
1
2
3
4
这个方法中只有两行代码，decodeSource()顾名思义是用来解析原图片的，而transformEncodeAndTranscode()则是用来对图片进行转换和转码的。我们先来看decodeSource()方法：

private Resource<T> decodeSource() throws Exception {
    Resource<T> decoded = null;
    try {
        long startTime = LogTime.getLogTime();
        final A data = fetcher.loadData(priority);
        if (isCancelled) {
            return null;
        }
        decoded = decodeFromSourceData(data);
    } finally {
        fetcher.cleanup();
    }
    return decoded;
}

private Resource<T> decodeFromSourceData(A data) throws IOException {
    final Resource<T> decoded;
    if (diskCacheStrategy.cacheSource()) {
        decoded = cacheAndDecodeSourceData(data);
    } else {
        long startTime = LogTime.getLogTime();
        decoded = loadProvider.getSourceDecoder().decode(data, width, height);
    }
    return decoded;
}

private Resource<T> cacheAndDecodeSourceData(A data) throws IOException {
    long startTime = LogTime.getLogTime();
    SourceWriter<A> writer = new SourceWriter<A>(loadProvider.getSourceEncoder(), data);
    diskCacheProvider.getDiskCache().put(resultKey.getOriginalKey(), writer);
    startTime = LogTime.getLogTime();
    Resource<T> result = loadFromCache(resultKey.getOriginalKey());
    return result;
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
这里会在第5行先调用fetcher的loadData()方法读取图片数据，然后在第9行调用decodeFromSourceData()方法来对图片进行解码。接下来会在第18行先判断是否允许缓存原始图片，如果允许的话又会调用cacheAndDecodeSourceData()方法。而在这个方法中同样调用了getDiskCache()方法来获取DiskLruCache实例，接着调用它的put()方法就可以写入硬盘缓存了，注意原始图片的缓存Key是用的resultKey.getOriginalKey()。

好的，原始图片的缓存写入就是这么简单，接下来我们分析一下transformEncodeAndTranscode()方法的源码，来看看转换过后的图片缓存是怎么写入的。代码如下所示：

private Resource<Z> transformEncodeAndTranscode(Resource<T> decoded) {
    long startTime = LogTime.getLogTime();
    Resource<T> transformed = transform(decoded);
    writeTransformedToCache(transformed);
    startTime = LogTime.getLogTime();
    Resource<Z> result = transcode(transformed);
    return result;
}

private void writeTransformedToCache(Resource<T> transformed) {
    if (transformed == null || !diskCacheStrategy.cacheResult()) {
        return;
    }
    long startTime = LogTime.getLogTime();
    SourceWriter<Resource<T>> writer = new SourceWriter<Resource<T>>(loadProvider.getEncoder(), transformed);
    diskCacheProvider.getDiskCache().put(resultKey, writer);
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
这里的逻辑就更加简单明了了。先是在第3行调用transform()方法来对图片进行转换，然后在writeTransformedToCache()方法中将转换过后的图片写入到硬盘缓存中，调用的同样是DiskLruCache实例的put()方法，不过这里用的缓存Key是resultKey。

这样我们就将Glide硬盘缓存的实现原理也分析完了。虽然这些源码看上去如此的复杂，但是经过Glide出色的封装，使得我们只需要通过skipMemoryCache()和diskCacheStrategy()这两个方法就可以轻松自如地控制Glide的缓存功能了。

了解了Glide缓存的实现原理之后，接下来我们再来学习一些Glide缓存的高级技巧吧。

高级技巧
虽说Glide将缓存功能高度封装之后，使得用法变得非常简单，但同时也带来了一些问题。

比如之前有一位群里的朋友就跟我说过，他们项目的图片资源都是存放在七牛云上面的，而七牛云为了对图片资源进行保护，会在图片url地址的基础之上再加上一个token参数。也就是说，一张图片的url地址可能会是如下格式：

http://url.com/image.jpg?token=d9caa6e02c990b0a
1
而使用Glide加载这张图片的话，也就会使用这个url地址来组成缓存Key。

但是接下来问题就来了，token作为一个验证身份的参数并不是一成不变的，很有可能时时刻刻都在变化。而如果token变了，那么图片的url也就跟着变了，图片url变了，缓存Key也就跟着变了。结果就造成了，明明是同一张图片，就因为token不断在改变，导致Glide的缓存功能完全失效了。

这其实是个挺棘手的问题，而且我相信绝对不仅仅是七牛云这一个个例，大家在使用Glide的时候很有可能都会遇到这个问题。

那么该如何解决这个问题呢？我们还是从源码的层面进行分析，首先再来看一下Glide生成缓存Key这部分的代码：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();
    
        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());
    
        ...
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
来看一下第11行，刚才已经说过了，这个id其实就是图片的url地址。那么，这里是通过调用fetcher.getId()方法来获取的图片url地址，而我们在上一篇文章中已经知道了，fetcher就是HttpUrlFetcher的实例，我们就来看一下它的getId()方法的源码吧，如下所示：

public class HttpUrlFetcher implements DataFetcher<InputStream> {

    private final GlideUrl glideUrl;
    ...
    
    public HttpUrlFetcher(GlideUrl glideUrl) {
        this(glideUrl, DEFAULT_CONNECTION_FACTORY);
    }
    
    HttpUrlFetcher(GlideUrl glideUrl, HttpUrlConnectionFactory connectionFactory) {
        this.glideUrl = glideUrl;
        this.connectionFactory = connectionFactory;
    }
    
    @Override
    public String getId() {
        return glideUrl.getCacheKey();
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
可以看到，getId()方法中又调用了GlideUrl的getCacheKey()方法。那么这个GlideUrl对象是从哪里来的呢？其实就是我们在load()方法中传入的图片url地址，然后Glide在内部把这个url地址包装成了一个GlideUrl对象。

很明显，接下来我们就要看一下GlideUrl的getCacheKey()方法的源码了，如下所示：

public class GlideUrl {

    private final URL url;
    private final String stringUrl;
    ...
    
    public GlideUrl(URL url) {
        this(url, Headers.DEFAULT);
    }
    
    public GlideUrl(String url) {
        this(url, Headers.DEFAULT);
    }
    
    public GlideUrl(URL url, Headers headers) {
        ...
        this.url = url;
        stringUrl = null;
    }
    
    public GlideUrl(String url, Headers headers) {
        ...
        this.stringUrl = url;
        this.url = null;
    }
    
    public String getCacheKey() {
        return stringUrl != null ? stringUrl : url.toString();
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
这里我将代码稍微进行了一点简化，这样看上去更加简单明了。GlideUrl类的构造函数接收两种类型的参数，一种是url字符串，一种是URL对象。然后getCacheKey()方法中的判断逻辑非常简单，如果传入的是url字符串，那么就直接返回这个字符串本身，如果传入的是URL对象，那么就返回这个对象toString()后的结果。

其实看到这里，我相信大家已经猜到解决方案了，因为getCacheKey()方法中的逻辑太直白了，直接就是将图片的url地址进行返回来作为缓存Key的。那么其实我们只需要重写这个getCacheKey()方法，加入一些自己的逻辑判断，就能轻松解决掉刚才的问题了。

创建一个MyGlideUrl继承自GlideUrl，代码如下所示：

public class MyGlideUrl extends GlideUrl {

    private String mUrl;
    
    public MyGlideUrl(String url) {
        super(url);
        mUrl = url;
    }
    
    @Override
    public String getCacheKey() {
        return mUrl.replace(findTokenParam(), "");
    }
    
    private String findTokenParam() {
        String tokenParam = "";
        int tokenKeyIndex = mUrl.indexOf("?token=") >= 0 ? mUrl.indexOf("?token=") : mUrl.indexOf("&token=");
        if (tokenKeyIndex != -1) {
            int nextAndIndex = mUrl.indexOf("&", tokenKeyIndex + 1);
            if (nextAndIndex != -1) {
                tokenParam = mUrl.substring(tokenKeyIndex + 1, nextAndIndex + 1);
            } else {
                tokenParam = mUrl.substring(tokenKeyIndex);
            }
        }
        return tokenParam;
    }

}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
可以看到，这里我们重写了getCacheKey()方法，在里面加入了一段逻辑用于将图片url地址中token参数的这一部分移除掉。这样getCacheKey()方法得到的就是一个没有token参数的url地址，从而不管token怎么变化，最终Glide的缓存Key都是固定不变的了。

当然，定义好了MyGlideUrl，我们还得使用它才行，将加载图片的代码改成如下方式即可：

Glide.with(this)
     .load(new MyGlideUrl(url))
     .into(imageView);
1
2
3
也就是说，我们需要在load()方法中传入这个自定义的MyGlideUrl对象，而不能再像之前那样直接传入url字符串了。不然的话Glide在内部还是会使用原始的GlideUrl类，而不是我们自定义的MyGlideUrl类。

这样我们就将这个棘手的缓存问题给解决掉了。
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

原文链接：

# 参考

[Android图片加载框架最全解析（三），深入探究Glide的缓存机制](https://blog.csdn.net/guolin_blog/article/details/54895665)

[Android：深入剖析图片加载库Glide缓存功能(源码分析）](https://www.jianshu.com/p/e70d16275e9a)

[Android图片加载的那些事：为什么你的Glide缓存没有起作用？](https://www.jianshu.com/p/3c2a8471361e)