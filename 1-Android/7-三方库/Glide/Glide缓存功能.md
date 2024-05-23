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

------

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

作者：Carson带你学安卓
链接：
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





# 参考

[Android：深入剖析图片加载库Glide缓存功能(源码分析）](https://www.jianshu.com/p/e70d16275e9a)

[Android图片加载的那些事：为什么你的Glide缓存没有起作用？](https://www.jianshu.com/p/3c2a8471361e)