# Glide流程分析

## 第一条主线

构建Request请求，将请求加入到队列中。（生产者-消费者模型）

**加入队列流程：**

```java
RequestManager with = Glide.with(this);
RequestBuilder<Drawable> load = with.load(url);
load.into(iv);   // 前面的暂时先不看，当调用into方法后，说明加载图片的请求才真正开始
```

**继续调用**

```java
return into(
    glideContext.buildImageViewTarget(view, transcodeClass),
    /*targetListener=*/ null,
    requestOptions);
```

**继续跟踪，会发现以下代码**

```java
requestManager.clear(target);
target.setRequest(request);
requestManager.track(target, request);//发送请求开始的地方

void track(Target<?> target, Request request) {
  targetTracker.track(target);
  requestTracker.runRequest(request);//从名字看叫运行请求
}
```

**继续跟踪**

```java
// 通过该方法得知Glide也有两个队列；运行队列和等待队列；
public void runRequest(Request request) {
  requests.add(request);//加入运行队列；
  if (!isPaused) {
    request.begin();//开始执行
  } else {
    pendingRequests.add(request);//加入等待队列
  }
}
```

## 第二条主线

**请求如何运行？**

在第一条主线中，`request.begin()`方法就是真正开始执行请求的时候；先找到`request`的实现类：`SingleRequest`，找到其`begin`方法；

**为什么找到的是SingleRequest？**

在第一条主线的`RequestBuilder.into`方法中有一句代码；

```java
Request request = buildRequest(target, targetListener, options);
```

**继续跟踪它**

```java
buildRequestRecursive() 找到构建request的方法；
```

**在该方法中，又能跟踪到**

```java
Request mainRequest = buildThumbnailRequestRecursive()
```

**继续跟踪**

```java
Request fullRequest =
    obtainRequest(
        target,
        targetListener,
        requestOptions,
        coordinator,
        transitionOptions,
        priority,
        overrideWidth,
        overrideHeight);
```

上述代码块最终调用的是`SingleRequest.obtain()`方法，从而得到一个SingleRequest对象；所以能得出结论，`request.begin（）`方法被调用时，即调用了`SingleRequest`的`begin`方法；继续跟踪`begin`方法，会发现`onSizeReady`方法；

```java
onSizeReady(overrideWidth, overrideHeight);
```

在`begin`方法中跟踪到`engine.load`方法，如下（只抽取了部分代码）：

```java
// 从活动缓存中获取
EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
    if (active != null) {
      cb.onResourceReady(active, DataSource.MEMORY_CACHE);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Loaded resource from active resources", startTime, key);
      }
      return null;
    }

// 从内存缓存中获取
    EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
    if (cached != null) {
      cb.onResourceReady(cached, DataSource.MEMORY_CACHE);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Loaded resource from cache", startTime, key);
      }
      return null;
    }

// 硬盘缓存，硬盘缓存也是io操作，所以也使用了线程池；动画线程池
    EngineJob<?> current = jobs.get(key, onlyRetrieveFromCache);
    if (current != null) {
      current.addCallback(cb);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Added to existing load", startTime, key);
      }
      return new LoadStatus(cb, current);
    }

    EngineJob<R> engineJob =
        engineJobFactory.build(
            key,
            isMemoryCacheable,
            useUnlimitedSourceExecutorPool,
            useAnimationPool,
            onlyRetrieveFromCache);

    DecodeJob<R> decodeJob =
        decodeJobFactory.build(
            glideContext,
            model,
            key,
            signature,
            width,
            height,
            resourceClass,
            transcodeClass,
            priority,
            diskCacheStrategy,
            transformations,
            isTransformationRequired,
            isScaleOnlyOrNoTransform,
            onlyRetrieveFromCache,
            options,
            engineJob);

    jobs.put(key, engineJob);

    engineJob.addCallback(cb);
    engineJob.start(decodeJob); //具体的加载，engineJob为加载管理类，decodeJob则为将返回的图片数据进行编码管理的类；
```

调用`engineJob.start()`方法后，则会执行以下代码：

```java
public void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    GlideExecutor executor = decodeJob.willDecodeFromCache()
        ? diskCacheExecutor
        : getActiveSourceExecutor();
    executor.execute(decodeJob);
  }
```

继续跟踪找到`DecodeJob`的`run`方法；

```java
DecodeJob.run() 继续调用 runWrapped(); 再继续调用getNextGenerator()

private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
      // 根据主线我们目前都先不去处理跟Cache相关的类，直接进入SourceGenerator；这里使用了设计模式-状态模式；（请自行根据第二节内容进行查询）
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
  }
```

继续跟踪到`SourceGenerator`类中的`startNext`方法；

```java
loadData.fetcher.loadData(helper.getPriority(), this);
```

根据`fetcher`找到`HttpUrlFetcher`，并找到对应的`loadData`方法；最终发现Glide是通过`HttpUrlConnection`访问的服务器，并返回最终的`stream`；

**问题来了？我怎么知道是这个类的？为什么不是其他类？在这里代码就看不懂了，怎么办？猜测；**

既然应该不是再继续从缓存拿，而应该要去访问网络了；**所以找到具体访问网络的；发现找不到，怎么办？**

![image.png](images/Glide解析/1.png)

找它的实现类，有一个HttpUrlFetcher，那它在哪里初始化的？

![image.png](images/Glide解析/2.png)

通过`Find Usages`找到哪里调用了--->找到了`HttpGlideUrlLoader`；

再看这个方法`HttpGlideUrlLoader`哪里调用了；

![image.png](images/Glide解析/3.png)

找到了Glide，继续往上寻找，找到了Glide种的`build`方法 ，找就能找到`Glide.get(context);`方法

## 第三条主线

队列怎么维护的？在`MainActivity`中我们调用了如下代码：

```java
RequestManager with = Glide.with(this);
```

**继续跟踪到**

```java
getRetriever(activity).get(activity)//这里得到了一个RequestManagerRetriever对象,再通过RequestManagerRetriever调用get方法得到RequestManager
```

**继续往下**

```java
androidx.fragment.app.FragmentManager fm = activity.getSupportFragmentManager();
return this.supportFragmentGet(activity, fm, (Fragment)null);
```

通过`this.supportFragmentGet`方法（如下代码），最终我们得到`SupportRequestManagerFragment`对象；

```java
private RequestManager supportFragmentGet(@NonNull Context context, @NonNull androidx.fragment.app.FragmentManager fm, @Nullable Fragment parentHint) {
    SupportRequestManagerFragment current = this.getSupportRequestManagerFragment(fm, parentHint);//这段代码的内部如果能够得到Fragment就得到，得不到就重新new一个，并且这个fragment中没有进行任何的UI处理；
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
        Glide glide = Glide.get(context);
        requestManager = this.factory.build(glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
        current.setRequestManager(requestManager);
    }

    return requestManager;
}
```

得到`Fragment`对象后，再将`RequestManager`对象赋值进去，如果`RequestManager`为空，则帮助创建；

而`RequestManager`对象则是生命周期管理的重要一环，因为它实现了`LifecycleListener`接口，并且在创建`RequestManager`的时候，会将这个接口设置给自己；也就意味着，Glide创建了一个无UI的`fragment`，这个`fragment`又与`RequestManager`进行绑定；当用户的`activity`或者`fragment`被调用，系统会自动调用`fragment`的生命周期方法；而生命周期方法中又会回调`LifecycleListener`的方法，进而调用`RequestManager`，`RequestManager`则也拥有了生命周期；

当`RequestManager`的`onStart`方法被调用后，会通过一系列的调用，将运行中的请求全部放开，进行访问；

当`onStop`方法被调用时，则将运行中队列的数据取出来，如果当前请求正在运行则暂停，然后将所有的数据从运行队列中添加到等待队列中去；

当`onDestory`方法被调用时，则将运行队列和等待队列中的数据全部清除；再将监听移除；将`requestManager`从Glide中的绑定关系解除；



# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



### 原理

图片加载框架：Glide实现原理

Glide：加载、缓存、LRU算法（LRUCache原理）

Glide如何加载GIF

Glide如何确定图片加载完毕？

Glide生命周期是如何绑定的？

LruCache的底层实现？

### 缓存

Glide的缓存实现？

Glide的图片三级缓存

Glide缓存特点

Glide内存缓存如何控制大小？

Glide为我们做了哪些内存优化

图片缓存框架设计

### 对比

Fresco与Glide的对比

Glide和Picasso有什么区别？

### 开放&拓展

Glide的优点

如何自己设计一个大图加载框架



# 参考

1、[深入理解Glide源码：三条主线分析 Glide 执行流程](https://juejin.cn/post/7028830670692548639)

2、[Android 【手撕Glide】--Glide缓存机制](https://www.jianshu.com/p/b85f89fce019)

3、[Glide 源码分析解读-基于最新版Glide 4.9.0](https://www.jianshu.com/p/9bb50924d42a)

4、[Glide 源码分析解读-缓存模块-基于最新版Glide 4.9.0](https://www.jianshu.com/p/62b7f990ee83)