# 前置知识

提前掌握这些知识，可以更容易地理解 Glide 的原理，因为这些知识点都在Glide中有用到。反之，如果不懂的话，那理解起来就有阻碍，不那么通畅。

GC原理，强软弱虚引用（java.lang.ref.Reference，WeakReference，ReferenceQueue），线程池，synchronized，volatile，LinkedHashMap，泛型，DiskLruCache，Bitmap，Handler，

下面对各个模块所涉及的知识点做个归纳：

加载模块：

缓存模块：

生命周期管理：androidx.lifecycle 中的 LifecycleRegistry、Lifecycle

> 基于 4.16.0
>
> 不分析 @Deprecated 标记的方法或类，比如 with() 就有几个弃用的。

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





# 总览

------

下面将 Glide 分成了几个模块，先有个整体的印象，采用自顶向下的方法分析源码。

按照逻辑功能划分，可以把 Glide 框架大概的分成如下几个部分：

![img](images/Glide解析/4.png)

Glide 大体上可以分为如上几个模块。

下面通过一个常用案例来分析整个流程。

一般来说，我们使用如下代码加载一张网络图片：

```java
Glide.with(this)
        .load(url)
        .into(imgView);
```

假设这是我们的 APP 第一次使用 Glide 加载一张图片，那么流程如下：

![image-20240328170937161](images/Glide解析/5.png)上面的流程是简化版，省去了一部分东西，通过这张图能直观的了解到 Glide 的加载流程以及机制。

# 模块介绍

根据模块学习事半功倍，先看看 Glide 的分包结构：

![img](images/Glide解析/6.png)

## Glide

Glide 是**单例类**，通过 Glide#get(Context) 方法可以获取到实例。

Glide 类算是个**全局的配置类**，Encoder、Decoder、ModelLoader、Pool 等等都在这里设置，此外还提供了创建 RequestManager 的接口（Glide#with() 方法）。

使用 Glide 时会最先调用 Glide#with() 方法创建 RequestManager，Glide 中的 with() 方法有五个重载：

```java
RequestManager with(Context context)
RequestManager with(android.app.Activity)
RequestManager with(android.app.Fragment)
RequestManager with(android.support.v4.app.Fragment)
RequestManager with(android.support.v4.app.FragmentActivity)
```

Glide#with() 方法会将 RequestManager 的创建委托给 RequestManagerRetriever，RequestManagerRetriever 为单例类，调用 get(Context) 创建 RequestManager。

## GlideBuilder

GlideBuilder 是用来创建 Glide 实例的类，其中包含了很多个 get/set 方法，例如设置 BitmapPool、MemoryCache、ArrayPool 等，最终通过这些设置调用 build 方法构建 Glide，可以截取 build 方法中的一段代码来看一下：

```java
if (bitmapPool == null) {
    //创建 Bitmap 池
    int size = memorySizeCalculator.getBitmapPoolSize();
    if (size > 0) {
        bitmapPool = new LruBitmapPool(size);
    } else {
        bitmapPool = new BitmapPoolAdapter();
    }
}

//创建数组池
if (arrayPool == null) {
    arrayPool = new LruArrayPool(memorySizeCalculator.getArrayPoolSizeInBytes());
}

//创建内存缓存
if (memoryCache == null) {
    memoryCache = new LruResourceCache(memorySizeCalculator.getMemoryCacheSize());
}

//创建磁盘缓存
if (diskCacheFactory == null) {
    diskCacheFactory = new InternalCacheDiskCacheFactory(context);
}
```

上面截取的几行代码很具有代表性，这些数组池、缓存实现等最终都会当做 Glide 构造器的参数创建 Glide 实例。

## RequestManagerRetriever

------

上面说的 5 个重载的 Glide#with() 方法对应 RequestManagerRetriever 中的 5 个重载的 get() 方法。
由于这个比较重要，而且跟我们使用息息相关，所以仔细的说一下~

创建 RequestManager 逻辑如下：

1. 如果 with 方法的参数为 Activity 或者 Fragment ，则最终调用 RequestManagerRetriever 中的 fragmentGet(Context, android.app.FragmentManager) 方法创建 RequestManager；
2. 如果 with 方法的参数为 android.support.v4.app.Fragment 或者android.support.v4.app.FragmentActivity，则最终调用 supportFragmentGet(Context, android.support.v4.app.FragmentManager) 方法创建 RequestManager；
3. 如果 with 方法的参数为 Context，则会判断其来源是否属于 FragmentActivity 及 Activity，是则按照上面的逻辑进行处理，否则最终调用 getApplicationManager(Context) 方法创建 RequestManager。

上面说的情况有个条件都是在主线程调用 Glide#with() 方法， 如果子线程调用 Glide#with() 或者系统版本小于 17，则最终会调用 getApplicationManager(Context) 方法创建 RequestManager 。

也就是说，无论使用什么参数，最终都会进入如下三个方法创建 RequestManager：

```java
RequestManager fragmentGet(Context context, android.app.FragmentManager fm);
RequestManager supportFragmentGet(Context context, android.support.v4.app.FragmentManager fm);
RequestManager getApplicationManager(Context context);
```

可以看到这三个方法作用都是用来创建 RequestManager，前两个方法主要是用来兼容 support 包中的 FragmentActivity、Fragment。

至于为什么需要传入一个 FragmentManager 参数留在后面说。

此外还有一种情况，即在**子线程**调用 Glide#with() 方法或传入 Context 对象为 ApplicationContext，此时会创建一个全局唯一的 RequestManager，生命周期与 APP 周期保持一致。

根据上述规则可以得出以下几个结论：

1. 同一个 Activity 对应一个 FragmentManager，一个 FragmentManager 对应一个 RequestManagerFragment，一个 RequestManagerFragment 对应一个 RequestManager，所以**一个 Activity 对应 一个 RequestManager**；
2. 同一个 Fragment 同样可得出上述结论；
3. 但如果 Fragment 属于 Activity，或者 Fragment 属于 Fragment，在 Activity、Framgnent 中分别创建 Glide 请求是并不会只创建一个 RequestManager；
4. **子线程**发起 Glide 请求或传入对象为 ApplicationContext，则使用全局单例的 RequestManager。

## RequestManager

------

RequestManager 主要由两个作用：

1. 创建 RequestBuilder ；
2. 通过生命周期管理请求的启动结束等。

我们都知道使用 Glide 加载图片时，如果当前页面被销毁或者不可见时会停止加载图片，但我们使用 Glide 加载图片时并没有显示的去设置 Glide 与当前页面的生命周期关联起来，只是传了个 Context 对象，那么 Glide 是如何通过一个上下文对象就能获取到页面生命周期的呢？

通过上面 RequestManagerRetriever 章节的介绍我们知道创建 RequestManager 时需要一个 FragmentManager 参数（全局 RequestManager 除外），那么再创建 RequestManager 时会**先创建一个不可见的 Fragment** ，通过 FM 加入到当前页面，用这个不可见的 Fragment 即可检测页面的生命周期。代码中保证了每个 Activity/Fragment 中只包含一个 RequestManagerFragment 与 一个 RequestManager。

创建 RequestBuilder 的 load 方法有很多：

```java
RequestBuilder<Drawable> load(@Nullable Bitmap bitmap);
RequestBuilder<Drawable> load(@Nullable Drawable drawable);
RequestBuilder<Drawable> load(@Nullable String string);
RequestBuilder<Drawable> load(@Nullable Uri uri);
RequestBuilder<Drawable> load(@Nullable File file);
RequestBuilder<Drawable> load(@RawRes @DrawableRes @Nullable Integer resourceId);
RequestBuilder<Drawable> load(@Nullable URL url);
RequestBuilder<Drawable> load(@Nullable byte[] model);
RequestBuilder<Drawable> load(@Nullable Object model);
```

看看有这么多重载方法，没一个都代表不同的加载源。
除此之外还有两个特殊的方法：

```java
RequestBuilder<File> downloadOnly();
RequestBuilder<File> download(@Nullable Object model);
```

这两个听名字就知道是用来下载图片的。

## RequestBuilder

------

RequestBuilder 用来构建请求，例如设置 RequestOption、缩略图、加载失败占位图等等。
上面说到的 RequestManager 中诸多的 load 重载方法，同样也对应 RequestBuilder 中的重载 load 方法，一般来说 load 方法之后就是调用 into 方法设置 ImageView 或者 Target，into 方法中最后会创建 Request，并启动，这个后面会详细介绍。

## Request

------

顾名思义， request 包下面的是封装的请求，里面有一个 Request 接口，估计所有的请求都是基于这个接口的，看一下：

![img](images/Glide解析/7.png)

接口定义了对请求的开始、结束、状态获取、回收等操作，所以请求中不仅包含基本的信息，还负责管理请求。
Request 主要的实现类有三个：

1. SingleRequest
2. ThumbnailRequestCoordinator
3. ErrorRequestCoordinator

一个个看。

### SingleRequest

------

这个类负责执行请求并将结果反映到 Target 上。
当我们使用 Glide 加载图片时，会先根据 Target 类型创建不同的 Target，然后 RequestBuilder 将这个 target 当做参数创建 Request 对象，Request 与 Target 就是这样关联起来的。

这里就会先创建一个包含 Target 的 SingleRequest 对象。考虑到性能问题，可能会连续创建很多个 SingleRequest 对象，所以使用了对象池来做缓存。
再来说说 SingleRequest 的请求发起流程。

我们经常在 Activity#onCreate 方法中直接使用 Glide 方法，但此时的图片大小还未确定，所以调用 Request#begin 时并不会直接发起请求，而是等待 ImageView 初始化完成，对于 ViewTarget 以及其子类来说，会注册View 的 OnPreDrawListener 事件，等待 View 初始化完成后就调用 SingleRequest#onSizeReady 方法，这个方法里就会开始加载图片了。

onSizeReady 方法并不会去直接加载图片，而是调用了 Engine#load 方法加载，这个方法差不多有二十个参数，所以 onSizeReady 方法算是用来构建参数列表并且调用 Engine#load 方法的。

clear 方法用于停止并清除请求，主要就是从 Engine 中移除掉这个任务以及回调接口。
另外，SingleRequest 实现了 ResourceCallback 接口，这个接口就连个方法：

```java
void onResourceReady(Resource<?> resource, DataSource dataSource);
void onLoadFailed(GlideException e);
```

即资源加载完成和加载失败的两个回调方法，刚刚说的 Engine#load 方法中有差不多二十个参数，其中有一个参数就是这个接口。那再来说这两个方法在 SingleRequest 中的实现。
其实很简单，重点就是调用 Target#onResourceReady 方法以及构建图片加载完成的动画，另外还要通知 ThumbnailRequestCoordinator 图片加载完成。
onLoadFailed 方法流程大体上也类似 onResourceReady。
那 SingleRequest 就差不多这样了。

### ThumbnailRequestCoordinator

------

这个类是用来**协调两个请求**，因为有的请求需要同时加载原图和缩略图，比如启动这两个请求、原图加载完成后缩略图其实就不需要加载了等等，这些控制都由这个类来操作。
RequestBuilder 中会将缩略图和原图的两个 SingleRequest 都交给它，后面再对其操作时都由这个类来同一控制。
所以这个类其实没什么太多的功能，就是对两个对象的一个统一个管理协调包装。

### ErrorRequestCoordinator

------

RequestBuilder 的父类 BaseRequestOptions 中有几个 error 的重载方法：

```java
T error(@Nullable Drawable drawable);
T error(@DrawableRes int resourceId);
```

一般地，我们会使用这个方法设置一个加载失败时的填充图，大部分情况下都是一个通过 resource 资源文件中获取到的图片 ID 或者 Drawable。
但 RequestBuilder 中还提供了另一个 error 方法：

```java
RequestBuilder<TranscodeType> error(@Nullable RequestBuilder<TranscodeType> errorBuilder);
```

考虑这样的一个场景，当我们加载失败时我可能希望继续去通过网络或者别的什么加载另一张图片，例如：

```java
Glide.with(context)
    .load((Object) null)
    .error(
        Glide.with(context)
            .load(errorModel)
            .listener(requestListener))
    .submit();
```

当我们这样使用 error 时最终就会创建一个 ErrorRequestCoordinator 对象，这个类的功能类似 ThumbnailRequestCoordinator，其中也没多少代码，主要用来协调 ThumbnailRequestCoordinator 以及 error 中的 Request。

通过上面的介绍就已经对 Request 的作用以及子类有一定的了解了，上面多次提到过 Target 是另一个很重要的概念，下面接着看一下这个类。

### Target

------

Target 代表一个**可被 Glide 加载并且具有生命周期的资源**。
当我们调用 RequestBuilder#into 方法时会根据传入参数创建对应类型的 Target 实现类。

那么 Target 在 Glide 的整个加载流程中到底扮演者什么样的角色呢？Target 的中文意思为：**目标**，实际上就是指加载完成后的图片应该放在哪， Target 默认提供了很多很有用的实现类，当然我们也可以自定义 Target。

Glide 默认提供了用于放在 ImageView 上的 ImageViewTarget（以及其各种子类）、放在 AppWidget 上的 AppWidgetTarget、用于同步加载图片的 FutureTarget（只有一个实现类：RequestFutureTarget）等等，下面分别来看一下。

#### CustomViewTarget

------

这个是抽象类，负责加载 Bitmap、Drawable 并且放到 View 上。

上文提到过，如果在 View 还未初始化完成时就调用了 Glide 加载图片会等待加载完成再去执行 onSizeReady 方法，那如何监听 View 初始化完成呢？
CustomViewTarget 就针对这个问题给出了解决方案，其中会调用 View#addOnAttachStateChangeListener 方法添加一个监听器，这个监听器可以监听到 View 被添加到 Widow 以及移除 Window 时的事件，从而更好的管理 Request 生命周期。

另外，构建好的 Request 会通过 View#setTag 方法存入 View 中，后面再通过 View#getTag 方法获取。

但这个抽象类并没有实现类，也没有被使用过，View 相关的 Target 都是继承 ViewTarget 抽象基类，但这个类已经被标记为过期类了，推荐将 ViewTarget 替换成 CustomViewTarget 使用。

#### ViewTarget

------

这个类又继承了抽象类 BaseTarget，这个基类里只是实现了 Target 接口的 setRequest 以及 getRequest 方法。
ViewTarget 基本上类似 CustomViewTarget ，只是具体的实现上有点不同。

#### ImageViewTarget

------

听名字就知道，这是加载到 ImageView 上的 Target，继承了 ViewTarget，同样也是个**抽象类**。

构造器中限定了**必须传入 ImageView 或者其子类**，图片数据加载完成后会回调其中的 onResourceReady 方法，第一步是将图片设置给 ImageView，第二部是判断是否需要使用动画，需要的话就执行动画。

ImageViewTarget 的实现类比较多，总共有 5 个，但内容都很简单，主要用于区分加载的资源时 Bitmap 类型还是 Drawable 类型，这个在构建请求时确定，默认的加载请求最终都是 Drawable 类型，但如果构建请求时调用了 asBitmap 方法那就资源就会被转成 Bitmap 类型，另外一个就是资源使用缩略图展示。

#### RequestFutureTarget

------

这是用来同步加载图片的 Target，调用 RequestBuilder#submit 将会返回一个 FutureTarget，调用 get 方法即可获取到加载的资源对象。

#### AppWidgetTarget

------

用于将下载的 Bitmap 设置到 RemoteView 上。

#### NotificationTarget

------

与 AppWidgetTarget 类似，不同的是这是用来将 Bitmap 设置到 Notification 中的 RemoteView 上。

## module

------

module 包下面的 GlideModel 比较重要，需要详细说一下。

这是用来**延迟设置 Glide 相关参数**的，我们可以通过这个接口使 Glide 在初始化时应用我们的设置，因为 Glide 是单例类，通过这个设置可以保证在 Glide 单例类初始时，所有请求发起之前应用到 Glide。

GlideModel 是个接口，所以代码很简单：

```java
@Deprecated
public interface GlideModule extends RegistersComponents, AppliesOptions { }
```

可以看到该接口被标识已过期，Glide 推荐使用 AppGlideModule 替代，不用管他。

GlideModel 接口本身没有代码内容，但其继承了 RegistersComponents 与 AppliesOptions 接口，先分别看一下这两个接口。

### RegistersComponents

------

这是用来注册 Glide 中一些组件的，这个接口只有一个方法：

```java
void registerComponents(@NonNull Context context, @NonNull Glide glide,
      @NonNull Registry registry);
```

这个方法中提供了一个 Registry 对象，这是用来管理注册 ModelLoader、Encoder、Decoder 等等，具体可以看看 Registry 提供的公开方法。

例如我们可以在这里注册自己的 ModelLoader，比如我们的网络请求使用的 OkHttp，Glide 默认使用的是HttpURLConnection，我们想改成 OkHttp 就可以在这里设置，具体的使用方式[点此查看使用案例。](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2F0xZhangKe%2FGlide-note%2Fblob%2Fmaster%2Fintegration%2Fokhttp3%2Fsrc%2Fmain%2Fjava%2Fcom%2Fbumptech%2Fglide%2Fintegration%2Fokhttp3%2FOkHttpGlideModule.java)

### AppliesOptions

------

这是用来管理一些 Glide 的参数设置项，同样只有一个方法。

```java
void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder);
```

这个方法提供了一个 GlideBuilder 参数，这是用来构建 Glide 的，我们可以使用 GlideBuilder 对象提供的公开方法做一些设置，例如设置线程池、设置 BitmapPool/ArrayPoll 等等。

那么说完这两个接口，在回过头来看看 GlideModel ，通过上面的描述已经明白 GlideModel 中两个方法的作用了，再来看看如何使用。

Glide 在实例化时会解析 manifest 文件并从中获取 value 为 GlideModule 的 meta-data 配置信息，我们定义好自己的 GlideModule 之后需要在 manifest 文件中进行配置，配置方式如下：

```xml
<meta-data
        android:name="com.zhangke.glide.samples.OkHttpGlideModule"
        android:value="GlideModule"/>
```

其中 OkHttpGlideModule 必须实现 GlideModel 接口。
具体的[配置方式点此查看](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2F0xZhangKe%2FGlide-note%2Ftree%2Fmaster%2Fintegration%2Fokhttp3)。

此外，Glide 默认提供了很多 ModelLoader，基本上可以满足所有场景的使用。
ModelLoader 的具体作用与机制后面会详细介绍。

## load

------

![img](images/Glide解析/8.png)

lload 包下面是加载资源的核心，里面的东西很多，也很复杂，所以我先把其中两个比较重要的接口介绍完了在介绍别的。

### ModelLoader

------

类路径：

```java
com.bumptech.glide.load.model.ModelLoader
```

工厂接口，用于将任意复杂的数据模型转换为可由 DataFetcher 用于获取模型所代表的资源的数据的具体数据类型。叫他加载器比较合适，用来加载资源的。

除此之外，还允许将图片按照 ImageView 大小按需加载。防止浪费内存。

Glide 初始化时会注册很多个 ModelLoader ，除了Glide 默认提供的之外还会注册用户在 manifest 中配置的 ModelLoader，也就是上面 GlideModel 章节介绍的内容。

ModelLoader 中有两个方法以及一个内部类：LoadData，下来看看这两个方法：

```java
@Nullable
LoadData<Data> buildLoadData(@NonNull Model model, int width, int height,
                                 @NonNull Options options);
boolean handles(@NonNull Model model);
```

buildLoadData 方法除了包含 Model 之外还有宽高以及 Option，所以光看参数列表应该能猜到，加载图片时可以根据需要的宽高以及其他设置做到按需加载。
返回的是 LoadData 实例，这个类待会再说。所以这个方法的意义就是通过参数构建一个 LoadData 实例。

handles 方法比较简单，就是用来判断给定模型是不是此加载器可能加载的已识别类型。

至于内部类 LoadData 呢，主要作用就是装了三个东西：

1. 用于识别资源唯一性的 Key;
2. 缓存相关的备用 Key 列表
3. DataFetcher

其中 DataFetcher最重要，为什么说它是最重要的呢，因为加载资源的根源就在这里（找了半天终于找到了），例如发起网络请求等等，都在这个里面。
那既然说到了 DataFetcher 就在说说它。

### DataFetcher

------

类路径：

```java
com.bumptech.glide.load.data.DataFetcher
```

DataFetcher 也是个接口，其中最重要的一个方法就是 loadData，听名字就很重要是吧：**加载数据**。

内部实现就是通过 HttpUrlConnect 发起网络请求，或者打开一个文件，或者使用 AssetManager 打开一个资源等等。。。

加载完成后通过 DataFetcher$DataCallback 接口回调。

DataCallback 中包含两个方法：

```java
void onDataReady(@Nullable T data);
void onLoadFailed(@NonNull Exception e);
```

分别代表数据加载成功或者加载失败回调。

## Encoder

------

Encoder 是个接口，在 Glide 中也是个很重要的概念，用来将给定的数据写入持久性存储介质中（文件）。

其中只有一个方法：

```java
public interface Encoder<T> {
  /**
   * Writes the given data to the given output stream and returns True if the write completed
   * successfully and should be committed.
   *
   * @param data The data to write.
   * @param file The File to write the data to.
   * @param options The put of options to apply when encoding.
   */
  boolean encode(@NonNull T data, 
                 @NonNull File file, 
                 @NonNull Options options);
}
```

比较简单，注释写的很清楚了，就是把 data 存入文件中。

数据加载完成之后会先使用 Encoder 将数据存入本地磁盘缓存文件中。
同样，Encoder 对应的实现类都是在 Glide 初始化时注册进去的。

## ResourceDecoder

------

与 Encoder 对应，**数据解码器**，用来**将原始数据解码成相应的数据类型**，针对不同的请求实现类都不同，例如通过网络请求最终获取到的是一个 InputStream，经过 ByteBufferBitmapDecoder 解码后再生成一个 Bitmap。

需要指出的是，这里解码时会根据 option 以及图片大小（如果有的话）按需加载 Bitmap，防止内存的浪费。

与 Encoder 一样，Glide 初始化时会注册很多个类型的 ResourceDecoder 实现类，图片数据获取到之后会根据不同的类型使用对应的解码器对其解码。

## Engine

作用：负责资源（Resource）的加载和管理（回收、加载策略）。

上面的 Request 中也讲到了 Engine 这个类，可理解为**执行引擎**，算是整个 Glide 的核心发动机。

Engine 负责管理请求以及活动资源、缓存等。主要关注 load 方法，这个方法主要做了如下几件事：

1. 通过请求构建 Key；
2. 从活动资源中获取资源（详见缓存章节），获取到则返回；
3. 从缓存中获取资源，获取到则直接返回；
4. 判断当前请求是否正在执行，是则直接返回；
5. 构建 EngineJob 与 DecodeJob 并执行。

关于缓存相关的都在缓存章节。下面说说 EngineJob 与 DecodeJob。

### Engine#load()

```kotlin
  /**
   * Starts a load for the given arguments.
   *
   * <p>Must be called on the main thread.
   *
   * <p>The flow for any request is as follows:
   *
   * <ul>
   *   <li>Check the current set of actively used resources, return the active resource if present,
   *       and move any newly inactive resources into the memory cache.
   *   <li>Check the memory cache and provide the cached resource if present.
   *   <li>Check the current set of in progress loads and add the cb to the in progress load if one
   *       is present.
   *   <li>Start a new load.
   * </ul>
   *
   * <p>Active resources are those that have been provided to at least one request and have not yet
   * been released. Once all consumers of a resource have released that resource, the resource then
   * goes to cache. If the resource is ever returned to a new consumer from cache, it is re-added to
   * the active resources. If the resource is evicted from the cache, its resources are recycled and
   * re-used if possible and the resource is discarded. There is no strict requirement that
   * consumers release their resources so active resources are held weakly.
   *
   * @param width The target width in pixels of the desired resource.
   * @param height The target height in pixels of the desired resource.
   * @param cb The callback that will be called when the load completes.
   */
  public <R> LoadStatus load(...){}
```

注释中介绍了load()方法的工作流程，即如何加载资源

1、检查当前正在活跃使用的资源列表，如果存在，就找到了，返回结果。并且移动任何新的不活跃资源到memory cache中。





**怎么理解 Active Resources（活跃资源）和 inactive resources 呢？Active Resources 的作用是什么？**

我们知道不论是 Active Resources 还是 memory cache，其实图片都已经在内存中，那区别是什么呢？为什么需要两个东西？

我们思考一个场景，有一张图片在 Activity1 中加载好了，然后用户打开了Activity2。。。这时候，系统是有可能把图片回收的。但用户之后可能又会返回Activity1，那如果不做任何处理的话，之前Activity1中加载的图片是不是又要重新加载了？这就是为什么有了  memory cache 后还要引入 Active Resources 的原因。不仅如此，注意这句话：Check the current set of actively used resources, return the active resource if present，这是不是说明 Active Resources 中的图片是支持复用的，2个 ImageView 显示一张图片时，并没有加载两张，而是共用了一张图片。

知道了为什么要用 Active Resources后，我们再来看，**Glide 是怎么把 Active Resources 移动到 memory cache 中的**。

为什么要使用弱引用呢？

There is no strict requirement that consumers release their resources so active resources are held weakly.

翻译：没有严格要求消费者释放他们的资源，因此活动资源被微弱地持有。

怎么理解这句话呢？因为Glide没有强制要求我们释放资源，所以我们在用完图片后，可能不会主动去释放图片，那就没有了告诉Glide图片不再使用的时机了。由于Glide想在我们不再使用图片时，将图片资源放到memory cache，所以Glide需要这么一个时机来做这件事。显然Glide选择通过用 WeakRef（弱引用）来实现。这就涉及到 WeakRef 的使用了，它可以监听 GC ，当监听到图片资源要被回收时，Glide会将资源添加到memory cache中，这样图片也就再次被Glide内部强引用了，也就做到了持久的保存在内存中了。

#### 小结

通过对load() 的分析，我们知道了 Glide 是怎么做缓存管理的。



### Resource 接口

作用：定义了一个资源接口，Engine 负责处理的资源都会实现此接口。

```kotlin
/**
 * A resource interface that wraps a particular type so that it can be pooled and reused.
 *
 * @param <Z> The type of resource wrapped by this class.
 */
public interface Resource<Z> {

  /** Returns the {@link Class} of the wrapped resource. */
  @NonNull
  Class<Z> getResourceClass();

  /**
   * Returns an instance of the wrapped resource.
   *
   * <p>Note - This does not have to be the same instance of the wrapped resource class and in fact
   * it is often appropriate to return a new instance for each call. For example, {@link
   * android.graphics.drawable.Drawable Drawable}s should only be used by a single {@link
   * android.view.View View} at a time so each call to this method for Resources that wrap {@link
   * android.graphics.drawable.Drawable Drawable}s should always return a new {@link
   * android.graphics.drawable.Drawable Drawable}.
   */
  @NonNull
  Z get();

  /**
   * Returns the size in bytes of the wrapped resource to use to determine how much of the memory
   * cache this resource uses.
   */
  int getSize();

  /**
   * Cleans up and recycles internal resources.
   *
   * <p>It is only safe to call this method if there are no current resource consumers and if this
   * method has not yet been called. Typically this occurs at one of two times:
   *
   * <ul>
   *   <li>During a resource load when the resource is transformed or transcoded before any consumer
   *       have ever had access to this resource
   *   <li>After all consumers have released this resource and it has been evicted from the cache
   * </ul>
   *
   * For most users of this class, the only time this method should ever be called is during
   * transformations or transcoders, the framework will call this method when all consumers have
   * released this resource and it has been evicted from the cache.
   */
  void recycle();
}
```



### EngineResource

作用：对 Resource 做了一次包装，主要是增加了【引用计数】的功能——在资源被使用或者释放时分别增加或减少计数，用来控制资源的回收。 



### ActiveResources

#### 作用

对 Active 资源进行一次缓存，并在资源不再使用时进行回收。Active 资源是什么？即正在使用的资源，是被强引用所持有的。

#### 实现原理

1、弱引用 ResourceWeakReference 的实现

作用：持有 Resource，Resource 会在 EngineResource 被回收后使用。假设这里不持有 Resource 的话，Resource 就会被 gc 掉了，也就不能做 ActiveResources 到 MemoryCache 的转化了。

ResourceWeakReference 是 ActiveResources 实现的基础，需要先知道其实现。

```Java
final class ActiveResources {
  
  static final class ResourceWeakReference extends WeakReference<EngineResource<?>> {
    final Key key;

    final boolean isCacheable;

    @Nullable
    Resource<?> resource;

    ResourceWeakReference(
        @NonNull Key key,
        @NonNull EngineResource<?> referent,
        @NonNull ReferenceQueue<? super EngineResource<?>> queue,
        boolean isActiveResourceRetentionAllowed) {
      //（1）ResourceWeakReference 直接持有的对象是 EngineResource，而不是图片资源。
      // 引用链：ResourceWeakReference -> EngineResource -> Resource -> 真正的图片资源
      //（2）支持传入 ReferenceQueue，用来在 EngineResource 被 gc 回收后，记录其对应的 ResourceWeakReference。
      super(referent, queue);
      this.key = Preconditions.checkNotNull(key);
      //（3）EngineResource 所持有的 Resource 也被 ResourceWeakReference 持有了（强引用），
      // 这意味着在 EngineResource 被回收时，Resource、图片资源其实还在内存中。
      // 这是实现 ActiveResources 转化为 MemoryCacheResource 的基础。
      this.resource =
          referent.isMemoryCacheable() && isActiveResourceRetentionAllowed
              ? Preconditions.checkNotNull(referent.getResource())
              : null;
      isCacheable = referent.isMemoryCacheable();
    }

    void reset() {
      //（4）释放资源
      resource = null;
      // 主动调用 clear 后，get方法将返回 null，即使此时 EngineResource 还没有被回收
      // 另外，主动调用，并不会将此引用加入到 ReferenceQueue 中
      // 并且之后 EngineResource 被回收了，也不会将 ResourceWeakReference 加入队列中
      clear();
    }
  }
}
```

ResourceWeakReference 是 ActiveResources 的内部类，通过这个类我们可以知道 4 个点，如注释所言。

2、缓存功能

缓存的基本功能有增加、删除、获取（查询），然后内部会有一个存储数据的集合，下面看源码：

```Java
package com.bumptech.glide.load.engine;

final class ActiveResources {
  // 通过前面对 ResourceWeakReference 的掌握，我们已经知道 ResourceWeakReference 中持有了 Resource
  // 存储结构，保存缓存资源，间接强引用了 Resource
  final Map<Key, ResourceWeakReference> activeEngineResources = new HashMap<>();
  // 引用队列，在 EngineResource 被 gc 回收后，会将 ResourceWeakReference 加入到队列中
  private final ReferenceQueue<EngineResource<?>> resourceReferenceQueue = new ReferenceQueue<>();

  // 添加（或者修改）缓存
  synchronized void activate(Key key, EngineResource<?> resource) {
    ResourceWeakReference toPut =
        new ResourceWeakReference(
            key, resource, resourceReferenceQueue, isActiveResourceRetentionAllowed);

    ResourceWeakReference removed = activeEngineResources.put(key, toPut);
    if (removed != null) {
      // 将之前的缓存对象reset。清除资源
      removed.reset();
    }
  }

  // 删除缓存
  synchronized void deactivate(Key key) {
    ResourceWeakReference removed = activeEngineResources.remove(key);
    if (removed != null) {
      removed.reset();
    }
  }

  // 获取缓存
  @Nullable
  synchronized EngineResource<?> get(Key key) {
    ResourceWeakReference activeRef = activeEngineResources.get(key);
    if (activeRef == null) {
      return null;
    }

    EngineResource<?> active = activeRef.get();
    if (active == null) {
      // activeRef.get() == null 理论上有两种情况：
      // 1、EngineResource 已经被 gc 掉了
      // 2、主动调用了 reset() 方法，我们已经知道这个方法内会将引用去除
      // 3、activate 传入的 Resource 为null。
     
      // 不管哪种情况，EngineResource 不再属于 ActiveResources 了，所以清除其对应的引用
      cleanupActiveReference(activeRef);
    }
    return active;
  }
  
  // 将资源从 ActiveResources 中移除
    void cleanupActiveReference(@NonNull ResourceWeakReference ref) {
    synchronized (this) {
      activeEngineResources.remove(ref.key);

      if (!ref.isCacheable || ref.resource == null) {
        return;
      }
    }

    // 构建一个新的 EngineResource，并将 ResourceWeakReference 中的 Resource 放入其中，回调给其他模块使用
    // 回调处理后面会分析，见：第 4 节（回调处理）
    EngineResource<?> newResource =
        new EngineResource<>(
            ref.resource,
            /* isMemoryCacheable= */ true,
            /* isRecyclable= */ false,
            ref.key,
            listener);
    listener.onResourceReleased(ref.key, newResource);
  }
}
```

3、监听资源被回收

创建 ActiveResources 时，会新建一个 monitorClearedResourcesExecutor 线程池来监视资源回收。

```java
final class ActiveResources {
  private final Executor monitorClearedResourcesExecutor;
  @VisibleForTesting final Map<Key, ResourceWeakReference> activeEngineResources = new HashMap<>();
  private final ReferenceQueue<EngineResource<?>> resourceReferenceQueue = new ReferenceQueue<>();
  
  ActiveResources(boolean isActiveResourceRetentionAllowed) {
    this(
        isActiveResourceRetentionAllowed,
        java.util.concurrent.Executors.newSingleThreadExecutor(
            new ThreadFactory() {
              @Override
              public Thread newThread(@NonNull final Runnable r) {
                return new Thread(
                    new Runnable() {
                      @Override
                      public void run() {
                        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                        r.run();
                      }
                    },
                    "glide-active-resources");
              }
            }));
  }

  @VisibleForTesting
  ActiveResources(
      boolean isActiveResourceRetentionAllowed, Executor monitorClearedResourcesExecutor) {
    this.isActiveResourceRetentionAllowed = isActiveResourceRetentionAllowed;
    this.monitorClearedResourcesExecutor = monitorClearedResourcesExecutor;

    monitorClearedResourcesExecutor.execute(
        new Runnable() {
          @Override
          public void run() {
            cleanReferenceQueue();
          }
        });
  }

  void cleanupActiveReference(@NonNull ResourceWeakReference ref) {
    synchronized (this) {
      activeEngineResources.remove(ref.key);

      if (!ref.isCacheable || ref.resource == null) {
        return;
      }
    }

    // 回调处理后面会分析，见：第 4 节（回调处理）
    EngineResource<?> newResource =
        new EngineResource<>(
            ref.resource,
            /* isMemoryCacheable= */ true,
            /* isRecyclable= */ false,
            ref.key,
            listener);
    listener.onResourceReleased(ref.key, newResource);
  }

  @SuppressWarnings("WeakerAccess")
  @Synthetic
  void cleanReferenceQueue() {
    // 创建死循环，不断从 resourceReferenceQueue 中读取是否有 ResourceWeakReference 被加入队列
    while (!isShutdown) {
      try {
        // 阻塞，直到获取到元素
        ResourceWeakReference ref = (ResourceWeakReference) resourceReferenceQueue.remove();
        // 清除资源，回调通知
        cleanupActiveReference(ref);

      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      }
    }
  }
}
```

ActiveResources 会创建一个线程池，并执行死循环不断从 resourceReferenceQueue 中读取 ResourceWeakReference 元素，如果读取到就意味着 ResourceWeakReference 所指向的对象被回收了，自然也就没有外部的引用了，可进行清除和后续回调处理。

4、回调处理

第 2 和 3 节已经提到在资源（EngineResource）被回收后，会调用 cleanupActiveReference 并回调 listener.onResourceReleased(ref.key, newResource)。

```kotlin
final class ActiveResources {
    interface ResourceListener {
    void onResourceReleased(Key key, EngineResource<?> resource);
  }
  
   private ResourceListener listener;
  
    void setListener(ResourceListener listener) {
    synchronized (listener) {
      synchronized (this) {
        this.listener = listener;
      }
    }
  }
  
void cleanupActiveReference(@NonNull ResourceWeakReference ref) {
    synchronized (this) {
      activeEngineResources.remove(ref.key);

      if (!ref.isCacheable || ref.resource == null) {
        return;
      }
    }

    EngineResource<?> newResource =
        new EngineResource<>(
            ref.resource,
            /* isMemoryCacheable= */ true,
            /* isRecyclable= */ false,
            ref.key,
            listener);
    listener.onResourceReleased(ref.key, newResource);
  }
}
```

下面是 setListener 和 listener.onResourceReleased 的实现：

Engine 实现了 ResourceListener 接口，并在 Engine 构造方法中，设置了 listener，

```kotlin
public class Engine
    implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
          
  Engine(
      MemoryCache cache,
      DiskCache.Factory diskCacheFactory,
      GlideExecutor diskCacheExecutor,
      GlideExecutor sourceExecutor,
      GlideExecutor sourceUnlimitedExecutor,
      GlideExecutor animationExecutor,
      Jobs jobs,
      EngineKeyFactory keyFactory,
      ActiveResources activeResources,
      EngineJobFactory engineJobFactory,
      DecodeJobFactory decodeJobFactory,
      ResourceRecycler resourceRecycler,
      boolean isActiveResourceRetentionAllowed) {
    
    if (activeResources == null) {
      activeResources = new ActiveResources(isActiveResourceRetentionAllowed);
    }
    this.activeResources = activeResources;
    activeResources.setListener(this);
  }
  
  // 回调方法：将资源从 ActiveResources 移到 MemoryCache 中
  @Override
  public void onResourceReleased(Key cacheKey, EngineResource<?> resource) {
    activeResources.deactivate(cacheKey);
    if (resource.isMemoryCacheable()) {
      // 支持内存缓存，缓存起来
      cache.put(cacheKey, resource);
    } else {
      // 不支持内存缓存，回收 resource
      resourceRecycler.recycle(resource, /* forceNextFrame= */ false);
    }
  }  
}
```



#### Why？

前面分析了 ActiveResources 的作用和实现原理，但是我们还没有回答一个问题——为什么要这么做？我们知道，ActiveResources 本质也是内存缓存，cache 也是属于内存缓存，那为什么需要两次内存缓存呢？区别又是什么？

通过对 ActiveResources 的分析，我们知道 ActiveResources 所管理的资源是被外部（应用层）正在使用的，也就是被外部强引用的。或者是刚使用完，已经不在被强引用，但是还没有被 gc 回收。这类资源，正在使用或者刚使用完，使用频率会更高，所以做了一次缓存分级。相较于全部使用MemoryCache，这类资源可以更块地加载到。

特别注意：ActiveResources 所管理的资源，指的是 EngineResource 和 Resource 实现类，并不是指真正的图片资源

#### 小结

Glide 为了实现更快的加载图片资源，在内存缓存的基础上增加了一个 ActiveResources 缓存，用来保存正在使用或刚用完但还没有被 gc 的资源。然后基于 WeakReference（强引用持有Resource） 和可监听对象被回收的机制，实现了资源从 ActiveResources 到 MemoryCache 的转化。

为什么要使用 WeakReference？是因为 Glide 想要知道外部是否正在使用资源（资源追踪），并在未使用时转化为 MemoryCache。那要知道是否在使用有两种方法，一是用户主动调用SDK告知已经释放，但用户不一定会主动调用 release 方法，所以这种方法不可靠；而 WeakReference + ReferenceQueue 提供了监听对象回收的能力，符合 Glide 此场景的需求。

Glide 在这里使用 WeakReference 的主要是为了追踪资源，即知道什么时候资源不在被使用。跟解决 Handler 持有 Activity 引起内存泄漏时的目的是不同的，需要注意。



## EngineJob

------

这个主要用来执行 DecodeJob 以及管理加载完成的回调，各种监听器，没有太多其他的东西。

## DecodeJob

------

负责从缓存或数据源中加载原始数据并通过解码器转换为相应的资源类型（Resource）。DecodeJob 实现了 Runnable 接口，由 EngineJob 将其运行在指定线程池中。

首次加载一张图片资源时，资源加载完成后会先存入到本地缓存文件中，然后再从文件中获取。

上面已经说过，图片的加载最终是通过 DataFetcher 来实现，但是此处并没有直接这么调用，考虑到缓存文件，这里面使用的是 DataFetcherGenerator，其有三个实现类，对应不同的加载方式，这里就不多做介绍了，只需要知道它会根据资源类型去 Glide 中获取已注册的 DataFetcher ，然后**通过 DataFetcher#loadData 方法获取原始数据**，获取完成后使用 Encoder 将数据存入磁盘缓存文件中，同时使用对应的解码器将原始数据转换为相应的资源文件，这样整个流程就差不多结束了。



# 缓存模块

关于缓存的获取、数据加载相关的逻辑在 Engine#load 方法中。先来看看缓存流程，流程如下图：

<img src="images/Glide解析/10.png" alt="image-20240328170713559" style="zoom:50%;" />

Glide 实例化时会实例化三个缓存相关的类以及一个计算缓存大小的类：

```java
//根据当前机器参数计算需要设置的缓存大小
MemorySizeCalculator calculator = new MemorySizeCalculator(context);
//创建 Bitmap 池
if (bitmapPool == null) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        int size = calculator.getBitmapPoolSize();
        bitmapPool = new LruBitmapPool(size);
    } else {
        bitmapPool = new BitmapPoolAdapter();
    }
}
//创建内存缓存
if (memoryCache == null) {
    memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
}
//创建磁盘缓存
if (diskCacheFactory == null) {
    diskCacheFactory = new InternalCacheDiskCacheFactory(context);
}
```

除此之外 Engine 中还有一个 ActiveResources 作为第一级缓存。下面分别来介绍一下。

## ActiveResources

------

ActiveResources 是**第一级缓存**，管理的资源是正在使用的或者最近使用的（刚用完还没被 gc 回收的），没有大小限制。类路径：

```css
com.bumptech.glide.load.engine.ActiveResources
```

Engine#load 方法中构建好 Key 之后第一件事就是去这个缓存中获取资源，获取到则直接返回，获取不到才继续从其他缓存中寻找。

当资源加载成功，或者通过缓存命中资源后都会将其放入 ActiveResources 中，资源被释放时移除出 ActiveResources 。

ActiveResources 中通过一个 Map 来存储数据，数据保存在一个**弱引用**（WeakReference）中。

刚刚说的 activeResource 使用一个 Map<Key, WeakReference<EngineResource<?>>> 来存储的，此外还有一个引用队列：

```php
ReferenceQueue<EngineResource<?>> resourceReferenceQueue;
```

每当向 activeResource 中添加一个 WeakReference 对象时都会将 resourceReferenceQueue 和这个 WeakReference 关联起来，用来跟踪这个 WeakReference 的 gc，一旦这个弱引用持有的对象被 gc 掉，就会将它从 activeResource 中移除。

那么 ReferenceQueue 具体是在何时去判断 WeakReference 是否被 gc 了呢，Handler 机制大家应该都知道，但不知道大家有没有用过 MessageQueue.IdleHandler ，可以调用 MessageQueue#addIdleHandler 添加一个 MessageQueue.IdleHandler 对象，Handler 会在**线程空闲时调用这个方法**。resourceReferenceQueue 在创建时会创建一个 Engine#RefQueueIdleHandler 对象并将其添加到当前线程的 MessageQueue 中，ReferenceQueue 会在 IdleHandler 回调的方法中去判断 activeResource 中的 WeakReference 是不是被 gc 了，如果是，则将引用从 activeResource 中移除，代码如下：

```java
//MessageQueue 中的消息暂时处理完回调
@Override
public boolean queueIdle() {
    ResourceWeakReference ref = (ResourceWeakReference) queue.poll();
    if (ref != null) {
        activeResources.remove(ref.key);
    }
    //返回 true，表示下次处理完仍然继续回调
    return true;
}
```

## MemorySizeCalculator

------

这个类是用来计算 BitmapPool 、ArrayPool 以及 MemoryCache **大小**的。计算方式如下：

```java
//默认为 4MB，如果是低内存设备则在此基础上除以二
arrayPoolSize =
        isLowMemoryDevice(builder.activityManager)
                ? builder.arrayPoolSizeBytes / LOW_MEMORY_BYTE_ARRAY_POOL_DIVISOR
                : builder.arrayPoolSizeBytes;
//其中会先获取当前进程可使用内存大小，
//然后通过判断是否是否为低内存设备乘以相应的系数，
//普通设备是乘以 0.4，低内存为 0.33，这样得到的是 Glide 可使用的最大内存阈值 maxSize
int maxSize =
        getMaxSize(
                builder.activityManager, builder.maxSizeMultiplier, builder.lowMemoryMaxSizeMultiplier);

int widthPixels = builder.screenDimensions.getWidthPixels();
int heightPixels = builder.screenDimensions.getHeightPixels();
//计算一张格式为 ARGB_8888 ，大小为屏幕大小的图片的占用内存大小
//BYTES_PER_ARGB_8888_PIXEL 值为 4
int screenSize = widthPixels * heightPixels * BYTES_PER_ARGB_8888_PIXEL;

int targetBitmapPoolSize = Math.round(screenSize * builder.bitmapPoolScreens);

int targetMemoryCacheSize = Math.round(screenSize * builder.memoryCacheScreens);
//去掉 ArrayPool 占用的内存后还剩余的内存
int availableSize = maxSize - arrayPoolSize;

if (targetMemoryCacheSize + targetBitmapPoolSize <= availableSize) {
    //未超出内存限制
    memoryCacheSize = targetMemoryCacheSize;
    bitmapPoolSize = targetBitmapPoolSize;
} else {
    //超出内存限制
    float part = availableSize / (builder.bitmapPoolScreens + builder.memoryCacheScreens);
    memoryCacheSize = Math.round(part * builder.memoryCacheScreens);
    bitmapPoolSize = Math.round(part * builder.bitmapPoolScreens);
}
```



## BitmapPool

------

BitmapPool 是用来**复用 Bitmap** 从而避免重复创建 Bitmap 而带来的内存浪费，Glide 通过 SDK 版本不同创建不同的 BitmapPool 实例，版本低于 Build.VERSION_CODES.HONEYCOMB(11) 实例为 BitmapPoolAdapter，其中的方法体几乎都是空的，也就是是个实例不做任何缓存。否则实例为 LruBitmapPool，先来看这个类。

### LruBitmapPool

------

LruBitmapPool 中没有做太多的事，主要任务都交给了 **LruPoolStrategy**，这里只是做一些缓存大小管理、封装、日志记录等等操作。

每次调用 put 缓存数据时都会调用 trimToSize 方法判断已缓存内容是否大于设定的最大内存，如果大于则使用 LruPoolStrategy#removeLast 方法逐步移除，直到内存小于设定的最大内存为止。

LruPoolStrategy 有两个实现类：SizeConfigStrategy 以及 AttributeStrategy，根据系统版本创建不同的实例，这两个差异不大，KITKAT 之后使用的都是 SizeConfigStrategy，这个比较重要。

#### SizeConfigStrategy

------

SizeConfigStrategy 顾名思义，是通过 Bitmap 的 size 与 Config 来当做 key 缓存 Bitmap，Key 也会通过 KeyPool 来缓存在一个队列（Queue）中。

与 AttributeStrategy 相同的是，其中都使用 Glide 内部自定义的数据结构：**GroupedLinkedMap** 来存储 Bitmap。

当调用 put 方法缓存一个 Bitmap 时会先通过 Bitmap 的大小以及 Bitmap.Config 创建（从 KeyPool 中获取）Key，然后将这个 Key 与 Bitmap 按照键值对的方式存入 GroupedLinkedMap 中。

此外其中还包含一个 sortedSizes，这是一个 HashMap，Key 对应 put 进来的 Bitmap.Config，value 对应一个 TreeMap，TreeMap 中记录着每一个 size 的 Bitmap 在当前缓存中的个数，即 put 时加一，get 时减一。

TreeMap 是**有序的**数据结构，当需要通过 Bitmap 的 size 与 Config 从缓存中获取一个 Biamp 时未必会一定要获取到 size 完全相同的 Bitmap，由于 TreeMap 的特性，调用其 ceilingKey 可以获取到一个相等或大于当前 size 的一个最小值，用这个 Key 去获取 Bitmap，然后重置一下大小即可。

重点看一下 GroupedLinkedMap，这是 Glide 为了 实现 [LRU 算法](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FLRU)自定义的一个数据结构，看名字是已分组的链表 Map？看一下下面的图就明白了：

![img](images/Glide解析/11.png)

GroupedLinkedMap

其中包含三种数据结构：哈希表（HashMap）、[循环链表](https://links.jianshu.com/go?to=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E5%BE%AA%E7%8E%AF%E9%93%BE%E8%A1%A8)以及列表（ArrayList）。
这个结构其实类似 Java 里提供的 [LinkedHashMap](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fapi%2Fjava%2Futil%2FLinkedHashMap.html) 类。

循环链表是通过内部类 GroupedLinkedMap$LinkedEntry 实现的，其中除了定义了链表结构需要的上下两个节点信息之外还包含着一个 Key 与一个 Values，定义如下：

```java
private static class LinkedEntry<K, V> {
    private final K key;
    private List<V> values;
    LinkedEntry<K, V> next;
    LinkedEntry<K, V> prev;
    
    ...
}
```

其实就是将 HashMap 的 Values 使用**链表**串了起来，每个 Value 中又存了个 **List**。

调用 put 方法时会先根据 Key 去这个 Map 中获取 LinkedEntry，获取不到则创建一个，并且加入到链表的尾部，然后将 value （也就是 Bitmap）存入 LinkedEntry 中的 List 中。

所以这里说的分组指的是通过 Key 来对 Bitmap 进行分组，对于同一个 Key（size 与 config 都相同）的 Bitmap 都会存入同一个 LinkedEntry 中。

调用 get 方法获取 Bitmap 时会先通过 Key 去 keyToEntry 中获取 LinkedEntry 对象，获取不到则创建一个，然后将其加入到链表头部，此时已经有了 LinkedEntry 对象，调用 LinkedEntry#removeLast 方法返回并删除 List 中的最后一个元素。

通过上面两步可以看到之所以使用链表是为了**支持 LRU 算法**，最近使用的 Bitmap 都会移动到链表的前端，使用次数越少就越靠后，当调用 removeLast 方法时就直接调用链表最后一个元素的 removeLast 方法移除元素。

好了 BitmapPool 大概就这么多内容，总结一下：

1. BitmapPool 大小通过 MemorySizeCalculator 设置；
2. 使用 LRU 算法维护 BitmapPool ；
3. Glide 会根据 Bitmap 的大小与 Config 生成一个 Key；
4. Key 也有自己对应的对象池，使用 Queue 实现；
5. 数据最终存储在 GroupedLinkedMap 中；
6. GroupedLinkedMap 使用哈希表、循环链表、List 来存储数据。

## MemoryCache

------

如果从 ActiveResources 中没获取到资源则开始从 MemoryCache 寻找。

内存缓存同样使用 **LRU 算法**，实现类为 LruResourceCache，继承自 LruCache。

### LruResourceCache

LruResourceCache 是在 LruCache 的基础上，拓展了一些回调方法，比如 trimMemory(int level) 回调，以及 ResourceRemovedListener 接口，当有资源从 MemoryCache 中被移除时会回调其中的方法，Engine 中接收到这个消息后就会进行 Bitmap 的回收操作。

```kotlin
public class LruResourceCache extends LruCache<Key, Resource<?>> implements MemoryCache {
}
```

缓存功能主要是在 LruCache 实现的。

### LruCache

```java
public class LruCache<T, Y> {
  private final Map<T, Entry<Y>> cache = new LinkedHashMap<>(100, 0.75f, true);
  private final long initialMaxSize;
  // 最大缓存
  private long maxSize;
  // 当前已经缓存的内存大小
  private long currentSize;

    public synchronized Y put(@NonNull T key, @Nullable Y item) {
    final int itemSize = getSize(item);
      // 1、itemSize >= maxSize，不缓存
    if (itemSize >= maxSize) {
      onItemEvicted(key, item);
      return null;
    }

    if (item != null) {
      // 2、计算新的 currentSize
      currentSize += itemSize;
    }
     // 3、put 新的 item
    @Nullable Entry<Y> old = cache.put(key, item == null ? null : new Entry<>(item, itemSize));
    if (old != null) {
      // 如果是替换，将旧的大小减去
      currentSize -= old.size;

      if (!old.value.equals(item)) {
        onItemEvicted(key, old.value);
      }
    }
      // 触发回收
    evict();

    return old != null ? old.value : null;
  }
  
  // 回收处理。遍历 item 直到 currentSize 小于 maxSize。
    protected synchronized void trimToSize(long size) {
    Map.Entry<T, Entry<Y>> last;
    Iterator<Map.Entry<T, Entry<Y>>> cacheIterator;
      // 如果 currentSize 大于 maxSize
    while (currentSize > size) {
      cacheIterator = cache.entrySet().iterator();
      last = cacheIterator.next();
      final Entry<Y> toRemove = last.getValue();
      currentSize -= toRemove.size;
      final T key = last.getKey();
      // 移除首个 item，根据 LinkedHashMap accessOrder = true 时的特性，首个 item 时最近最少使用的
      cacheIterator.remove();
      onItemEvicted(key, toRemove.value);
    }
  }

  private void evict() {
    trimToSize(maxSize);
  }
}  
```

Java 集合里面提供了一个很好的用来实现 LRU 算法的数据结构——**LinkedHashMap**。其基于 HashMap 实现，同时又将 HashMap 中的 Entity 串成了一个双向链表。LruCache 中就是使用这个集合来缓存数据，主要是在 LinkedHashMap 的基础上又提供了对内存的管理操作。

Glide LruCache 的实现策略是根据缓存资源大小来决定是否回收（移除item）的，另一种常见的实现 LruCache 方式是按照 LinkedHashMap 中 size 数量去回收的，显然 Glide 的这种实现更合适些，这样如果每张图片都很小的话，就可以缓存更多张了。

## 磁盘缓存

------

缓存路径默认为 Context#getCacheDir() 下面的 image_manager_disk_cache 文件夹，默认缓存大小为 250MB。

磁盘缓存实现类由 InternalCacheDiskCacheFactory 创建，最终会通过缓存路径及缓存文件夹最大值创建一个 DiskLruCacheWrapper 对象。

DiskLruCacheWrapper 实现了 DiskCache 接口，接口主要的代码如下：

```java
File get(Key key);
void put(Key key, Writer writer);
void delete(Key key);
void clear();
```

可以看到其中提供了作为一个缓存类必须的几个方法，并且文件以 Key 的形式操作。

**SafeKeyGenerator** 类用来将 Key 对象转换为字符串，Key 不同的实现类生成 Key 的方式也不同，一般来说会通过图片宽高、加密解码器、引擎等等生成一个 byte[] 然后再转为字符串，以此来保证图片资源的**唯一性**。

另外，在向磁盘写入文件时（put 方法）会使用**重入锁**来同步代码，也就是 DiskCacheWriteLocker 类，其中主要是对 **ReentrantLock** 的包装。

DiskLruCacheWrapper 顾名思义也是一个包装类，包装的是 **DiskLruCache**。

### DiskLruCache

------

这里考虑一个问题，磁盘缓存同样使用的是 LRU 算法，但文件是存在磁盘中的，如何在 APP 启动之后准确的按照使用次数排序读取缓存文件呢？

Glide 是使用一个**日志清单文件**来保存这种顺序，DiskLruCache 在 APP 第一次安装时会在缓存文件夹下创建一个 **journal** 日志文件来记录图片的添加、删除、读取等等操作，后面每次打开 APP 都会读取这个文件，把其中记录下来的缓存文件名读取到 LinkedHashMap 中，后面每次对图片的操作不仅是操作这个 LinkedHashMap 还要记录在 journal 文件中.
journal 文件内容如下图：

![img](images/Glide解析/12.png)

开头的 libcore.io.DiskLruCache 是魔数，用来标识文件，后面的三个 1 是版本号 valueCount 等等，再往下就是图片的操作日志了。

DIRTY、CLEAN 代表操作类型，除了这两个还有 REMOVE 以及 READ，紧接着的一长串字符串是文件的 Key，由上文提到的 SafeKeyGenerator 类生成，是由图片的宽、高、加密解码器等等生成的 SHA-256 散列码 后面的数字是图片大小。

根据这个字符串就可以在同目录下找到对应的图片缓存文件，那么打开缓存文件夹即可看到上面日志中记录的文件：

![img](images/Glide解析/13.png)

缓存文件列表

可以看到日志文件中记录的缓存文件就在这个文件夹下面。

由于涉及到磁盘缓存的外部排序问题，所以相对而言磁盘缓存比较复杂。

那么 Glide 的缓存模块至此就结束了，主要是 BitmapPool 中的数据结构以及磁盘缓存比较复杂，其他的倒也不是很复杂。



# 生命周期绑定原理

1、实现原理

在Activity中添加无UI的Fragment，通过Fragment接收Activity传递的生命周期。Fragment和RequestManager基于LifeCycle接口建立联系，并传递生命周期事件，实现生命周期感知。

如何绑定生命周期

在调用Glide.with(Activity activity)的时候，我们跟一下流程

```kotlin
 // with入口
 public static RequestManager with(@NonNull FragmentActivity activity) {
     return getRetriever(activity).get(activity);
 }

 // 此处拿到对应的 FragmentManager，为生成Fragment做准备
 public RequestManager get(@NonNull FragmentActivity activity) {
     if(Util.isOnBackgroundThread()) {
         return this.get(activity.getApplicationContext());
     } else {
         assertNotDestroyed(activity);
         android.support.v4.app.FragmentManager fm = activity.getSupportFragmentManager();
         return this.supportFragmentGet(activity, fm, (Fragment)null, isActivityVisible(activity));
     }
 }
 
 private RequestManager supportFragmentGet(@NonNull Context context, @NonNull android.support.v4.app.FragmentManager fm, @Nullable Fragment parentHint, boolean isParentVisible) {
 	// current就是一个无UI的Fragment实例
     SupportRequestManagerFragment current = this.getSupportRequestManagerFragment(fm, parentHint, isParentVisible);
     RequestManager requestManager = current.getRequestManager();
     if(requestManager == null) {
         Glide glide = Glide.get(context);
 		// 将Fragment的LifeCycle传入RequestManager中，建立起来联系
         requestManager = this.factory.build(glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
         current.setRequestManager(requestManager);
     }

     return requestManager;
 }

 //RequestManager的构造方法中绑定LifeCycle，将自己的引用存入LifeCycle，调用LifeCycle的生命周期时进行回调
 lifecycle.addListener(this);

```

 1. Glide绑定Activity时，生成一个无UI的Fragment
 2. 将无UI的Fragment的LifeCycle传入到RequestManager中
 3. 在RequestManager的构造方法中，将RequestManager存入到之前传入的Fragment的LifeCycle，在回调LifeCycle时会回调到

如何通过Fragment的生命周期回调调用Glide的对应方法

通过Fragment的回调调用到Glide的RequestManager的对应的方法即可执行不同的操作，主要绑定的三个方法为：`onStart()`,`onStop()`,`onDestroy()`。回调的源码：

```kotlin
 //RequestManager的构造方法中绑定LifeCycle，将自己的引用存入LifeCycle，调用LifeCycle的生命周期时进行回调
 //这个this是RequestManager的实例
 lifecycle.addListener(this);

 // onDestory的回调示例
 void onDestroy() {
     this.isDestroyed = true;
     Iterator var1 = Util.getSnapshot(this.lifecycleListeners).iterator();

     while(var1.hasNext()) {
         LifecycleListener lifecycleListener = (LifecycleListener)var1.next();
         lifecycleListener.onDestroy();
     }

 }

 //下面看一下RequestManager里面的onDestory方法，里面主要做一些解绑和清除操作
 public void onDestroy() {
     this.targetTracker.onDestroy();
     Iterator var1 = this.targetTracker.getAll().iterator();

     while(var1.hasNext()) {
         Target<?> target = (Target)var1.next();
         this.clear(target);
     }

     this.targetTracker.clear();
     this.requestTracker.clearRequests();
     this.lifecycle.removeListener(this);
     this.lifecycle.removeListener(this.connectivityMonitor);
     this.mainHandler.removeCallbacks(this.addSelfToLifecycle);
     this.glide.unregisterRequestManager(this);
 }
```





# Glide 初始化流程分析

从 Glide.with() 到 Glide.get(context) 获取 Glide 实例

```kotlin
  	public static RequestManager with(@NonNull Context context) {
      // 步骤1：调用 getRetriever 获得 RequestManagerRetriever 对象 - 单例实现
      // 步骤2：调用RequestManagerRetriever实例的get()获取RequestManager对象 & 绑定图片加载的生命周期 ->>分析2
    	return getRetriever(context).get(context);
  	}  

  	private static RequestManagerRetriever getRetriever(@Nullable Context context) {
    	Preconditions.checkNotNull(context, DESTROYED_ACTIVITY_WARNING);
    	return Glide.get(context).getRequestManagerRetriever();
  	}

// 获取 Glide 单例，双重校验锁（Double checked locking）实现。
public static Glide get(@NonNull Context context) {
    if (glide == null) {
      // 分析1：GeneratedAppGlideModule的作用
      GeneratedAppGlideModule annotationGeneratedModule =
          getAnnotationGeneratedGlideModules(context.getApplicationContext());
      synchronized (Glide.class) {
        if (glide == null) {
          checkAndInitializeGlide(context, annotationGeneratedModule);
        }
      }
    }

    return glide;
  }

  static void checkAndInitializeGlide(
      @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
    ...
    isInitializing = true;
    try {
      initializeGlide(context, generatedAppGlideModule);
    } finally {
      isInitializing = false;
    }
  }

  private static void initializeGlide(
      @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
    initializeGlide(context, new GlideBuilder(), generatedAppGlideModule);
  }

  private static void initializeGlide(
      @NonNull Context context,
      @NonNull GlideBuilder builder,
      @Nullable GeneratedAppGlideModule annotationGeneratedModule) {
    // 分析0：无论with()传入的参数是什么，用于初始化 Glide 的 Context 总是 ApplicationContext
    Context applicationContext = context.getApplicationContext();
    
    // 分析1.1-start
    List<GlideModule> manifestModules = Collections.emptyList();
    if (annotationGeneratedModule == null || annotationGeneratedModule.isManifestParsingEnabled()) {
      // 解析清单文件配置的自定义GlideModule的metadata标签，返回一个GlideModule集合
      manifestModules = new ManifestParser(applicationContext).parse();
    }

    if (annotationGeneratedModule != null
        && !annotationGeneratedModule.getExcludedModuleClasses().isEmpty()) {
      Set<Class<?>> excludedModuleClasses = annotationGeneratedModule.getExcludedModuleClasses();
      Iterator<GlideModule> iterator = manifestModules.iterator();
      while (iterator.hasNext()) {
        GlideModule current = iterator.next();
        if (!excludedModuleClasses.contains(current.getClass())) {
          continue;
        }
        ...
        iterator.remove();
      }
    }
...
    RequestManagerRetriever.RequestManagerFactory factory =
        annotationGeneratedModule != null
            ? annotationGeneratedModule.getRequestManagerFactory()
            : null;
    builder.setRequestManagerFactory(factory);
    for (GlideModule module : manifestModules) {
      module.applyOptions(applicationContext, builder);
    }
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.applyOptions(applicationContext, builder);
    }
    // 分析1.1-end
    
    // 分析2：GlideBuilder的作用
    // GlideBuilder会为Glide设置一默认配置，如：Engine，RequestOptions，GlideExecutor，MemorySizeCalculator
    Glide glide = builder.build(applicationContext, manifestModules, annotationGeneratedModule);
    // 分析3
    applicationContext.registerComponentCallbacks(glide);
    Glide.glide = glide;
  }
```

分析0：无论with()传入的参数是什么，用于初始化 Glide 的 Context 总是 ApplicationContext

### 分析1：GeneratedAppGlideModule的作用

支持应用自定义 AppGlideModule（用注解@GlideModule标记） 来配置 Glide。比如项目中常用的给 Glide 添加 OkHttp 日志。

分析1.1-start —— 分析1.1-end：

GlideModule接口 和 AppGlideModule 作用一样，都是为了给 Glide 作自定义配置。

> 注意：这里的 GlideModule 是接口，跟注解@GlideModule是两个东西，Glide 用了相同的命名）

但是 GlideModule 已经被 AppGlideModule 替代了。因为 GlideModule 是通过 Manifest 配置的，在运行时还需要解析 xml。而 AppGlideModule 则是利用了编译时注解生成类，在运行时通过反射生成类，然后获取到我们的配置类。

### 分析2：GlideBuilder的作用

通过 build() 构造 Glide 所需的对象实例，比如RequestManagerRetriever、Engine、MemoryCache、BitmapPool、ArrayPool等，最后通过参数传给 Glide 构造方法，并完成 Glide 的创建。

### 分析3：Glide 实现 ComponentCallbacks2 接口

Glide 实现了 ComponentCallbacks2 接口来实现内存的管理，系统内存变化时回调到 onTrimMemory 方法。

# 图片加载源码分析

`Glide`源码较为难懂、难分析的其中一个原因是：许多对象都是很早之前就初始化好，而并非在使用前才初始化。所以当真正使用该对象时，开发者可能已经忘记是在哪里初始化、该对象是作什么用的了。**所以本文会在每个阶段进行一次总结，而读者则需要经常往返看该总结，从而解决上述问题。**

下面将根据 `Glide` 加载图片的使用步骤分析源码：

```csharp
Glide.with(this).load(url).into(imageView);
```

## with()

- 定义：`Glide` 类中的静态方法，根据传入 不同参数 进行 方法重载
- 作用：
  1. 得到一个`RequestManager`对象
  2. 根据传入`with()`方法的参数 **将Glide图片加载的生命周期与Activity/Fragment的生命周期进行绑定，从而实现自动执行请求，暂停操作**
- 下面先说明一些重要对象名

![img](https:////upload-images.jianshu.io/upload_images/944365-057fc1354b98ec1c.png?imageMogr2/auto-orient/strip|imageView2/2/w/770/format/webp)

问题

Q：为什么支持传入不同的参数？

Q：不同的参数是被怎么处理的

- 具体源码

### 分析1：从 Glide.with() 开始

```java
public class Glide {
    // with()重载种类非常多，根据传入的参数可分为：
    // 1. 非Application类型的参数（Activity & Fragment  ）
    // 2. Application类型的参数（Context）
    // 下面将详细分析

 		// 参数1：Application类型
  	public static RequestManager with(@NonNull Context context) {
      // 步骤1：调用 getRetriever 获得 RequestManagerRetriever 对象 - 单例实现
      // 步骤2：调用RequestManagerRetriever实例的get()获取RequestManager对象 & 绑定图片加载的生命周期 ->>分析2
    	return getRetriever(context).get(context);
  	}
    
  	// 参数2：非Application类型（Activity & Fragment ）
    public static RequestManager with(@NonNull FragmentActivity activity) {
    	return getRetriever(activity).get(activity);
  	}
    
  	public static RequestManager with(@NonNull Fragment fragment) {
    	return getRetriever(fragment.getContext()).get(fragment);
  	}
      
   	public static RequestManager with(@NonNull View view) {
  	  return getRetriever(view.getContext()).get(view);
 	 	}
  	
  	private static RequestManagerRetriever getRetriever(@Nullable Context context) {
    	Preconditions.checkNotNull(context, DESTROYED_ACTIVITY_WARNING);
    	return Glide.get(context).getRequestManagerRetriever();
  	}
}
```

将请求参数转交给 RequestManagerRetriever 处理，并通过 RequestManagerRetriever.get(Context | FragmentActivity | Fragment | View) 获取 RequestManager。

Glide.get(context) 在【Glide初始化流程分析】一节分析。

### 分析2：RequestManagerRetriever 的作用

```kotlin

	// 作用：
  // 1. 获取RequestManager对象
  // 2. 将图片加载的生命周期与Activity/Fragment的生命周期进行绑定
public class RequestManagerRetriever implements Handler.Callback {
...
   public RequestManager get(Context context) {
     // 在子类中找是否有匹配的类型，优先使用 FragmentActivity、ContextWrapper 来创建 RequestManager
         if (context == null) {
      throw new IllegalArgumentException("You cannot start a load on a null Context");
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
      if (context instanceof FragmentActivity) {
        return get((FragmentActivity) context);
      } else if (context instanceof ContextWrapper
          && ((ContextWrapper) context).getBaseContext().getApplicationContext() != null) {
        return get(((ContextWrapper) context).getBaseContext());
      }
    }
    // 调用getApplicationManager（）最终获取一个RequestManager对象 ->>分析2
    // 因为Application对象的生命周期即App的生命周期
    // 所以Glide加载图片的生命周期是自动与应用程序的生命周期绑定，不需要做特殊处理（若应用程序关闭，Glide的加载也会终止）
    return getApplicationManager(context);
    }

  @NonNull
  public RequestManager get(@NonNull FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    }
    // 判断activity是否已经销毁
    assertNotDestroyed(activity);
    frameWaiter.registerSelf(activity);
    boolean isActivityVisible = isActivityVisible(activity);
    Glide glide = Glide.get(activity.getApplicationContext());
    return lifecycleRequestManagerRetriever.getOrCreate(
        activity,
        glide,
      	// 获取 activity 的 Lifecycle
        activity.getLifecycle(),
      	// 获取FragmentManager 对象
        activity.getSupportFragmentManager(),
        isActivityVisible);
  }
  
    public RequestManager get(Fragment fragment) {
      // 逻辑同上
    }
}
```

### 小结

1. 创建 Glide 实例（第一次）
2. 得到一个`RequestManager`对象（其实现了 LifecycleListener 接口），因此可以响应 `Activity` 和 `Fragment` 的生命周期方法。

## load()

RequestManager#load() 会预先创建好对图片进行一系列操作（加载、编解码、转码）的对象，并全部封装到 `DrawableTypeRequest` 对象中。

> 1. `Glide` 支持加载 图片的URL字符串、图片本地路径等，因此`RequestManager` 类 存在`load()`的重载
> 2. 此处主要讲解 最常见的加载图片 `URL` 字符串的`load()`，即`load(String url)`

- 具体过程

```java
public class RequestManager implements LifecycleListener {

    // 仅贴出关键代码
    ...
      
  public RequestBuilder<Drawable> load(@Nullable String string) {
    return asDrawable().load(string);
  }
  
  public RequestBuilder<Drawable> asDrawable() {
    return as(Drawable.class);
  }
  
  public <ResourceType> RequestBuilder<ResourceType> as(
      @NonNull Class<ResourceType> resourceClass) {
    return new RequestBuilder<>(glide, this, resourceClass, context);
  }
  
<-- 分析2：loadGeneric（）-->
    private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {

        ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
        // 创建第1个ModelLoader对象；作用：加载图片
        // Glide会根据load()方法传入不同类型参数，得到不同的ModelLoader对象
        // 此处传入参数是String.class，因此得到的是StreamStringLoader对象（实现了ModelLoader接口）

        ModelLoader<T, ParcelFileDescriptor> fileDescriptorModelLoader = Glide.buildFileDescriptorModelLoader(modelClass, context);
         // 创建第2个ModelLoader对象，作用同上：加载图片
        // 此处得到的是FileDescriptorModelLoader对象

        return optionsApplier.apply(
                new DrawableTypeRequest<T>(modelClass, streamModelLoader, fileDescriptorModelLoader, context,
                        glide, requestTracker, lifecycle, optionsApplier));
            // 创建DrawableTypeRequest对象 & 传入刚才创建的ModelLoader对象 和 其他初始化配置的参数
            // DrawableTypeRequest类分析 ->>分析3
    }

    ...

<-- 分析3：DrawableTypeRequest类（）-->
public class DrawableTypeRequest<ModelType> extends DrawableRequestBuilder<ModelType> implements DownloadOptions {

// 关注1：构造方法
      DrawableTypeRequest(Class<ModelType> modelClass, ModelLoader<ModelType, InputStream> streamModelLoader,
            ModelLoader<ModelType, ParcelFileDescriptor> fileDescriptorModelLoader, Context context, Glide glide,
            RequestTracker requestTracker, Lifecycle lifecycle, RequestManager.OptionsApplier optionsApplier) {
        super(context, modelClass,
                buildProvider(glide, streamModelLoader, fileDescriptorModelLoader, GifBitmapWrapper.class,
                        GlideDrawable.class, null),
                glide, requestTracker, lifecycle);
      // 调用buildProvider()方法 -->分析4
      // 并把上述创建的streamModelLoader和fileDescriptorModelLoader等参数传入到buildProvider()中

// 关注2：DrawableTypeRequest类主要提供2个方法： asBitmap() & asGif() 

    // asBitmap()作用：强制加载 静态图片
    public BitmapTypeRequest<ModelType> asBitmap() {
        return optionsApplier.apply(new BitmapTypeRequest<ModelType>(this, streamModelLoader,
                fileDescriptorModelLoader, optionsApplier));
        // 创建BitmapTypeRequest对象
    }

    // asGif() 作用：强制加载 动态图片
    public GifTypeRequest<ModelType> asGif() {
        return optionsApplier.apply(new GifTypeRequest<ModelType>(this, streamModelLoader, optionsApplier));
        // 创建GifTypeRequest对象

        // 注：若没指定，则默认使用DrawableTypeRequest
    }

}

<-- 分析4：buildProvider(）-->
private static <A, Z, R> FixedLoadProvider<A, ImageVideoWrapper, Z, R> buildProvider(Glide glide,
            ModelLoader<A, InputStream> streamModelLoader,
            ModelLoader<A, ParcelFileDescriptor> fileDescriptorModelLoader, Class<Z> resourceClass,
            Class<R> transcodedClass,
            ResourceTranscoder<Z, R> transcoder) {

        if (transcoder == null) {
            transcoder = glide.buildTranscoder(resourceClass, transcodedClass);
            // 创建GifBitmapWrapperDrawableTranscoder对象（实现了ResourceTranscoder接口）
            // 作用：对图片进行转码
        }

        DataLoadProvider<ImageVideoWrapper, Z> dataLoadProvider = glide.buildDataProvider(ImageVideoWrapper.class,
                resourceClass);
        // 创建ImageVideoGifDrawableLoadProvider对象（实现了DataLoadProvider接口）
        // 作用：对图片进行编解码

        ImageVideoModelLoader<A> modelLoader = new ImageVideoModelLoader<A>(streamModelLoader,
                fileDescriptorModelLoader);
        // 创建ImageVideoModelLoader
        // 并把上面创建的两个ModelLoader：streamModelLoader和fileDescriptorModelLoader封装到了ImageVideoModelLoader中

        return new FixedLoadProvider<A, ImageVideoWrapper, Z, R>(modelLoader, transcoder, dataLoadProvider);
       // 创建FixedLoadProvider对象
       // 把上面创建的GifBitmapWrapperDrawableTranscoder、ImageVideoModelLoader、ImageVideoGifDrawableLoadProvider都封装进去
      // 注：FixedLoadProvider对象就是第3步into（）中onSizeReady()的loadProvider对象
    }
      // 回到分析3的关注点2
```



```kotlin
class RequestBuilder {
    @Nullable private Object model;
  
  public RequestBuilder<TranscodeType> load(@Nullable String string) {
    return loadGeneric(string);
  }
  
  private RequestBuilder<TranscodeType> loadGeneric(@Nullable Object model) {
    if (isAutoCloneEnabled()) {
      return clone().loadGeneric(model);
    }
    this.model = model;
    isModelSet = true;
    return selfOrThrowIfLocked();
  }
  
  protected final T selfOrThrowIfLocked() {
    if (isLocked) {
      throw new IllegalStateException("You cannot modify locked T, consider clone()");
    }
    return self();
  }
  
    private T self() {
    return (T) this;
  }
}  
```

### 小结

通过 load 将要加载的资源地址保存到 RequestBuilder.model 字段中。

- 在`RequestManager`的`load()`中，通过`fromString()`最终返回一个`DrawableTypeRequest`对象，并调用该对象的`load()` 传入图片的URL地址

> 请回看分析1上面的代码

- 但从上面的分析3可看出，`DrawableTypeRequest`类中并没有`load()`和第3步需要分析的`into（）`，所以`load()` 和 `into（）` 是在`DrawableTypeRequest`类的父类中：`DrawableRequestBuilder`类中。继承关系如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-f5780a3e42011902.png?imageMogr2/auto-orient/strip|imageView2/2/w/180/format/webp)

```java
public class DrawableRequestBuilder<ModelType>
        extends GenericRequestBuilder<ModelType, ImageVideoWrapper, GifBitmapWrapper, GlideDrawable>
        implements BitmapOptions, DrawableOptions {

        ... 

// 最终load()方法返回的其实就是一个DrawableTypeRequest对象
@Override
    public Target<GlideDrawable> into(ImageView view) {
        return super.into(view);
    }

// 特别注意：DrawableRequestBuilder类中有很多使用Glide的API方法，此处不做过多描述

}
```

至此，第2步的 `load（）`分析完成

### 总结

`load（）`中预先创建好对图片进行一系列操作（加载、编解码、转码）的对象，并全部封装到 `DrawableTypeRequest`对象中。

![img](https:////upload-images.jianshu.io/upload_images/944365-b3b2dae583d9a690.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## into()

- 作用：构建网络请求对象 并 执行 该网络请求

> 即 获取图片资源 & 加载图片并显示

- 总体逻辑如下:

  ![img](https:////upload-images.jianshu.io/upload_images/944365-f27576b3fa7d63f6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  示意图

- 详细过程：

在第2步的`RequestManager`的`load()`中，最终返回一个`DrawableTypeRequest`对象

> 封装好了对图片进行一系列操作（加载、编解码、转码）的对象

- 但 `DrawableTypeRequest`类中并没有`load()`和第3步需要分析的`into（）`，所以`load()` 和 `into（）` 是在`DrawableTypeRequest`类的父类中：`DrawableRequestBuilder`类

> 继承关系如下

![img](https:////upload-images.jianshu.io/upload_images/944365-e53445a13dd5144b.png?imageMogr2/auto-orient/strip|imageView2/2/w/180/format/webp)

示意图

所以，第三步是调用`DrawableRequestBuilder`类的 `into（）`完成图片的最终加载。



```kotlin
  @NonNull
  public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
    Util.assertMainThread();
    Preconditions.checkNotNull(view);

    // 分析1：配置 BaseRequestOptions
    BaseRequestOptions<?> requestOptions = this;
    if (!requestOptions.isTransformationSet()
        && requestOptions.isTransformationAllowed()
        && view.getScaleType() != null) {
      // Clone in this method so that if we use this RequestBuilder to load into a View and then
      // into a different target, we don't retain the transformation applied based on the previous
      // View's scale type.
      switch (view.getScaleType()) {
        case CENTER_CROP:
          requestOptions = requestOptions.clone().optionalCenterCrop();
          break;
        case CENTER_INSIDE:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case FIT_CENTER:
        case FIT_START:
        case FIT_END:
          requestOptions = requestOptions.clone().optionalFitCenter();
          break;
        case FIT_XY:
          requestOptions = requestOptions.clone().optionalCenterInside();
          break;
        case CENTER:
        case MATRIX:
        default:
          // Do nothing.
      }
    }

    return into(
      	// 分析2：构建 Target，buildImageViewTarget
     		// target 就是最终图片的使用者
        glideContext.buildImageViewTarget(view, transcodeClass),
        /* targetListener= */ null,
        requestOptions,
        Executors.mainThreadExecutor());
  }

  private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> options,
      Executor callbackExecutor) {
    Preconditions.checkNotNull(target);
    if (!isModelSet) {
      throw new IllegalArgumentException("You must call #load() before calling #into()");
    }

    // 分析3：buildRequest
    Request request = buildRequest(target, targetListener, options, callbackExecutor);

    Request previous = target.getRequest();
    if (request.isEquivalentTo(previous)
        && !isSkipMemoryCacheWithCompletePreviousRequest(options, previous)) {
      if (!Preconditions.checkNotNull(previous).isRunning()) {
        previous.begin();
      }
      return target;
    }

    requestManager.clear(target);
    target.setRequest(request);
    
    // 分析4：将请求交给 requestManager
    requestManager.track(target, request);

    return target;
  }

```

### 分析1：配置 BaseRequestOptions

### 分析2：构建 Target，buildImageViewTarget

```kotlin
  public <X> ViewTarget<ImageView, X> buildImageViewTarget(
      @NonNull ImageView imageView, @NonNull Class<X> transcodeClass) {
		// 分析2.1 transcodeClass 是传给 Target 的资源类型，默认是Drawable.class
    return imageViewTargetFactory.buildTarget(imageView, transcodeClass);
  }

public class ImageViewTargetFactory {
  @NonNull
  @SuppressWarnings("unchecked")
  public <Z> ViewTarget<ImageView, Z> buildTarget(
      @NonNull ImageView view, @NonNull Class<Z> clazz) {
    if (Bitmap.class.equals(clazz)) {
      return (ViewTarget<ImageView, Z>) new BitmapImageViewTarget(view);
    } else if (Drawable.class.isAssignableFrom(clazz)) {
      return (ViewTarget<ImageView, Z>) new DrawableImageViewTarget(view);
    } else {
      throw new IllegalArgumentException(
          "Unhandled class: " + clazz + ", try .as*(Class).transcode(ResourceTranscoder)");
    }
  }
}

public class DrawableImageViewTarget extends ImageViewTarget<Drawable> {
  public DrawableImageViewTarget(ImageView view) {
    super(view);
  }

  public DrawableImageViewTarget(ImageView view, boolean waitForLayout) {
    super(view, waitForLayout);
  }

  @Override
  protected void setResource(@Nullable Drawable resource) {
    // 分析2.2
    view.setImageDrawable(resource);
  }
}
```

分析2.2：所以等图片资源准备完成后，把图片通过 setResource 交给 Target，就能完成图片的设置。

### 分析3：buildRequest

```kotlin
  private Request buildThumbnailRequestRecursive(
      Object requestLock,
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {
    if (thumbnailBuilder != null) {
...
    } else if (thumbSizeMultiplier != null) {
...
    } else {
      // Base case: no thumbnail.
      return obtainRequest(
          requestLock,
          target,
          targetListener,
          requestOptions,
          parentCoordinator,
          transitionOptions,
          priority,
          overrideWidth,
          overrideHeight,
          callbackExecutor);
    }
  }

  private Request obtainRequest(
      Object requestLock,
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> requestOptions,
      RequestCoordinator requestCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      Executor callbackExecutor) {
    return SingleRequest.obtain(
        context,
        glideContext,
        requestLock,
      // load()设置的资源地址
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        glideContext.getEngine(),
        transitionOptions.getTransitionFactory(),
        callbackExecutor);
  }
```

在没有设置缩略图时，最终只会构建一个 SingleRequest。

### 分析4：将请求交给 requestManager

RequestManager#requestTracker

```kotlin
synchronized void track(@NonNull Target<?> target, @NonNull Request request) {
    targetTracker.track(target);
    requestTracker.runRequest(request);
  }
```

RequestTracker#runRequest：执行请求获取图片资源

```kotlin
  public void runRequest(@NonNull Request request) {
    // 将每个提交的请求加入到一个set中：管理请求
    requests.add(request);
    if (!isPaused) {
      request.begin();
    } else {
      request.clear();
      // 若处于暂停，则先将Request添加到待执行队列里面，等暂停状态解除后再执行
      pendingRequests.add(request);
    }
  }
```

SingleRequest#begin

```kotlin
  @Override
  public void begin() {
    synchronized (requestLock) {
      assertNotCallingCallbacks();
      stateVerifier.throwIfRecycled();
      startTime = LogTime.getLogTime();
...
      // 分析4.1
      if (status == Status.COMPLETE) {
        onResourceReady(
            resource, DataSource.MEMORY_CACHE, /* isLoadedFromAlternateCacheKey= */ false);
        return;
      }

      status = Status.WAITING_FOR_SIZE;
  
		// 图片加载情况分两种：
		// 1. 开发者使用了override() API为图片指定了一个固定的宽高
		// 2. 无使用
      
		// 情况1：使用了override() API为图片指定了一个固定的宽高
      if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        // 调用onSizeReady()加载
        onSizeReady(overrideWidth, overrideHeight);
      } else {
        // 4.2 添加 getSize() 回调
        // target.getSize()的内部会根据ImageView的layout_width和layout_height值做一系列的计算，来算出图片显示的宽高
        // 计算后，也会调用onSizeReady()方法进行加载
        target.getSize(this);
      }

      if ((status == Status.RUNNING || status == Status.WAITING_FOR_SIZE)
          && canNotifyStatusChanged()) {
        // 在图片请求开始时，会先显示Loading占位图
        target.onLoadStarted(getPlaceholderDrawable());
      }
    }
  }

Target#getSize
    void getSize(@NonNull SizeReadyCallback cb) {
      int currentWidth = getTargetWidth();
      int currentHeight = getTargetHeight();
      if (isViewStateAndSizeValid(currentWidth, currentHeight)) {
        cb.onSizeReady(currentWidth, currentHeight);
        return;
      }

      if (!cbs.contains(cb)) {
        cbs.add(cb);
      }
      if (layoutListener == null) {
        ViewTreeObserver observer = view.getViewTreeObserver();
        layoutListener = new SizeDeterminerLayoutListener(this);
        observer.addOnPreDrawListener(layoutListener);
      }
    }
```

分析4.1：SingleRequest 会记录自身的请求状态

分析4.2：可以看到在 begin 后请求并没有直接发出，而是先去获取 view 的宽高，只有获取到宽高后，请求才会真正发出

getSize()传入的是一个 callback，当获取到 view 的宽高后，会回调方法 SingleRequest#onSizeReady 去加载资源

```kotlin
 public void onSizeReady(int width, int height) {
    stateVerifier.throwIfRecycled();
    synchronized (requestLock) {
      if (status != Status.WAITING_FOR_SIZE) {
        return;
      }
      status = Status.RUNNING;

      float sizeMultiplier = requestOptions.getSizeMultiplier();
      this.width = maybeApplySizeMultiplier(width, sizeMultiplier);
      this.height = maybeApplySizeMultiplier(height, sizeMultiplier);

      loadStatus =
      // 分析5
          engine.load(
              glideContext,
              model,
              requestOptions.getSignature(),
              this.width,
              this.height,
              requestOptions.getResourceClass(),
              transcodeClass,
              priority,
              requestOptions.getDiskCacheStrategy(),
              requestOptions.getTransformations(),
              requestOptions.isTransformationRequired(),
              requestOptions.isScaleOnlyOrNoTransform(),
              requestOptions.getOptions(),
              requestOptions.isMemoryCacheable(),
              requestOptions.getUseUnlimitedSourceGeneratorsPool(),
              requestOptions.getUseAnimationPool(),
              requestOptions.getOnlyRetrieveFromCache(),
              this,
              callbackExecutor);

      if (status != Status.RUNNING) {
        loadStatus = null;
      }
    }
  }
```

### 分析5：engine.load

```kotlin
   public <R> LoadStatus load(
      GlideContext glideContext,
      Object model,
      Key signature,
      int width,
      int height,
      Class<?> resourceClass,
      Class<R> transcodeClass,
      Priority priority,
      DiskCacheStrategy diskCacheStrategy,
      Map<Class<?>, Transformation<?>> transformations,
      boolean isTransformationRequired,
      boolean isScaleOnlyOrNoTransform,
      Options options,
      boolean isMemoryCacheable,
      boolean useUnlimitedSourceExecutorPool,
      boolean useAnimationPool,
      boolean onlyRetrieveFromCache,
      ResourceCallback cb,
      Executor callbackExecutor) {
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;

     // 资源Key，用来缓存资源
    EngineKey key =
        keyFactory.buildKey(
            model,
            signature,
            width,
            height,
            transformations,
            resourceClass,
            transcodeClass,
            options);

    EngineResource<?> memoryResource;
    synchronized (this) {
      // 分析6 从内存缓存中获取资源
      memoryResource = loadFromMemory(key, isMemoryCacheable, startTime);

      if (memoryResource == null) {
        return waitForExistingOrStartNewJob(
            glideContext,
            model,
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
            options,
            isMemoryCacheable,
            useUnlimitedSourceExecutorPool,
            useAnimationPool,
            onlyRetrieveFromCache,
            cb,
            callbackExecutor,
            key,
            startTime);
      }
    }

    // Avoid calling back while holding the engine lock, doing so makes it easier for callers to
    // deadlock.
    cb.onResourceReady(
        memoryResource, DataSource.MEMORY_CACHE, /* isLoadedFromAlternateCacheKey= */ false);
    return null;
  }

  private <R> LoadStatus waitForExistingOrStartNewJob(
      GlideContext glideContext,
      Object model,
      Key signature,
      int width,
      int height,
      Class<?> resourceClass,
      Class<R> transcodeClass,
      Priority priority,
      DiskCacheStrategy diskCacheStrategy,
      Map<Class<?>, Transformation<?>> transformations,
      boolean isTransformationRequired,
      boolean isScaleOnlyOrNoTransform,
      Options options,
      boolean isMemoryCacheable,
      boolean useUnlimitedSourceExecutorPool,
      boolean useAnimationPool,
      boolean onlyRetrieveFromCache,
      ResourceCallback cb,
      Executor callbackExecutor,
      EngineKey key,
      long startTime) {

    EngineJob<?> current = jobs.get(key, onlyRetrieveFromCache);
    if (current != null) {
      current.addCallback(cb, callbackExecutor);
      return new LoadStatus(cb, current);
    }
    
    // 创建EngineJob对象
    // 作用：开启线程（作异步加载图片）
    EngineJob<R> engineJob =
        engineJobFactory.build(
            key,
            isMemoryCacheable,
            useUnlimitedSourceExecutorPool,
            useAnimationPool,
            onlyRetrieveFromCache);

    // 创建DecodeJob对象
    // 作用：对图片解码（较复杂，下面会详细说明）
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

    engineJob.addCallback(cb, callbackExecutor);
    
    // 执行EngineRunnable对象
    // 即在子线程中执行EngineRunnable的run()方法
    // 分析7：
    engineJob.start(decodeJob);
    return new LoadStatus(cb, engineJob);
  }
```

### 分析6：从内存缓存中获取资源

### 分析7：engineJob.start从磁盘缓存或网络获取资源

engineJob.start

```kotlin
  public synchronized void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    GlideExecutor executor =
        decodeJob.willDecodeFromCache() ? diskCacheExecutor : getActiveSourceExecutor();
    // 分析8：decodeJob
    executor.execute(decodeJob);
  }
```

### 分析8：decodeJob

```kotlin
  @Override 
public void run() {
    DataFetcher<?> localFetcher = currentFetcher;
      runWrapped();
  }

  private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
      // 分析8.1：计算当前状态
        stage = getNextStage(Stage.INITIALIZE);
      // 分析8.2 根据当前状态创建对应的资源获取器
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }

  private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE
            : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE
            : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
  }

  private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
  }

  private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    while (!isCancelled
        && currentGenerator != null
           // 分析9：currentGenerator.startNext() 去获取资源
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      if (stage == Stage.SOURCE) {
        reschedule(RunReason.SWITCH_TO_SOURCE_SERVICE);
        return;
      }
    }
  }
```

以首次网络请求为例，则会获取到 SourceGenerator

### 分析9：SourceGenerator获取资源

```kotlin
  @Override
  public boolean startNext() {
    if (dataToCache != null) {
...首次无缓存
    }
    
    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      loadData = helper.getLoadData().get(loadDataListIndex++);
      if (loadData != null
          && (helper.getDiskCacheStrategy().isDataCacheable(loadData.fetcher.getDataSource())
              || helper.hasLoadPath(loadData.fetcher.getDataClass()))) {
        started = true;
        // 加载资源，loadData 中记录了 load() 传入的资源地址
        startNextLoad(loadData);
      }
    }
    return started;
  }

  private void startNextLoad(final LoadData<?> toStart) {
    loadData.fetcher.loadData(
        helper.getPriority(),
        new DataCallback<Object>() {
          @Override
          public void onDataReady(@Nullable Object data) {
            if (isCurrentRequest(toStart)) {
              // 加载网络资源
              onDataReadyInternal(toStart, data);
            }
          }

          @Override
          public void onLoadFailed(@NonNull Exception e) {
            if (isCurrentRequest(toStart)) {
              onLoadFailedInternal(toStart, e);
            }
          }
        });
  }

  void onDataReadyInternal(LoadData<?> loadData, Object data) {
    DiskCacheStrategy diskCacheStrategy = helper.getDiskCacheStrategy();
    // 
    if (data != null && diskCacheStrategy.isDataCacheable(loadData.fetcher.getDataSource())) {
      dataToCache = data;
      // We might be being called back on someone else's thread. Before doing anything, we should
      // reschedule to get back onto Glide's thread. Then once we're back on Glide's thread, we'll
      // get called again and we can write the retrieved data to cache.
      cb.reschedule();
    } else {
      cb.onDataFetcherReady(
          loadData.sourceKey,
          data,
          loadData.fetcher,
          loadData.fetcher.getDataSource(),
          originalKey);
    }
  }
```









```java

<--分析18：HttpUrlFetcher的loadData（）  -->
// 此处是网络请求的代码
public class HttpUrlFetcher implements DataFetcher<InputStream> {

    @Override
    public InputStream loadData(Priority priority) throws Exception {
        return loadDataWithRedirects(glideUrl.toURL(), 0 /*redirects*/, null /*lastUrl*/, glideUrl.getHeaders());
        // 继续往下看
    }

    private InputStream loadDataWithRedirects(URL url, int redirects, URL lastUrl, Map<String, String> headers)
           ...

        // 静态工厂模式创建HttpURLConnection对象
        urlConnection = connectionFactory.build(url);
        for (Map.Entry<String, String> headerEntry : headers.entrySet()) {
          urlConnection.addRequestProperty(headerEntry.getKey(), headerEntry.getValue());
        }
        //设置请求参数
        //设置连接超时时间2500ms
        urlConnection.setConnectTimeout(2500);
        //设置读取超时时间2500ms
        urlConnection.setReadTimeout(2500);
        //不使用http缓存
        urlConnection.setUseCaches(false);
        urlConnection.setDoInput(true);

        // Connect explicitly to avoid errors in decoders if connection fails.
        urlConnection.connect();
        if (isCancelled) {
            return null;
        }
        final int statusCode = urlConnection.getResponseCode();
        if (statusCode / 100 == 2) {
              //请求成功
            return getStreamForSuccessfulRequest(urlConnection);
            // 继续往下看
        } 
    }

      private InputStream getStreamForSuccessfulRequest(HttpURLConnection urlConnection)
            throws IOException {
        if (TextUtils.isEmpty(urlConnection.getContentEncoding())) {
            int contentLength = urlConnection.getContentLength();
            stream = ContentLengthInputStream.obtain(urlConnection.getInputStream(), contentLength);
        } else {
            if (Log.isLoggable(TAG, Log.DEBUG)) {
                Log.d(TAG, "Got non empty content encoding: " + urlConnection.getContentEncoding());
            }
            stream = urlConnection.getInputStream();
        }
        return stream;
        // 最终返回InputStream对象（但还没开始读取数据）
        // 回到分析17中的最后一行
    }
        
    }
}
```

### 分析19：图片的解码

```java
<--分析19：decodeFromSourceData()（）  -->
private Resource<T> decodeFromSourceData(A data) throws IOException {

        decoded = loadProvider.getSourceDecoder().decode(data, width, height);
        // 调用loadProvider.getSourceDecoder()得到的是GifBitmapWrapperResourceDecoder对象
        // 即调用GifBitmapWrapperResourceDecoder对象的decode()来对图片进行解码 ->>分析20
    
    return decoded;
}

<--分析20：GifBitmapWrapperResourceDecoder对象的decode()  -->
    public class GifBitmapWrapperResourceDecoder implements ResourceDecoder<ImageVideoWrapper, GifBitmapWrapper> {

    ...

    @Override
    public Resource<GifBitmapWrapper> decode(ImageVideoWrapper source, int width, int height) throws IOException {

            wrapper = decode(source, width, height, tempBytes);
            // 传入参数，并调用了另外一个decode（）进行重载 ->>分析21

    }

<--分析21：重载的decode（） -->
    private GifBitmapWrapper decode(ImageVideoWrapper source, int width, int height, byte[] bytes) throws IOException {
        final GifBitmapWrapper result;
        if (source.getStream() != null) {
            result = decodeStream(source, width, height, bytes);
            // 作用：从服务器返回的流当中读取数据- >>分析22
            
        } else {
            result = decodeBitmapWrapper(source, width, height);
        }
        return result;
    }

<--分析22：decodeStream（） -->
// 作用：从服务器返回的流当中读取数据
// 读取方式：
        // 1. 从流中读取2个字节的数据：判断该图是GIF图还是普通的静图
        // 2. 若是GIF图，就调用decodeGifWrapper() 解码
        // 3. 若普通静图，就调用decodeBitmapWrapper() 解码
        // 此处仅分析 对于静图解码
    private GifBitmapWrapper decodeStream(ImageVideoWrapper source, int width, int height, byte[] bytes)
            throws IOException {
        
        // 步骤1：从流中读取两个2字节数据进行图片类型的判断
        InputStream bis = streamFactory.build(source.getStream(), bytes);
        bis.mark(MARK_LIMIT_BYTES);
        ImageHeaderParser.ImageType type = parser.parse(bis);
        bis.reset();
        GifBitmapWrapper result = null;

        // 步骤2：若是GIF图，就调用decodeGifWrapper() 解码
        if (type == ImageHeaderParser.ImageType.GIF) {
            result = decodeGifWrapper(bis, width, height);
        }

        // 步骤3：若是普通静图，就调用decodeBitmapWrapper()解码
        if (result == null) {
            ImageVideoWrapper forBitmapDecoder = new ImageVideoWrapper(bis, source.getFileDescriptor());
            result = decodeBitmapWrapper(forBitmapDecoder, width, height);
            // ->>分析23
        }
        return result;
    }

<-- 分析23：decodeBitmapWrapper() -->
    private GifBitmapWrapper decodeBitmapWrapper(ImageVideoWrapper toDecode, int width, int height) throws IOException {
        GifBitmapWrapper result = null;
        Resource<Bitmap> bitmapResource = bitmapDecoder.decode(toDecode, width, height);
       //  bitmapDecoder是一个ImageVideoBitmapDecoder对象
       // 即调用ImageVideoBitmapDecoder对象的decode（）->>分析24
        if (bitmapResource != null) {
            result = new GifBitmapWrapper(bitmapResource, null);
        }
        return result;
    }

    ...
}

<-- 分析24：ImageVideoBitmapDecoder.decode() -->
public class ImageVideoBitmapDecoder implements ResourceDecoder<ImageVideoWrapper, Bitmap> {

    ...

    @Override
    public Resource<Bitmap> decode(ImageVideoWrapper source, int width, int height) throws IOException {
        Resource<Bitmap> result = null;
        InputStream is = source.getStream();
        // 步骤1：获取到服务器返回的InputStream
        if (is != null) {
            try {
                result = streamDecoder.decode(is, width, height);
                // 步骤2：调用streamDecoder.decode()进行解码
                // streamDecode是一个StreamBitmapDecoder对象 ->>分析25

            } catch (IOException e) {
    ...
}

<-- 分析25：StreamBitmapDecoder.decode() -->
public class StreamBitmapDecoder implements ResourceDecoder<InputStream, Bitmap> {

    ...

    @Override
    public Resource<Bitmap> decode(InputStream source, int width, int height) {
        Bitmap bitmap = downsampler.decode(source, bitmapPool, width, height, decodeFormat);
        // Downsampler的decode() ->>分析26

        // 从分析26回来看这里：
        return BitmapResource.obtain(bitmap, bitmapPool);
        // 作用：将分析26中返回的Bitmap对象包装成Resource<Bitmap>对象
        // 因为decode()返回的是一个Resource<Bitmap>对象；而从Downsampler中得到的是一个Bitmap对象，需要进行类型的转换
        // 经过这样一层包装后，如果还需要获取Bitmap，只需要调用Resource<Bitmap>的get()即可
        // 接下来，我们需要一层层地向上返回（请向下看直到跳出该代码块）
    }

    ...
}

<-- 分析26：downsampler.decode（）  -->
// 主要作用：读取服务器返回的InputStream & 加载图片
// 其他作用：对图片的压缩、旋转、圆角等逻辑处理
public abstract class Downsampler implements BitmapDecoder<InputStream> {

    ...

    @Override
    public Bitmap decode(InputStream is, BitmapPool pool, int outWidth, int outHeight, DecodeFormat decodeFormat) {
        final ByteArrayPool byteArrayPool = ByteArrayPool.get();
        final byte[] bytesForOptions = byteArrayPool.getBytes();
        final byte[] bytesForStream = byteArrayPool.getBytes();
        final BitmapFactory.Options options = getDefaultOptions();
        // Use to fix the mark limit to avoid allocating buffers that fit entire images.
        RecyclableBufferedInputStream bufferedStream = new RecyclableBufferedInputStream(
                is, bytesForStream);
        // Use to retrieve exceptions thrown while reading.
        // TODO(#126): when the framework no longer returns partially decoded Bitmaps or provides a way to determine
        // if a Bitmap is partially decoded, consider removing.
        ExceptionCatchingInputStream exceptionStream =
                ExceptionCatchingInputStream.obtain(bufferedStream);
        // Use to read data.
        // Ensures that we can always reset after reading an image header so that we can still attempt to decode the
        // full image even when the header decode fails and/or overflows our read buffer. See #283.
        MarkEnforcingInputStream invalidatingStream = new MarkEnforcingInputStream(exceptionStream);
        try {
            exceptionStream.mark(MARK_POSITION);
            int orientation = 0;
            try {
                orientation = new ImageHeaderParser(exceptionStream).getOrientation();
            } catch (IOException e) {
                if (Log.isLoggable(TAG, Log.WARN)) {
                    Log.w(TAG, "Cannot determine the image orientation from header", e);
                }
            } finally {
                try {
                    exceptionStream.reset();
                } catch (IOException e) {
                    if (Log.isLoggable(TAG, Log.WARN)) {
                        Log.w(TAG, "Cannot reset the input stream", e);
                    }
                }
            }
            options.inTempStorage = bytesForOptions;
            final int[] inDimens = getDimensions(invalidatingStream, bufferedStream, options);
            final int inWidth = inDimens[0];
            final int inHeight = inDimens[1];
            final int degreesToRotate = TransformationUtils.getExifOrientationDegrees(orientation);
            final int sampleSize = getRoundedSampleSize(degreesToRotate, inWidth, inHeight, outWidth, outHeight);
            final Bitmap downsampled =
                    downsampleWithSize(invalidatingStream, bufferedStream, options, pool, inWidth, inHeight, sampleSize,
                            decodeFormat);
            // BitmapFactory swallows exceptions during decodes and in some cases when inBitmap is non null, may catch
            // and log a stack trace but still return a non null bitmap. To avoid displaying partially decoded bitmaps,
            // we catch exceptions reading from the stream in our ExceptionCatchingInputStream and throw them here.
            final Exception streamException = exceptionStream.getException();
            if (streamException != null) {
                throw new RuntimeException(streamException);
            }
            Bitmap rotated = null;
            if (downsampled != null) {
                rotated = TransformationUtils.rotateImageExif(downsampled, pool, orientation);
                if (!downsampled.equals(rotated) && !pool.put(downsampled)) {
                    downsampled.recycle();
                }
            }
            return rotated;
        } finally {
            byteArrayPool.releaseBytes(bytesForOptions);
            byteArrayPool.releaseBytes(bytesForStream);
            exceptionStream.release();
            releaseOptions(options);
        }
    }

    private Bitmap downsampleWithSize(MarkEnforcingInputStream is, RecyclableBufferedInputStream  bufferedStream,
            BitmapFactory.Options options, BitmapPool pool, int inWidth, int inHeight, int sampleSize,
            DecodeFormat decodeFormat) {
        // Prior to KitKat, the inBitmap size must exactly match the size of the bitmap we're decoding.
        Bitmap.Config config = getConfig(is, decodeFormat);
        options.inSampleSize = sampleSize;
        options.inPreferredConfig = config;
        if ((options.inSampleSize == 1 || Build.VERSION_CODES.KITKAT <= Build.VERSION.SDK_INT) && shouldUsePool(is)) {
            int targetWidth = (int) Math.ceil(inWidth / (double) sampleSize);
            int targetHeight = (int) Math.ceil(inHeight / (double) sampleSize);
            // BitmapFactory will clear out the Bitmap before writing to it, so getDirty is safe.
            setInBitmap(options, pool.getDirty(targetWidth, targetHeight, config));
        }
        return decodeStream(is, bufferedStream, options);
    }

    /**
     * A method for getting the dimensions of an image from the given InputStream.
     *
     * @param is The InputStream representing the image.
     * @param options The options to pass to
     *          {@link BitmapFactory#decodeStream(InputStream, android.graphics.Rect,
     *              BitmapFactory.Options)}.
     * @return an array containing the dimensions of the image in the form {width, height}.
     */
    public int[] getDimensions(MarkEnforcingInputStream is, RecyclableBufferedInputStream bufferedStream,
            BitmapFactory.Options options) {
        options.inJustDecodeBounds = true;
        decodeStream(is, bufferedStream, options);
        options.inJustDecodeBounds = false;
        return new int[] { options.outWidth, options.outHeight };
    }

    private static Bitmap decodeStream(MarkEnforcingInputStream is, RecyclableBufferedInputStream bufferedStream,
            BitmapFactory.Options options) {
         if (options.inJustDecodeBounds) {
             // This is large, but jpeg headers are not size bounded so we need something large enough to minimize
             // the possibility of not being able to fit enough of the header in the buffer to get the image size so
             // that we don't fail to load images. The BufferedInputStream will create a new buffer of 2x the
             // original size each time we use up the buffer space without passing the mark so this is a maximum
             // bound on the buffer size, not a default. Most of the time we won't go past our pre-allocated 16kb.
             is.mark(MARK_POSITION);
         } else {
             // Once we've read the image header, we no longer need to allow the buffer to expand in size. To avoid
             // unnecessary allocations reading image data, we fix the mark limit so that it is no larger than our
             // current buffer size here. See issue #225.
             bufferedStream.fixMarkLimit();
         }

        final Bitmap result = BitmapFactory.decodeStream(is, null, options);
        return result;
        // decode()方法执行后会返回一个Bitmap对象
        // 此时图片已经被加载出来
        // 接下来的工作是让加载了的Bitmap显示到界面上
        // 请回到分析25
    }

    ...
}
```

------

#### 步骤3：返回图片资源

加载完图片后，需要一层层向上返回

- 返回路径
   `StreamBitmapDecoder`（分析25）-> `ImageVideoBitmapDecoder`（分析24）-> `GifBitmapWrapperResourceDecoder``decodeBitmapWrapper()`（分析23）
- 由于隔得太远，我重新把（分析23）`decodeBitmapWrapper()`贴出

```java
<-- 分析23：decodeBitmapWrapper -->
private GifBitmapWrapper decodeBitmapWrapper(ImageVideoWrapper toDecode, int width, int height) throws IOException {
    GifBitmapWrapper result = null;
    Resource<Bitmap> bitmapResource = bitmapDecoder.decode(toDecode, width, height);
    if (bitmapResource != null) {
        result = new GifBitmapWrapper(bitmapResource, null);
        // 将Resource<Bitmap>封装到了一个GifBitmapWrapper对象
    }
    return result;
   // 最终返回的是一个GifBitmapWrapper对象：既能封装GIF，又能封装Bitmap，从而保证了不管是什么类型的图片，Glide都能加载
   // 接下来我们分析下GifBitmapWrapper（） ->>分析27
}

<-- 分析27：GifBitmapWrapper（） -->
// 作用：分别对gifResource和bitmapResource做了一层封装
public class GifBitmapWrapper {
    private final Resource<GifDrawable> gifResource;
    private final Resource<Bitmap> bitmapResource;

    public GifBitmapWrapper(Resource<Bitmap> bitmapResource, Resource<GifDrawable> gifResource) {
        if (bitmapResource != null && gifResource != null) {
            throw new IllegalArgumentException("Can only contain either a bitmap resource or a gif resource, not both");
        }
        if (bitmapResource == null && gifResource == null) {
            throw new IllegalArgumentException("Must contain either a bitmap resource or a gif resource");
        }
        this.bitmapResource = bitmapResource;
        this.gifResource = gifResource;
    }

    /**
     * Returns the size of the wrapped resource.
     */
    public int getSize() {
        if (bitmapResource != null) {
            return bitmapResource.getSize();
        } else {
            return gifResource.getSize();
        }
    }

    /**
     * Returns the wrapped {@link Bitmap} resource if it exists, or null.
     */
    public Resource<Bitmap> getBitmapResource() {
        return bitmapResource;
    }

    /**
     * Returns the wrapped {@link GifDrawable} resource if it exists, or null.
     */
    public Resource<GifDrawable> getGifResource() {
        return gifResource;
    }
}
```

- 然后该`GifBitmapWrapper`对象会一直向上返回
- 直到返回到`GifBitmapWrapperResourceDecoder的decode()`时（分析20），会对`GifBitmapWrapper`对象再做一次封装，如下所示：

> 此处将上面的分析20再次粘贴过来

```java
<--分析20：GifBitmapWrapperResourceDecoder对象的decode()  -->
public class GifBitmapWrapperResourceDecoder implements ResourceDecoder<ImageVideoWrapper, GifBitmapWrapper> {

    ...
    @Override
    public Resource<GifBitmapWrapper> decode(ImageVideoWrapper source, int width, int height) throws IOException {

        try {
            wrapper = decode(source, width, height, tempBytes);
        } finally {
            pool.releaseBytes(tempBytes);
        }

        // 直接看这里
        return wrapper != null ? new GifBitmapWrapperResource(wrapper) : null;
        // 将GifBitmapWrapper封装到一个GifBitmapWrapperResource对象中（Resource<GifBitmapWrapper>类型） 并返回
        // 该GifBitmapWrapperResource和上述的BitmapResource类似- 实现了Resource接口，可通过get()来获取封装的具体内容
        // GifBitmapWrapperResource（）源码分析 - >>分析28
    }

<-- 分析28： GifBitmapWrapperResource（）-->
// 作用：经过这层封装后，我们从网络上得到的图片就能够以Resource接口的形式返回，并且还能同时处理Bitmap图片和GIF图片这两种情况。
public class GifBitmapWrapperResource implements Resource<GifBitmapWrapper> {
    private final GifBitmapWrapper data;

    public GifBitmapWrapperResource(GifBitmapWrapper data) {
        if (data == null) {
            throw new NullPointerException("Data must not be null");
        }
        this.data = data;
    }

    @Override
    public GifBitmapWrapper get() {
        return data;
    }

    @Override
    public int getSize() {
        return data.getSize();
    }

    @Override
    public void recycle() {
        Resource<Bitmap> bitmapResource = data.getBitmapResource();
        if (bitmapResource != null) {
            bitmapResource.recycle();
        }
        Resource<GifDrawable> gifDataResource = data.getGifResource();
        if (gifDataResource != null) {
            gifDataResource.recycle();
        }
    }
}
```

继续返回到`DecodeJob`的`decodeFromSourceData()`（分析19）中：



```kotlin
<-- 分析19：decodeFromSourceData()（）  -->
private Resource<T> decodeFromSourceData(A data) throws IOException {

        decoded = loadProvider.getSourceDecoder().decode(data, width, height);

    return decoded;
    // 该方法返回的是一个`Resource<T>`对象，其实就是Resource<GifBitmapWrapper>对象
}
```

- 继续向上返回，最终返回到`DecodeJob`的`decodeFromSource()`中（分析15）
- 如下所示：



```java
<-- 分析15：DecodeJob的decodeFromSource()  -->
class DecodeJob<A, T, Z> {
    ...

 public Resource<Z> decodeFromSource() throws Exception {
        Resource<T> decoded = decodeSource();
        // 返回到这里，最终得到了这个Resource<T>对象，即Resource<GifBitmapWrapper>对象
        return transformEncodeAndTranscode(decoded);
        // 作用：将该Resource<T>对象 转换成 Resource<Z>对象 -->分析29
    }


<--分析29：transformEncodeAndTranscode（） -->
private Resource<Z> transformEncodeAndTranscode(Resource<T> decoded) {

    Resource<Z> result = transcode(transformed);
     // 把Resource<T>对象转换成Resource<Z>对象 ->>分析30
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Transcoded transformed from source", startTime);
    }
    return result;
}

<-- 分析30：transcode(transformed)  -->

private Resource<Z> transcode(Resource<T> transformed) {
    if (transformed == null) {
        return null;
    }
    return transcoder.transcode(transformed);
    // 调用了transcoder的transcode()
    // 这里的transcoder就是第二步load（）中的GifBitmapWrapperDrawableTranscoder对象（回看下第2步生成对象的表） 
    // 接下来请看 ->>分析31
}

<-- 分析31：GifBitmapWrapperDrawableTranscoder.transcode(transformed) -->
// 作用：转码，即从Resource<GifBitmapWrapper>中取出GifBitmapWrapper对象，然后再从GifBitmapWrapper中取出Resource<Bitmap>对象。
// 因为GifBitmapWrapper是无法直接显示到ImageView上的，只有Bitmap或者Drawable才能显示到ImageView上。

public class GifBitmapWrapperDrawableTranscoder implements ResourceTranscoder<GifBitmapWrapper, GlideDrawable> {

...

    @Override
    public Resource<GlideDrawable> transcode(Resource<GifBitmapWrapper> toTranscode) {
        GifBitmapWrapper gifBitmap = toTranscode.get();
        // 步骤1：从Resource<GifBitmapWrapper>中取出GifBitmapWrapper对象（上面提到的调用get（）进行提取）
        Resource<Bitmap> bitmapResource = gifBitmap.getBitmapResource();
        // 步骤2：从GifBitmapWrapper中取出Resource<Bitmap>对象

        final Resource<? extends GlideDrawable> result;


        // 接下来做了一个判断：

        // 1. 若Resource<Bitmap>不为空
        if (bitmapResource != null) {
            result = bitmapDrawableResourceTranscoder.transcode(bitmapResource);
            // 则需要再做一次转码：将Bitmap转换成Drawable对象
            // 因为要保证静图和动图的类型一致性，否则难以处理->>分析32
        } else {

      // 2. 若Resource<Bitmap>为空（说明此时加载的是GIF图）
      // 那么直接调用getGifResource()方法将图片取出
      // 因为Glide用于加载GIF图片是使用的GifDrawable这个类，它本身就是一个Drawable对象
            result = gifBitmap.getGifResource();
        }
        return (Resource<GlideDrawable>) result;
    }

    ...
}

<-- 分析32：bitmapDrawableResourceTranscoder.transcode(bitmapResource)-->
// 作用：再做一次转码：将Bitmap转换成Drawable对象
public class GlideBitmapDrawableTranscoder implements ResourceTranscoder<Bitmap, GlideBitmapDrawable> {

...

    @Override
    public Resource<GlideBitmapDrawable> transcode(Resource<Bitmap> toTranscode) {

        GlideBitmapDrawable drawable = new GlideBitmapDrawable(resources, toTranscode.get());
        // 创建GlideBitmapDrawable对象，并把Bitmap封装到里面

        return new GlideBitmapDrawableResource(drawable, bitmapPool);
        // 对GlideBitmapDrawable再进行一次封装，返回Resource<GlideBitmapDrawable>对象
    }

}
```

- 此时，无论是静图的 `Resource<GlideBitmapDrawable>` 对象，还是动图的`Resource<GifDrawable>` 对象，它们都属于父类`Resource<GlideDrawable>`对象
- 因此`transcode()`返回的是`Resource<GlideDrawable>`对象，即转换过后的`Resource<Z>`

------

所以，分析15`DecodeJob的decodeFromSource()`中，得到的Resource<Z>对象  是  `Resource<GlideDrawable>`对象

------

### 步骤4：在主线程显示图片

继续向上返回，最终返回到 `EngineRunnable` 的 `run()` 中（分析12）

> 重新贴出这部分代码



```java
<--分析12：EngineRunnable的run() -->
@Override
public void run() {

    try {
        resource = decode();
        // 最终得到了Resource<GlideDrawable>对象
        // 接下来的工作：将该图片显示出来

    } catch (Exception e) {
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            Log.v(TAG, "Exception decoding", e);
        }
        exception = e;
    }
    if (isCancelled) {
        if (resource != null) {
            resource.recycle();
        }
        return;
    }
    if (resource == null) {
        onLoadFailed(exception);
    } else {
        onLoadComplete(resource);
        // 表示图片加载已经完成 ->>分析33
    }
}

<-- 分析33：  onLoadComplete(resource) -->
private void onLoadComplete(Resource resource) {
    manager.onResourceReady(resource);
    // 该manager即EngineJob对象
    // 实际上调用的是EngineJob的onResourceReady() - >>分析34
}

<-- 分析34：EngineJob的onResourceReady() ： -->
class EngineJob implements EngineRunnable.EngineRunnableManager {
    ...

    private static final Handler MAIN_THREAD_HANDLER = new Handler(Looper.getMainLooper(), new MainThreadCallback());
    // 创建线程，并绑定主线程的Looper
    private final List<ResourceCallback> cbs = new ArrayList<ResourceCallback>();

    @Override
    public void onResourceReady(final Resource<?> resource) {
        this.resource = resource;
        MAIN_THREAD_HANDLER.obtainMessage(MSG_COMPLETE, this).sendToTarget();
       // 使用Handler发出一条 MSG_COMPLETE 消息
      // 那么在MainThreadCallback的handleMessage()方法中就会收到这条消息 ->>分析35
      // 从此处开始，所有逻辑又回到主线程中进行了，即更新UI

    }

<-- 分析35：MainThreadCallback的handleMessage()-->
    private static class MainThreadCallback implements Handler.Callback {

        @Override
        public boolean handleMessage(Message message) {
            if (MSG_COMPLETE == message.what || MSG_EXCEPTION == message.what) {
                EngineJob job = (EngineJob) message.obj;
                if (MSG_COMPLETE == message.what) {
                    job.handleResultOnMainThread();
                    // 调用 EngineJob的handleResultOnMainThread() ->>分析36
                } else {
                    job.handleExceptionOnMainThread();
                }
                return true;
            }
            return false;
        }
    }

    ...
}

<-- 分析36：handleResultOnMainThread() -->
private void handleResultOnMainThread() {

        // 通过循环，调用了所有ResourceCallback的onResourceReady()
        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                engineResource.acquire();
                cb.onResourceReady(engineResource);
                // ResourceCallback 是在addCallback()方法当中添加的->>分析37
            }
        }
        engineResource.release();
    }

<-- 分析37：addCallback() -->
//
    public void addCallback(ResourceCallback cb) {
        Util.assertMainThread();
        if (hasResource) {
            cb.onResourceReady(engineResource);
            // 会向cbs集合中去添加ResourceCallback
        } else if (hasException) {
            cb.onException(exception);
        } else {
            cbs.add(cb);
        }
    }

// 而addCallback()是在分析11：Engine的load()中调用的：
<-- 上面的分析11：Engine的load() -->
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

      ...

      public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder, Priority priority, 
            boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {

              engineJob.addCallback(cb);
           // 调用addCallback()注册了一个ResourceCallback
           // 上述参数cb是load()传入的的最后一个参数
            // 而load（）是在GenericRequest的onSizeReady()调用的->>回到分析9（下面重新贴多了一次）
        return new LoadStatus(cb, engineJob);
    }

    ...
}

<-- 上面的分析9：onSizeReady() -->
public void onSizeReady(int width, int height) {

... 
    loadStatus = engine.load(signature, width, height, dataFetcher, loadProvider, transformation, transcoder,
            priority, isMemoryCacheable, diskCacheStrategy, this);
            // load（）最后一个参数是this
            // 所以，ResourceCallback类型参数cb是this
            // 而GenericRequest本身实现了ResourceCallback接口
            // 因此，EngineJob的回调 = cb.onResourceReady(engineResource) = 最终回调GenericRequest的onResourceReady() -->>分析6

    }
}

<-- 分析38：GenericRequest的onResourceReady() -->
// onResourceReady()存在两个方法重载

// 重载1
public void onResourceReady(Resource<?> resource) {
    Object received = resource.get();
    // 获取封装的图片对象（GlideBitmapDrawable对象 或 GifDrawable对象

       onResourceReady(resource, (R) received);
      // 然后将该获得的图片对象传入到了onResourceReady()的重载方法中 ->>看重载2
}


// 重载2
private void onResourceReady(Resource<?> resource, R result) {
    
        ...

        target.onResourceReady(result, animation);
        // Target是在第3步into()的最后1行调用glide.buildImageViewTarget()方法来构建出的Target：GlideDrawableImageViewTarget对象
        // ->>分析39

    }

<-- 分析39：GlideDrawableImageViewTarget.onResourceReady  -->
public class GlideDrawableImageViewTarget extends ImageViewTarget<GlideDrawable> {

    @Override
    public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> animation) {
        if (!resource.isAnimated()) {
            float viewRatio = view.getWidth() / (float) view.getHeight();
            float drawableRatio = resource.getIntrinsicWidth() / (float) resource.getIntrinsicHeight();
            if (Math.abs(viewRatio - 1f) <= SQUARE_RATIO_MARGIN
                    && Math.abs(drawableRatio - 1f) <= SQUARE_RATIO_MARGIN) {
                resource = new SquaringDrawable(resource, view.getWidth());
            }
        }
        super.onResourceReady(resource, animation);
        // 若是静态图片，就调用父类的.onResourceReady() 将GlideDrawable显示到ImageView上
        // GlideDrawableImageViewTarget的父类是ImageViewTarget ->>分析40
        this.resource = resource;
        resource.setLoopCount(maxLoopCount);
        resource.start();
        // 如果是GIF图片，就调用resource.start()方法开始播放图片
    }

    @Override
    protected void setResource(GlideDrawable resource) {
        view.setImageDrawable(resource);
    }

 ...
}

<-- 分析40：ImageViewTarget.onResourceReady（） -->
public abstract class ImageViewTarget<Z> extends ViewTarget<ImageView, Z> implements GlideAnimation.ViewAdapter {

    ...

    @Override
    public void onResourceReady(Z resource, GlideAnimation<? super Z> glideAnimation) {
        if (glideAnimation == null || !glideAnimation.animate(resource, this)) {
            setResource(resource);
            // 继续往下看
        }
    }

    protected abstract void setResource(Z resource);
    // setResource()是一个抽象方法
   // 需要在子类具体实现：请回看上面分析39子类GlideDrawableImageViewTarget类重写的setResource()：调用view.setImageDrawable()，而这个view就是ImageView
  // 即setResource()的具体实现是调用ImageView的setImageDrawable() 并 传入图片，于是就实现了图片显示。

}
```

终于，静图 / Gif图 成功显示出来

### 总结

![img](https:////upload-images.jianshu.io/upload_images/944365-7d34aac6838edd6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

至此，`Glide`的基本功能 **图片加载**的全功能 解析完毕。



### 回调和监听

into()方法的源码：

```kotlin
public Target<TranscodeType> into(ImageView view) {
    Util.assertMainThread();
    if (view == null) {
        throw new IllegalArgumentException("You must pass in a non null View");
    }
    if (!isTransformationSet && view.getScaleType() != null) {
        switch (view.getScaleType()) {
            case CENTER_CROP:
                applyCenterCrop();
                break;
            case FIT_CENTER:
            case FIT_START:
            case FIT_END:
                applyFitCenter();
                break;
            default:
                // Do nothing.
        }
    }
    return into(glide.buildImageViewTarget(view, transcodeClass));
}
```

最后会调用 glide.buildImageViewTarget() 构建出一个Target对象，然后再把它传入到另一个接收Target参数的into()方法中。Target对象是用来最终展示图片用的，glide.buildImageViewTarget() 源码：

```kotlin
public class ImageViewTargetFactory {
@SuppressWarnings("unchecked")
public <Z> Target<Z> buildTarget(ImageView view, Class<Z> clazz) {
    if (GlideDrawable.class.isAssignableFrom(clazz)) {
        return (Target<Z>) new GlideDrawableImageViewTarget(view);
    } else if (Bitmap.class.equals(clazz)) {
        return (Target<Z>) new BitmapImageViewTarget(view);
    } else if (Drawable.class.isAssignableFrom(clazz)) {
        return (Target<Z>) new DrawableImageViewTarget(view);
    } else {
        throw new IllegalArgumentException("Unhandled class: " + clazz
                + ", try .as*(Class).transcode(ResourceTranscoder)");
    }
}
}
```


buildTarget()方法会根据传入的class参数来构建不同的Target对象，如果你在使用Glide加载图片的时候调用了asBitmap()方法，那么这里就会构建出BitmapImageViewTarget对象，否则的话构建的都是GlideDrawableImageViewTarget对象。至于上述代码中的DrawableImageViewTarget对象，这个通常都是用不到的，我们可以暂时不用管它。

之后就会把这里构建出来的Target对象传入到GenericRequest当中，而Glide在图片加载完成之后又会回调GenericRequest的onResourceReady()方法，我们来看一下这部分源码：

```kotlin
public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {
private Target<R> target;
...

private void onResourceReady(Resource<?> resource, R result) {
    boolean isFirstResource = isFirstReadyResource();
    status = Status.COMPLETE;
    this.resource = resource;
    if (requestListener == null || !requestListener.onResourceReady(result, model, target,
            loadedFromMemoryCache, isFirstResource)) {
        GlideAnimation<R> animation = animationFactory.build(loadedFromMemoryCache, isFirstResource);
        target.onResourceReady(result, animation);
    }
    notifyLoadSuccess();
}
...
}
```

这里在第14行调用了target.onResourceReady()方法，而刚才我们已经知道，这里的target就是GlideDrawableImageViewTarget对象，那么我们再来看一下它的源码：

```kotlin
public class GlideDrawableImageViewTarget extends ImageViewTarget<GlideDrawable> {
    ...
@Override
public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> animation) {
    if (!resource.isAnimated()) {
        float viewRatio = view.getWidth() / (float) view.getHeight();
        float drawableRatio = resource.getIntrinsicWidth() / (float) resource.getIntrinsicHeight();
        if (Math.abs(viewRatio - 1f) <= SQUARE_RATIO_MARGIN
                && Math.abs(drawableRatio - 1f) <= SQUARE_RATIO_MARGIN) {
            resource = new SquaringDrawable(resource, view.getWidth());
        }
    }
    super.onResourceReady(resource, animation);
    this.resource = resource;
    resource.setLoopCount(maxLoopCount);
    resource.start();
}

@Override
protected void setResource(GlideDrawable resource) {
    view.setImageDrawable(resource);
}

...
}
```


可以看到，这里在onResourceReady()方法中处理了图片展示，还有GIF播放的逻辑，那么一张图片也就显示出来了，这也就是Glide回调的基本实现原理。



# preload()方法

是在GenericRequestBuilder类当中的，代码如下所示：

```kotlin
public class GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> implements Cloneable {
    ...
public Target<TranscodeType> preload(int width, int height) {
    final PreloadTarget<TranscodeType> target = PreloadTarget.obtain(width, height);
    return into(target);
}

public Target<TranscodeType> preload() {
    return preload(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
}

...
}
```


正如刚才所说，preload()方法有两个方法重载，你可以调用带参数的preload()方法来明确指定图片的宽和高，也可以调用不带参数的preload()方法，它会在内部自动将图片的宽和高都指定成Target.SIZE_ORIGINAL，也就是图片的原始尺寸。

然后我们可以看到，这里在第5行调用了PreloadTarget.obtain()方法获取一个PreloadTarget的实例，并把它传入到了into()方法当中。从刚才的继承结构图中可以看出，PreloadTarget是SimpleTarget的子类，因此它是可以直接传入到into()方法中的。

那么现在的问题就是，PreloadTarget具体的实现到底是什么样子的了，我们看一下它的源码，如下所示：

```kotlin
public final class PreloadTarget<Z> extends SimpleTarget<Z> {
public static <Z> PreloadTarget<Z> obtain(int width, int height) {
    return new PreloadTarget<Z>(width, height);
}

private PreloadTarget(int width, int height) {
    super(width, height);
}

@Override
public void onResourceReady(Z resource, GlideAnimation<? super Z> glideAnimation) {
    Glide.clear(this);
}
}
```


PreloadTarget的源码非常简单，obtain()方法中就是new了一个PreloadTarget的实例而已，而onResourceReady()方法中也没做什么事情，只是调用了Glide.clear()方法。

这里的Glide.clear()并不是清空缓存的意思，而是表示加载已完成，释放资源的意思，因此不用在这里产生疑惑。

其实PreloadTarget的思想和我们刚才提到设计思路是一样的，就是什么都不做就可以了。因为图片加载完成之后只将它缓存而不去显示它，那不就相当于预加载了嘛。



# 5. 总结

一图总结`Glide`的基本功能 **图片加载**的全过程

![img](https:////upload-images.jianshu.io/upload_images/944365-3e5246821f3a3eed.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



> 源码分析方法：抽丝剥茧、点到即止。应该认准一个功能点，然后去分析这个功能点是如何实现的。但只要去追寻主体的实现逻辑即可，千万不要试图去搞懂每一行代码都是什么意思，那样很容易会陷入到思维黑洞当中，而且越陷越深。因为这些庞大的系统都不是由一个人写出来的，每一行代码都想搞明白，就会感觉自己是在盲人摸象，永远也研究不透。如果只是去分析主体的实现逻辑，那么就有比较明确的目的性，这样阅读源码会更加轻松，也更加有成效。
>
> ——郭霖





可以看到，在ImageVideoFetcher的loadData()方法的第6行，这里又去调用了streamFetcher.loadData()方法，那么这个streamFetcher是什么呢？自然就是刚才在组装ImageVideoFetcher对象时传进来的HttpUrlFetcher了。因此这里又会去调用HttpUrlFetcher的loadData()方法，那么我们继续跟进去瞧一瞧：

public class HttpUrlFetcher implements DataFetcher<InputStream> {

    ...
    
    @Override
    public InputStream loadData(Priority priority) throws Exception {
        return loadDataWithRedirects(glideUrl.toURL(), 0 /*redirects*/, null /*lastUrl*/, glideUrl.getHeaders());
    }
    
    private InputStream loadDataWithRedirects(URL url, int redirects, URL lastUrl, Map<String, String> headers)
            throws IOException {
        if (redirects >= MAXIMUM_REDIRECTS) {
            throw new IOException("Too many (> " + MAXIMUM_REDIRECTS + ") redirects!");
        } else {
            // Comparing the URLs using .equals performs additional network I/O and is generally broken.
            // See http://michaelscharf.blogspot.com/2006/11/javaneturlequals-and-hashcode-make.html.
            try {
                if (lastUrl != null && url.toURI().equals(lastUrl.toURI())) {
                    throw new IOException("In re-direct loop");
                }
            } catch (URISyntaxException e) {
                // Do nothing, this is best effort.
            }
        }
        urlConnection = connectionFactory.build(url);
        for (Map.Entry<String, String> headerEntry : headers.entrySet()) {
          urlConnection.addRequestProperty(headerEntry.getKey(), headerEntry.getValue());
        }
        urlConnection.setConnectTimeout(2500);
        urlConnection.setReadTimeout(2500);
        urlConnection.setUseCaches(false);
        urlConnection.setDoInput(true);
    
        // Connect explicitly to avoid errors in decoders if connection fails.
        urlConnection.connect();
        if (isCancelled) {
            return null;
        }
        final int statusCode = urlConnection.getResponseCode();
        if (statusCode / 100 == 2) {
            return getStreamForSuccessfulRequest(urlConnection);
        } else if (statusCode / 100 == 3) {
            String redirectUrlString = urlConnection.getHeaderField("Location");
            if (TextUtils.isEmpty(redirectUrlString)) {
                throw new IOException("Received empty or null redirect url");
            }
            URL redirectUrl = new URL(url, redirectUrlString);
            return loadDataWithRedirects(redirectUrl, redirects + 1, url, headers);
        } else {
            if (statusCode == -1) {
                throw new IOException("Unable to retrieve response code from HttpUrlConnection.");
            }
            throw new IOException("Request failed " + statusCode + ": " + urlConnection.getResponseMessage());
        }
    }
    
    private InputStream getStreamForSuccessfulRequest(HttpURLConnection urlConnection)
            throws IOException {
        if (TextUtils.isEmpty(urlConnection.getContentEncoding())) {
            int contentLength = urlConnection.getContentLength();
            stream = ContentLengthInputStream.obtain(urlConnection.getInputStream(), contentLength);
        } else {
            if (Log.isLoggable(TAG, Log.DEBUG)) {
                Log.d(TAG, "Got non empty content encoding: " + urlConnection.getContentEncoding());
            }
            stream = urlConnection.getInputStream();
        }
        return stream;
    }
    
    ...
}
经过一层一层地跋山涉水，我们终于在这里找到网络通讯的代码了！之前有朋友跟我讲过，说Glide的源码实在是太复杂了，甚至连网络请求是在哪里发出去的都找不到。我们也是经过一段一段又一段的代码跟踪，终于把网络请求的代码给找出来了，实在是太不容易了。

不过也别高兴得太早，现在离最终分析完还早着呢。可以看到，loadData()方法只是返回了一个InputStream，服务器返回的数据连读都还没开始读呢。所以我们还是要静下心来继续分析，回到刚才ImageVideoFetcher的loadData()方法中，在这个方法的最后一行，创建了一个ImageVideoWrapper对象，并把刚才得到的InputStream作为参数传了进去。

然后我们回到再上一层，也就是DecodeJob的decodeSource()方法当中，在得到了这个ImageVideoWrapper对象之后，紧接着又将这个对象传入到了decodeFromSourceData()当中，来去解码这个对象。decodeFromSourceData()方法的代码如下所示：

private Resource<T> decodeFromSourceData(A data) throws IOException {
    final Resource<T> decoded;
    if (diskCacheStrategy.cacheSource()) {
        decoded = cacheAndDecodeSourceData(data);
    } else {
        long startTime = LogTime.getLogTime();
        decoded = loadProvider.getSourceDecoder().decode(data, width, height);
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Decoded from source", startTime);
        }
    }
    return decoded;
}
可以看到，这里在第7行调用了loadProvider.getSourceDecoder().decode()方法来进行解码。loadProvider就是刚才在onSizeReady()方法中得到的FixedLoadProvider，而getSourceDecoder()得到的则是一个GifBitmapWrapperResourceDecoder对象，也就是要调用这个对象的decode()方法来对图片进行解码。那么我们来看下GifBitmapWrapperResourceDecoder的代码：

public class GifBitmapWrapperResourceDecoder implements ResourceDecoder<ImageVideoWrapper, GifBitmapWrapper> {

    ...
    
    @SuppressWarnings("resource")
    // @see ResourceDecoder.decode
    @Override
    public Resource<GifBitmapWrapper> decode(ImageVideoWrapper source, int width, int height) throws IOException {
        ByteArrayPool pool = ByteArrayPool.get();
        byte[] tempBytes = pool.getBytes();
        GifBitmapWrapper wrapper = null;
        try {
            wrapper = decode(source, width, height, tempBytes);
        } finally {
            pool.releaseBytes(tempBytes);
        }
        return wrapper != null ? new GifBitmapWrapperResource(wrapper) : null;
    }
    
    private GifBitmapWrapper decode(ImageVideoWrapper source, int width, int height, byte[] bytes) throws IOException {
        final GifBitmapWrapper result;
        if (source.getStream() != null) {
            result = decodeStream(source, width, height, bytes);
        } else {
            result = decodeBitmapWrapper(source, width, height);
        }
        return result;
    }
    
    private GifBitmapWrapper decodeStream(ImageVideoWrapper source, int width, int height, byte[] bytes)
            throws IOException {
        InputStream bis = streamFactory.build(source.getStream(), bytes);
        bis.mark(MARK_LIMIT_BYTES);
        ImageHeaderParser.ImageType type = parser.parse(bis);
        bis.reset();
        GifBitmapWrapper result = null;
        if (type == ImageHeaderParser.ImageType.GIF) {
            result = decodeGifWrapper(bis, width, height);
        }
        // Decoding the gif may fail even if the type matches.
        if (result == null) {
            // We can only reset the buffered InputStream, so to start from the beginning of the stream, we need to
            // pass in a new source containing the buffered stream rather than the original stream.
            ImageVideoWrapper forBitmapDecoder = new ImageVideoWrapper(bis, source.getFileDescriptor());
            result = decodeBitmapWrapper(forBitmapDecoder, width, height);
        }
        return result;
    }
    
    private GifBitmapWrapper decodeBitmapWrapper(ImageVideoWrapper toDecode, int width, int height) throws IOException {
        GifBitmapWrapper result = null;
        Resource<Bitmap> bitmapResource = bitmapDecoder.decode(toDecode, width, height);
        if (bitmapResource != null) {
            result = new GifBitmapWrapper(bitmapResource, null);
        }
        return result;
    }
    
    ...
}
首先，在decode()方法中，又去调用了另外一个decode()方法的重载。然后在第23行调用了decodeStream()方法，准备从服务器返回的流当中读取数据。decodeStream()方法中会先从流中读取2个字节的数据，来判断这张图是GIF图还是普通的静图，如果是GIF图就调用decodeGifWrapper()方法来进行解码，如果是普通的静图就用调用decodeBitmapWrapper()方法来进行解码。这里我们只分析普通静图的实现流程，GIF图的实现有点过于复杂了，无法在本篇文章当中分析。

然后我们来看一下decodeBitmapWrapper()方法，这里在第52行调用了bitmapDecoder.decode()方法。这个bitmapDecoder是一个ImageVideoBitmapDecoder对象，那么我们来看一下它的代码，如下所示：

public class ImageVideoBitmapDecoder implements ResourceDecoder<ImageVideoWrapper, Bitmap> {
    private final ResourceDecoder<InputStream, Bitmap> streamDecoder;
    private final ResourceDecoder<ParcelFileDescriptor, Bitmap> fileDescriptorDecoder;

    public ImageVideoBitmapDecoder(ResourceDecoder<InputStream, Bitmap> streamDecoder,
            ResourceDecoder<ParcelFileDescriptor, Bitmap> fileDescriptorDecoder) {
        this.streamDecoder = streamDecoder;
        this.fileDescriptorDecoder = fileDescriptorDecoder;
    }
    
    @Override
    public Resource<Bitmap> decode(ImageVideoWrapper source, int width, int height) throws IOException {
        Resource<Bitmap> result = null;
        InputStream is = source.getStream();
        if (is != null) {
            try {
                result = streamDecoder.decode(is, width, height);
            } catch (IOException e) {
                if (Log.isLoggable(TAG, Log.VERBOSE)) {
                    Log.v(TAG, "Failed to load image from stream, trying FileDescriptor", e);
                }
            }
        }
        if (result == null) {
            ParcelFileDescriptor fileDescriptor = source.getFileDescriptor();
            if (fileDescriptor != null) {
                result = fileDescriptorDecoder.decode(fileDescriptor, width, height);
            }
        }
        return result;
    }
    
    ...
}
代码并不复杂，在第14行先调用了source.getStream()来获取到服务器返回的InputStream，然后在第17行调用streamDecoder.decode()方法进行解码。streamDecode是一个StreamBitmapDecoder对象，那么我们再来看这个类的源码，如下所示：

public class StreamBitmapDecoder implements ResourceDecoder<InputStream, Bitmap> {

    ...
    
    private final Downsampler downsampler;
    private BitmapPool bitmapPool;
    private DecodeFormat decodeFormat;
    
    public StreamBitmapDecoder(Downsampler downsampler, BitmapPool bitmapPool, DecodeFormat decodeFormat) {
        this.downsampler = downsampler;
        this.bitmapPool = bitmapPool;
        this.decodeFormat = decodeFormat;
    }
    
    @Override
    public Resource<Bitmap> decode(InputStream source, int width, int height) {
        Bitmap bitmap = downsampler.decode(source, bitmapPool, width, height, decodeFormat);
        return BitmapResource.obtain(bitmap, bitmapPool);
    }
    
    ...
}
可以看到，它的decode()方法又去调用了Downsampler的decode()方法。接下来又到了激动人心的时刻了，Downsampler的代码如下所示：

public abstract class Downsampler implements BitmapDecoder<InputStream> {

    ...
    
    @Override
    public Bitmap decode(InputStream is, BitmapPool pool, int outWidth, int outHeight, DecodeFormat decodeFormat) {
        final ByteArrayPool byteArrayPool = ByteArrayPool.get();
        final byte[] bytesForOptions = byteArrayPool.getBytes();
        final byte[] bytesForStream = byteArrayPool.getBytes();
        final BitmapFactory.Options options = getDefaultOptions();
        // Use to fix the mark limit to avoid allocating buffers that fit entire images.
        RecyclableBufferedInputStream bufferedStream = new RecyclableBufferedInputStream(
                is, bytesForStream);
        // Use to retrieve exceptions thrown while reading.
        // TODO(#126): when the framework no longer returns partially decoded Bitmaps or provides a way to determine
        // if a Bitmap is partially decoded, consider removing.
        ExceptionCatchingInputStream exceptionStream =
                ExceptionCatchingInputStream.obtain(bufferedStream);
        // Use to read data.
        // Ensures that we can always reset after reading an image header so that we can still attempt to decode the
        // full image even when the header decode fails and/or overflows our read buffer. See #283.
        MarkEnforcingInputStream invalidatingStream = new MarkEnforcingInputStream(exceptionStream);
        try {
            exceptionStream.mark(MARK_POSITION);
            int orientation = 0;
            try {
                orientation = new ImageHeaderParser(exceptionStream).getOrientation();
            } catch (IOException e) {
                if (Log.isLoggable(TAG, Log.WARN)) {
                    Log.w(TAG, "Cannot determine the image orientation from header", e);
                }
            } finally {
                try {
                    exceptionStream.reset();
                } catch (IOException e) {
                    if (Log.isLoggable(TAG, Log.WARN)) {
                        Log.w(TAG, "Cannot reset the input stream", e);
                    }
                }
            }
            options.inTempStorage = bytesForOptions;
            final int[] inDimens = getDimensions(invalidatingStream, bufferedStream, options);
            final int inWidth = inDimens[0];
            final int inHeight = inDimens[1];
            final int degreesToRotate = TransformationUtils.getExifOrientationDegrees(orientation);
            final int sampleSize = getRoundedSampleSize(degreesToRotate, inWidth, inHeight, outWidth, outHeight);
            final Bitmap downsampled =
                    downsampleWithSize(invalidatingStream, bufferedStream, options, pool, inWidth, inHeight, sampleSize,
                            decodeFormat);
            // BitmapFactory swallows exceptions during decodes and in some cases when inBitmap is non null, may catch
            // and log a stack trace but still return a non null bitmap. To avoid displaying partially decoded bitmaps,
            // we catch exceptions reading from the stream in our ExceptionCatchingInputStream and throw them here.
            final Exception streamException = exceptionStream.getException();
            if (streamException != null) {
                throw new RuntimeException(streamException);
            }
            Bitmap rotated = null;
            if (downsampled != null) {
                rotated = TransformationUtils.rotateImageExif(downsampled, pool, orientation);
                if (!downsampled.equals(rotated) && !pool.put(downsampled)) {
                    downsampled.recycle();
                }
            }
            return rotated;
        } finally {
            byteArrayPool.releaseBytes(bytesForOptions);
            byteArrayPool.releaseBytes(bytesForStream);
            exceptionStream.release();
            releaseOptions(options);
        }
    }
    
    private Bitmap downsampleWithSize(MarkEnforcingInputStream is, RecyclableBufferedInputStream  bufferedStream,
            BitmapFactory.Options options, BitmapPool pool, int inWidth, int inHeight, int sampleSize,
            DecodeFormat decodeFormat) {
        // Prior to KitKat, the inBitmap size must exactly match the size of the bitmap we're decoding.
        Bitmap.Config config = getConfig(is, decodeFormat);
        options.inSampleSize = sampleSize;
        options.inPreferredConfig = config;
        if ((options.inSampleSize == 1 || Build.VERSION_CODES.KITKAT <= Build.VERSION.SDK_INT) && shouldUsePool(is)) {
            int targetWidth = (int) Math.ceil(inWidth / (double) sampleSize);
            int targetHeight = (int) Math.ceil(inHeight / (double) sampleSize);
            // BitmapFactory will clear out the Bitmap before writing to it, so getDirty is safe.
            setInBitmap(options, pool.getDirty(targetWidth, targetHeight, config));
        }
        return decodeStream(is, bufferedStream, options);
    }
    
    /**
     * A method for getting the dimensions of an image from the given InputStream.
     *
     * @param is The InputStream representing the image.
     * @param options The options to pass to
     *          {@link BitmapFactory#decodeStream(InputStream, android.graphics.Rect,
     *              BitmapFactory.Options)}.
     * @return an array containing the dimensions of the image in the form {width, height}.
     */
    public int[] getDimensions(MarkEnforcingInputStream is, RecyclableBufferedInputStream bufferedStream,
            BitmapFactory.Options options) {
        options.inJustDecodeBounds = true;
        decodeStream(is, bufferedStream, options);
        options.inJustDecodeBounds = false;
        return new int[] { options.outWidth, options.outHeight };
    }
    
    private static Bitmap decodeStream(MarkEnforcingInputStream is, RecyclableBufferedInputStream bufferedStream,
            BitmapFactory.Options options) {
         if (options.inJustDecodeBounds) {
             // This is large, but jpeg headers are not size bounded so we need something large enough to minimize
             // the possibility of not being able to fit enough of the header in the buffer to get the image size so
             // that we don't fail to load images. The BufferedInputStream will create a new buffer of 2x the
             // original size each time we use up the buffer space without passing the mark so this is a maximum
             // bound on the buffer size, not a default. Most of the time we won't go past our pre-allocated 16kb.
             is.mark(MARK_POSITION);
         } else {
             // Once we've read the image header, we no longer need to allow the buffer to expand in size. To avoid
             // unnecessary allocations reading image data, we fix the mark limit so that it is no larger than our
             // current buffer size here. See issue #225.
             bufferedStream.fixMarkLimit();
         }
        final Bitmap result = BitmapFactory.decodeStream(is, null, options);
        try {
            if (options.inJustDecodeBounds) {
                is.reset();
            }
        } catch (IOException e) {
            if (Log.isLoggable(TAG, Log.ERROR)) {
                Log.e(TAG, "Exception loading inDecodeBounds=" + options.inJustDecodeBounds
                        + " sample=" + options.inSampleSize, e);
            }
        }
    
        return result;
    }
    
    ...
}
可以看到，对服务器返回的InputStream的读取，以及对图片的加载全都在这里了。当然这里其实处理了很多的逻辑，包括对图片的压缩，甚至还有旋转、圆角等逻辑处理，但是我们目前只需要关注主线逻辑就行了。decode()方法执行之后，会返回一个Bitmap对象，那么图片在这里其实也就已经被加载出来了，剩下的工作就是如果让这个Bitmap显示到界面上，我们继续往下分析。

回到刚才的StreamBitmapDecoder当中，你会发现，它的decode()方法返回的是一个Resource<Bitmap>对象。而我们从Downsampler中得到的是一个Bitmap对象，因此这里在第18行又调用了BitmapResource.obtain()方法，将Bitmap对象包装成了Resource<Bitmap>对象。代码如下所示：

public class BitmapResource implements Resource<Bitmap> {
    private final Bitmap bitmap;
    private final BitmapPool bitmapPool;

    /**
     * Returns a new {@link BitmapResource} wrapping the given {@link Bitmap} if the Bitmap is non-null or null if the
     * given Bitmap is null.
     *
     * @param bitmap A Bitmap.
     * @param bitmapPool A non-null {@link BitmapPool}.
     */
    public static BitmapResource obtain(Bitmap bitmap, BitmapPool bitmapPool) {
        if (bitmap == null) {
            return null;
        } else {
            return new BitmapResource(bitmap, bitmapPool);
        }
    }
    
    public BitmapResource(Bitmap bitmap, BitmapPool bitmapPool) {
        if (bitmap == null) {
            throw new NullPointerException("Bitmap must not be null");
        }
        if (bitmapPool == null) {
            throw new NullPointerException("BitmapPool must not be null");
        }
        this.bitmap = bitmap;
        this.bitmapPool = bitmapPool;
    }
    
    @Override
    public Bitmap get() {
        return bitmap;
    }
    
    @Override
    public int getSize() {
        return Util.getBitmapByteSize(bitmap);
    }
    
    @Override
    public void recycle() {
        if (!bitmapPool.put(bitmap)) {
            bitmap.recycle();
        }
    }
}
BitmapResource的源码也非常简单，经过这样一层包装之后，如果我还需要获取Bitmap，只需要调用Resource<Bitmap>的get()方法就可以了。

然后我们需要一层层继续向上返回，StreamBitmapDecoder会将值返回到ImageVideoBitmapDecoder当中，而ImageVideoBitmapDecoder又会将值返回到GifBitmapWrapperResourceDecoder的decodeBitmapWrapper()方法当中。由于代码隔得有点太远了，我重新把decodeBitmapWrapper()方法的代码贴一下：

private GifBitmapWrapper decodeBitmapWrapper(ImageVideoWrapper toDecode, int width, int height) throws IOException {
    GifBitmapWrapper result = null;
    Resource<Bitmap> bitmapResource = bitmapDecoder.decode(toDecode, width, height);
    if (bitmapResource != null) {
        result = new GifBitmapWrapper(bitmapResource, null);
    }
    return result;
}
可以看到，decodeBitmapWrapper()方法返回的是一个GifBitmapWrapper对象。因此，这里在第5行，又将Resource<Bitmap>封装到了一个GifBitmapWrapper对象当中。这个GifBitmapWrapper顾名思义，就是既能封装GIF，又能封装Bitmap，从而保证了不管是什么类型的图片Glide都能从容应对。我们顺便来看下GifBitmapWrapper的源码吧，如下所示：

public class GifBitmapWrapper {
    private final Resource<GifDrawable> gifResource;
    private final Resource<Bitmap> bitmapResource;

    public GifBitmapWrapper(Resource<Bitmap> bitmapResource, Resource<GifDrawable> gifResource) {
        if (bitmapResource != null && gifResource != null) {
            throw new IllegalArgumentException("Can only contain either a bitmap resource or a gif resource, not both");
        }
        if (bitmapResource == null && gifResource == null) {
            throw new IllegalArgumentException("Must contain either a bitmap resource or a gif resource");
        }
        this.bitmapResource = bitmapResource;
        this.gifResource = gifResource;
    }
    
    /**
     * Returns the size of the wrapped resource.
     */
    public int getSize() {
        if (bitmapResource != null) {
            return bitmapResource.getSize();
        } else {
            return gifResource.getSize();
        }
    }
    
    /**
     * Returns the wrapped {@link Bitmap} resource if it exists, or null.
     */
    public Resource<Bitmap> getBitmapResource() {
        return bitmapResource;
    }
    
    /**
     * Returns the wrapped {@link GifDrawable} resource if it exists, or null.
     */
    public Resource<GifDrawable> getGifResource() {
        return gifResource;
    }
}
还是比较简单的，就是分别对gifResource和bitmapResource做了一层封装而已，相信没有什么解释的必要。

然后这个GifBitmapWrapper对象会一直向上返回，返回到GifBitmapWrapperResourceDecoder最外层的decode()方法的时候，会对它再做一次封装，如下所示：

@Override
public Resource<GifBitmapWrapper> decode(ImageVideoWrapper source, int width, int height) throws IOException {
    ByteArrayPool pool = ByteArrayPool.get();
    byte[] tempBytes = pool.getBytes();
    GifBitmapWrapper wrapper = null;
    try {
        wrapper = decode(source, width, height, tempBytes);
    } finally {
        pool.releaseBytes(tempBytes);
    }
    return wrapper != null ? new GifBitmapWrapperResource(wrapper) : null;
}
可以看到，这里在第11行，又将GifBitmapWrapper封装到了一个GifBitmapWrapperResource对象当中，最终返回的是一个Resource<GifBitmapWrapper>对象。这个GifBitmapWrapperResource和刚才的BitmapResource是相似的，它们都实现的Resource接口，都可以通过get()方法来获取封装起来的具体内容。GifBitmapWrapperResource的源码如下所示：

public class GifBitmapWrapperResource implements Resource<GifBitmapWrapper> {
    private final GifBitmapWrapper data;

    public GifBitmapWrapperResource(GifBitmapWrapper data) {
        if (data == null) {
            throw new NullPointerException("Data must not be null");
        }
        this.data = data;
    }
    
    @Override
    public GifBitmapWrapper get() {
        return data;
    }
    
    @Override
    public int getSize() {
        return data.getSize();
    }
    
    @Override
    public void recycle() {
        Resource<Bitmap> bitmapResource = data.getBitmapResource();
        if (bitmapResource != null) {
            bitmapResource.recycle();
        }
        Resource<GifDrawable> gifDataResource = data.getGifResource();
        if (gifDataResource != null) {
            gifDataResource.recycle();
        }
    }
}
经过这一层的封装之后，我们从网络上得到的图片就能够以Resource接口的形式返回，并且还能同时处理Bitmap图片和GIF图片这两种情况。

那么现在我们可以回到DecodeJob当中了，它的decodeFromSourceData()方法返回的是一个Resource<T>对象，其实也就是Resource<GifBitmapWrapper>对象了。然后继续向上返回，最终返回到decodeFromSource()方法当中，如下所示：

public Resource<Z> decodeFromSource() throws Exception {
    Resource<T> decoded = decodeSource();
    return transformEncodeAndTranscode(decoded);
}
刚才我们就是从这里跟进到decodeSource()方法当中，然后执行了一大堆一大堆的逻辑，最终得到了这个Resource<T>对象。然而你会发现，decodeFromSource()方法最终返回的却是一个Resource<Z>对象，那么这到底是怎么回事呢？我们就需要跟进到transformEncodeAndTranscode()方法来瞧一瞧了，代码如下所示：

private Resource<Z> transformEncodeAndTranscode(Resource<T> decoded) {
    long startTime = LogTime.getLogTime();
    Resource<T> transformed = transform(decoded);
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Transformed resource from source", startTime);
    }
    writeTransformedToCache(transformed);
    startTime = LogTime.getLogTime();
    Resource<Z> result = transcode(transformed);
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Transcoded transformed from source", startTime);
    }
    return result;
}

private Resource<Z> transcode(Resource<T> transformed) {
    if (transformed == null) {
        return null;
    }
    return transcoder.transcode(transformed);
}
首先，这个方法开头的几行transform还有cache，这都是我们后面才会学习的东西，现在不用管它们就可以了。需要注意的是第9行，这里调用了一个transcode()方法，就把Resource<T>对象转换成Resource<Z>对象了。

而transcode()方法中又是调用了transcoder的transcode()方法，那么这个transcoder是什么呢？其实这也是Glide源码特别难懂的原因之一，就是它用到的很多对象都是很早很早之前就初始化的，在初始化的时候你可能完全就没有留意过它，因为一时半会根本就用不着，但是真正需要用到的时候你却早就记不起来这个对象是从哪儿来的了。

那么这里我来提醒一下大家吧，在第二步load()方法返回的那个DrawableTypeRequest对象，它的构建函数中去构建了一个FixedLoadProvider对象，然后我们将三个参数传入到了FixedLoadProvider当中，其中就有一个GifBitmapWrapperDrawableTranscoder对象。后来在onSizeReady()方法中获取到了这个参数，并传递到了Engine当中，然后又由Engine传递到了DecodeJob当中。因此，这里的transcoder其实就是这个GifBitmapWrapperDrawableTranscoder对象。那么我们来看一下它的源码：

public class GifBitmapWrapperDrawableTranscoder implements ResourceTranscoder<GifBitmapWrapper, GlideDrawable> {
    private final ResourceTranscoder<Bitmap, GlideBitmapDrawable> bitmapDrawableResourceTranscoder;

    public GifBitmapWrapperDrawableTranscoder(
            ResourceTranscoder<Bitmap, GlideBitmapDrawable> bitmapDrawableResourceTranscoder) {
        this.bitmapDrawableResourceTranscoder = bitmapDrawableResourceTranscoder;
    }
    
    @Override
    public Resource<GlideDrawable> transcode(Resource<GifBitmapWrapper> toTranscode) {
        GifBitmapWrapper gifBitmap = toTranscode.get();
        Resource<Bitmap> bitmapResource = gifBitmap.getBitmapResource();
        final Resource<? extends GlideDrawable> result;
        if (bitmapResource != null) {
            result = bitmapDrawableResourceTranscoder.transcode(bitmapResource);
        } else {
            result = gifBitmap.getGifResource();
        }
        return (Resource<GlideDrawable>) result;
    }
    
    ...
}
这里我来简单解释一下，GifBitmapWrapperDrawableTranscoder的核心作用就是用来转码的。因为GifBitmapWrapper是无法直接显示到ImageView上面的，只有Bitmap或者Drawable才能显示到ImageView上。因此，这里的transcode()方法先从Resource<GifBitmapWrapper>中取出GifBitmapWrapper对象，然后再从GifBitmapWrapper中取出Resource<Bitmap>对象。

接下来做了一个判断，如果Resource<Bitmap>为空，那么说明此时加载的是GIF图，直接调用getGifResource()方法将图片取出即可，因为Glide用于加载GIF图片是使用的GifDrawable这个类，它本身就是一个Drawable对象了。而如果Resource<Bitmap>不为空，那么就需要再做一次转码，将Bitmap转换成Drawable对象才行，因为要保证静图和动图的类型一致性，不然逻辑上是不好处理的。

这里在第15行又进行了一次转码，是调用的GlideBitmapDrawableTranscoder对象的transcode()方法，代码如下所示：

public class GlideBitmapDrawableTranscoder implements ResourceTranscoder<Bitmap, GlideBitmapDrawable> {
    private final Resources resources;
    private final BitmapPool bitmapPool;

    public GlideBitmapDrawableTranscoder(Context context) {
        this(context.getResources(), Glide.get(context).getBitmapPool());
    }
    
    public GlideBitmapDrawableTranscoder(Resources resources, BitmapPool bitmapPool) {
        this.resources = resources;
        this.bitmapPool = bitmapPool;
    }
    
    @Override
    public Resource<GlideBitmapDrawable> transcode(Resource<Bitmap> toTranscode) {
        GlideBitmapDrawable drawable = new GlideBitmapDrawable(resources, toTranscode.get());
        return new GlideBitmapDrawableResource(drawable, bitmapPool);
    }
    
    ...
}
可以看到，这里new出了一个GlideBitmapDrawable对象，并把Bitmap封装到里面。然后对GlideBitmapDrawable再进行一次封装，返回一个Resource<GlideBitmapDrawable>对象。

现在再返回到GifBitmapWrapperDrawableTranscoder的transcode()方法中，你会发现它们的类型就一致了。因为不管是静图的Resource<GlideBitmapDrawable>对象，还是动图的Resource<GifDrawable>对象，它们都是属于父类Resource<GlideDrawable>对象的。因此transcode()方法也是直接返回了Resource<GlideDrawable>，而这个Resource<GlideDrawable>其实也就是转换过后的Resource<Z>了。

那么我们继续回到DecodeJob当中，它的decodeFromSource()方法得到了Resource<Z>对象，当然也就是Resource<GlideDrawable>对象。然后继续向上返回会回到EngineRunnable的decodeFromSource()方法，再回到decode()方法，再回到run()方法当中。那么我们重新再贴一下EngineRunnable run()方法的源码：

@Override
public void run() {
    if (isCancelled) {
        return;
    }
    Exception exception = null;
    Resource<?> resource = null;
    try {
        resource = decode();
    } catch (Exception e) {
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            Log.v(TAG, "Exception decoding", e);
        }
        exception = e;
    }
    if (isCancelled) {
        if (resource != null) {
            resource.recycle();
        }
        return;
    }
    if (resource == null) {
        onLoadFailed(exception);
    } else {
        onLoadComplete(resource);
    }
}
也就是说，经过第9行decode()方法的执行，我们最终得到了这个Resource<GlideDrawable>对象，那么接下来就是如何将它显示出来了。可以看到，这里在第25行调用了onLoadComplete()方法，表示图片加载已经完成了，代码如下所示：

private void onLoadComplete(Resource resource) {
    manager.onResourceReady(resource);
}
这个manager就是EngineJob对象，因此这里实际上调用的是EngineJob的onResourceReady()方法，代码如下所示：

class EngineJob implements EngineRunnable.EngineRunnableManager {

    private static final Handler MAIN_THREAD_HANDLER = new Handler(Looper.getMainLooper(), new MainThreadCallback());
    
    private final List<ResourceCallback> cbs = new ArrayList<ResourceCallback>();
    
    ...
    
    public void addCallback(ResourceCallback cb) {
        Util.assertMainThread();
        if (hasResource) {
            cb.onResourceReady(engineResource);
        } else if (hasException) {
            cb.onException(exception);
        } else {
            cbs.add(cb);
        }
    }
    
    @Override
    public void onResourceReady(final Resource<?> resource) {
        this.resource = resource;
        MAIN_THREAD_HANDLER.obtainMessage(MSG_COMPLETE, this).sendToTarget();
    }
    
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
    
    @Override
    public void onException(final Exception e) {
        this.exception = e;
        MAIN_THREAD_HANDLER.obtainMessage(MSG_EXCEPTION, this).sendToTarget();
    }
    
    private void handleExceptionOnMainThread() {
        if (isCancelled) {
            return;
        } else if (cbs.isEmpty()) {
            throw new IllegalStateException("Received an exception without any callbacks to notify");
        }
        hasException = true;
        listener.onEngineJobComplete(key, null);
        for (ResourceCallback cb : cbs) {
            if (!isInIgnoredCallbacks(cb)) {
                cb.onException(exception);
            }
        }
    }
    
    private static class MainThreadCallback implements Handler.Callback {
    
        @Override
        public boolean handleMessage(Message message) {
            if (MSG_COMPLETE == message.what || MSG_EXCEPTION == message.what) {
                EngineJob job = (EngineJob) message.obj;
                if (MSG_COMPLETE == message.what) {
                    job.handleResultOnMainThread();
                } else {
                    job.handleExceptionOnMainThread();
                }
                return true;
            }
            return false;
        }
    }
    
    ...
}
可以看到，这里在onResourceReady()方法使用Handler发出了一条MSG_COMPLETE消息，那么在MainThreadCallback的handleMessage()方法中就会收到这条消息。从这里开始，所有的逻辑又回到主线程当中进行了，因为很快就需要更新UI了。

然后在第72行调用了handleResultOnMainThread()方法，这个方法中又通过一个循环，调用了所有ResourceCallback的onResourceReady()方法。那么这个ResourceCallback是什么呢？答案在addCallback()方法当中，它会向cbs集合中去添加ResourceCallback。那么这个addCallback()方法又是哪里调用的呢？其实调用的地方我们早就已经看过了，只不过之前没有注意，现在重新来看一下Engine的load()方法，如下所示：

public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {

    ...    
    
    public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder, Priority priority, 
            boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
    
        ...
    
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
这次把目光放在第18行上面，看到了吗？就是在这里调用的EngineJob的addCallback()方法来注册的一个ResourceCallback。那么接下来的问题就是，Engine.load()方法的ResourceCallback参数又是谁传过来的呢？这就需要回到GenericRequest的onSizeReady()方法当中了，我们看到ResourceCallback是load()方法的最后一个参数，那么在onSizeReady()方法中调用load()方法时传入的最后一个参数是什么？代码如下所示：

public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {

    ...
    
    @Override
    public void onSizeReady(int width, int height) {
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("Got onSizeReady in " + LogTime.getElapsedMillis(startTime));
        }
        if (status != Status.WAITING_FOR_SIZE) {
            return;
        }
        status = Status.RUNNING;
        width = Math.round(sizeMultiplier * width);
        height = Math.round(sizeMultiplier * height);
        ModelLoader<A, T> modelLoader = loadProvider.getModelLoader();
        final DataFetcher<T> dataFetcher = modelLoader.getResourceFetcher(model, width, height);
        if (dataFetcher == null) {
            onException(new Exception("Failed to load model: \'" + model + "\'"));
            return;
        }
        ResourceTranscoder<Z, R> transcoder = loadProvider.getTranscoder();
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("finished setup for calling load in " + LogTime.getElapsedMillis(startTime));
        }
        loadedFromMemoryCache = true;
        loadStatus = engine.load(signature, width, height, dataFetcher, loadProvider, transformation, 
                transcoder, priority, isMemoryCacheable, diskCacheStrategy, this);
        loadedFromMemoryCache = resource != null;
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("finished onSizeReady in " + LogTime.getElapsedMillis(startTime));
        }
    }
    
    ...
}
请将目光锁定在第29行的最后一个参数，this。没错，就是this。GenericRequest本身就实现了ResourceCallback的接口，因此EngineJob的回调最终其实就是回调到了GenericRequest的onResourceReady()方法当中了，代码如下所示：

public void onResourceReady(Resource<?> resource) {
    if (resource == null) {
        onException(new Exception("Expected to receive a Resource<R> with an object of " + transcodeClass
                + " inside, but instead got null."));
        return;
    }
    Object received = resource.get();
    if (received == null || !transcodeClass.isAssignableFrom(received.getClass())) {
        releaseResource(resource);
        onException(new Exception("Expected to receive an object of " + transcodeClass
                + " but instead got " + (received != null ? received.getClass() : "") + "{" + received + "}"
                + " inside Resource{" + resource + "}."
                + (received != null ? "" : " "
                    + "To indicate failure return a null Resource object, "
                    + "rather than a Resource object containing null data.")
        ));
        return;
    }
    if (!canSetResource()) {
        releaseResource(resource);
        // We can't set the status to complete before asking canSetResource().
        status = Status.COMPLETE;
        return;
    }
    onResourceReady(resource, (R) received);
}

private void onResourceReady(Resource<?> resource, R result) {
    // We must call isFirstReadyResource before setting status.
    boolean isFirstResource = isFirstReadyResource();
    status = Status.COMPLETE;
    this.resource = resource;
    if (requestListener == null || !requestListener.onResourceReady(result, model, target, loadedFromMemoryCache,
            isFirstResource)) {
        GlideAnimation<R> animation = animationFactory.build(loadedFromMemoryCache, isFirstResource);
        target.onResourceReady(result, animation);
    }
    notifyLoadSuccess();
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logV("Resource ready in " + LogTime.getElapsedMillis(startTime) + " size: "
                + (resource.getSize() * TO_MEGABYTE) + " fromCache: " + loadedFromMemoryCache);
        }
}
这里有两个onResourceReady()方法，首先在第一个onResourceReady()方法当中，调用resource.get()方法获取到了封装的图片对象，也就是GlideBitmapDrawable对象，或者是GifDrawable对象。然后将这个值传入到了第二个onResourceReady()方法当中，并在第36行调用了target.onResourceReady()方法。

那么这个target又是什么呢？这个又需要向上翻很久了，在第三步into()方法的一开始，我们就分析了在into()方法的最后一行，调用了glide.buildImageViewTarget()方法来构建出一个Target，而这个Target就是一个GlideDrawableImageViewTarget对象。

那么我们去看GlideDrawableImageViewTarget的源码就可以了，如下所示：

    public class GlideDrawableImageViewTarget extends ImageViewTarget<GlideDrawable> {
        private static final float SQUARE_RATIO_MARGIN = 0.05f;
        private int maxLoopCount;
        private GlideDrawable resource;
        
    public GlideDrawableImageViewTarget(ImageView view) {
        this(view, GlideDrawable.LOOP_FOREVER);
    }
    
    public GlideDrawableImageViewTarget(ImageView view, int maxLoopCount) {
        super(view);
        this.maxLoopCount = maxLoopCount;
    }
    
    @Override
    public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> animation) {
        if (!resource.isAnimated()) {
            float viewRatio = view.getWidth() / (float) view.getHeight();
            float drawableRatio = resource.getIntrinsicWidth() / (float) resource.getIntrinsicHeight();
            if (Math.abs(viewRatio - 1f) <= SQUARE_RATIO_MARGIN
                    && Math.abs(drawableRatio - 1f) <= SQUARE_RATIO_MARGIN) {
                resource = new SquaringDrawable(resource, view.getWidth());
            }
        }
        super.onResourceReady(resource, animation);
        this.resource = resource;
        resource.setLoopCount(maxLoopCount);
        resource.start();
    }
    
    @Override
    protected void setResource(GlideDrawable resource) {
        view.setImageDrawable(resource);
    }
    
    @Override
    public void onStart() {
        if (resource != null) {
            resource.start();
        }
    }
    
    @Override
    public void onStop() {
        if (resource != null) {
            resource.stop();
        }
    }
    }

在GlideDrawableImageViewTarget的onResourceReady()方法中做了一些逻辑处理，包括如果是GIF图片的话，就调用resource.start()方法开始播放图片，但是好像并没有看到哪里有将GlideDrawable显示到ImageView上的逻辑。

确实没有，不过父类里面有，这里在第25行调用了super.onResourceReady()方法，GlideDrawableImageViewTarget的父类是ImageViewTarget，我们来看下它的代码吧：

public abstract class ImageViewTarget<Z> extends ViewTarget<ImageView, Z> implements GlideAnimation.ViewAdapter {

    ...
    
    @Override
    public void onResourceReady(Z resource, GlideAnimation<? super Z> glideAnimation) {
        if (glideAnimation == null || !glideAnimation.animate(resource, this)) {
            setResource(resource);
        }
    }
    
    protected abstract void setResource(Z resource);

}

可以看到，在ImageViewTarget的onResourceReady()方法当中调用了setResource()方法，而ImageViewTarget的setResource()方法是一个抽象方法，具体的实现还是在子类那边实现的。

那子类的setResource()方法是怎么实现的呢？回头再来看一下GlideDrawableImageViewTarget的setResource()方法，没错，调用的view.setImageDrawable()方法，而这个view就是ImageView。代码执行到这里，图片终于也就显示出来了。

那么，我们对Glide执行流程的源码分析，到这里也终于结束了。

总结
真是好长的一篇文章，这也可能是我目前所写过的最长的一篇文章了。如果你之前没有读过Glide的源码，真的很难相信，这短短一行代码：

Glide.with(this).load(url).into(imageView);
1
背后竟然蕴藏着如此极其复杂的逻辑吧？

不过Glide也并不是有意要将代码写得如此复杂，实在是因为Glide的功能太强大了，而上述代码只是使用了Glide最最基本的功能而已。



# downloadOnly(int width, int height)方法

工作原理到底是什么样的呢？我们来简单快速地看一下它的源码吧。

首先在DrawableTypeRequest类当中可以找到定义这个方法的地方，如下所示：

```kotlin
public class DrawableTypeRequest<ModelType> extends DrawableRequestBuilder<ModelType>
        implements DownloadOptions {
    ...

public FutureTarget<File> downloadOnly(int width, int height) {
    return getDownloadOnlyRequest().downloadOnly(width, height);
}

private GenericTranscodeRequest<ModelType, InputStream, File> getDownloadOnlyRequest() {
    return optionsApplier.apply(new GenericTranscodeRequest<ModelType, InputStream, File>(
        File.class, this, streamModelLoader, InputStream.class, File.class, optionsApplier));
}

}
```

这里会先调用getDownloadOnlyRequest()方法得到一个GenericTranscodeRequest对象，然后再调用它的downloadOnly()方法，代码如下所示：

```kotlin
public class GenericTranscodeRequest<ModelType, DataType, ResourceType>
    implements DownloadOptions {
    ...
public FutureTarget<File> downloadOnly(int width, int height) {
    return getDownloadOnlyRequest().into(width, height);
}

private GenericRequestBuilder<ModelType, DataType, File, File> getDownloadOnlyRequest() {
    ResourceTranscoder<File, File> transcoder = UnitTranscoder.get();
    DataLoadProvider<DataType, File> dataLoadProvider = glide.buildDataProvider(dataClass, File.class);
    FixedLoadProvider<ModelType, DataType, File, File> fixedLoadProvider =
        new FixedLoadProvider<ModelType, DataType, File, File>(modelLoader, transcoder, dataLoadProvider);
    return optionsApplier.apply(
            new GenericRequestBuilder<ModelType, DataType, File, File>(fixedLoadProvider,
            File.class, this))
            .priority(Priority.LOW)
            .diskCacheStrategy(DiskCacheStrategy.SOURCE)
            .skipMemoryCache(true);
}
}
```

这里又是调用了一个getDownloadOnlyRequest()方法来构建了一个图片下载的请求，getDownloadOnlyRequest()方法会返回一个GenericRequestBuilder对象，接着调用它的into(width, height)方法，我们继续跟进去瞧一瞧：

```kotlin
public FutureTarget<TranscodeType> into(int width, int height) {
    final RequestFutureTarget<ModelType, TranscodeType> target =
            new RequestFutureTarget<ModelType, TranscodeType>(glide.getMainHandler(), width, height);
    glide.getMainHandler().post(new Runnable() {
        @Override
        public void run() {
            if (!target.isCancelled()) {
                into(target);
            }
        }
    });
    return target;
}
```


可以看到，这里首先是new出了一个RequestFutureTarget对象，RequestFutureTarget也是Target的子类之一。然后通过Handler将线程切回到主线程当中，再将这个RequestFutureTarget传入到into()方法当中。

那么也就是说，其实这里就是调用了接收Target参数的into()方法，然后Glide就开始执行正常的图片加载逻辑了。那么现在剩下的问题就是，这个RequestFutureTarget中到底处理了些什么逻辑？我们打开它的源码看一看：

```kotlin
public class RequestFutureTarget<T, R> implements FutureTarget<R>, Runnable {
    ...
@Override
public R get() throws InterruptedException, ExecutionException {
    try {
        return doGet(null);
    } catch (TimeoutException e) {
        throw new AssertionError(e);
    }
}

@Override
public R get(long time, TimeUnit timeUnit) throws InterruptedException, ExecutionException, 
    TimeoutException {
    return doGet(timeUnit.toMillis(time));
}

@Override
public void getSize(SizeReadyCallback cb) {
    cb.onSizeReady(width, height);
}

@Override
public synchronized void onLoadFailed(Exception e, Drawable errorDrawable) {
    exceptionReceived = true;
    this.exception = e;
    waiter.notifyAll(this);
}

@Override
public synchronized void onResourceReady(R resource, GlideAnimation<? super R> glideAnimation) {
    resultReceived = true;
    this.resource = resource;
    waiter.notifyAll(this);
}

private synchronized R doGet(Long timeoutMillis) throws ExecutionException, InterruptedException, 
    TimeoutException {
    if (assertBackgroundThread) {
        Util.assertBackgroundThread();
    }

    if (isCancelled) {
        throw new CancellationException();
    } else if (exceptionReceived) {
        throw new ExecutionException(exception);
    } else if (resultReceived) {
        return resource;
    }

    if (timeoutMillis == null) {
        waiter.waitForTimeout(this, 0);
    } else if (timeoutMillis > 0) {
        waiter.waitForTimeout(this, timeoutMillis);
    }

    if (Thread.interrupted()) {
        throw new InterruptedException();
    } else if (exceptionReceived) {
        throw new ExecutionException(exception);
    } else if (isCancelled) {
        throw new CancellationException();
    } else if (!resultReceived) {
        throw new TimeoutException();
    }

    return resource;
}

static class Waiter {

    public void waitForTimeout(Object toWaitOn, long timeoutMillis) throws InterruptedException {
        toWaitOn.wait(timeoutMillis);
    }

    public void notifyAll(Object toNotify) {
        toNotify.notifyAll();
    }
}

...
}
```

这里我对RequestFutureTarget的源码做了一些精简，我们只看最主要的逻辑就可以了。

刚才我们已经学习过了downloadOnly()方法的基本用法，在调用了downloadOnly()方法之后，再调用FutureTarget的get()方法，就能获取到下载的图片文件了。而downloadOnly()方法返回的FutureTarget对象其实就是这个RequestFutureTarget，因此我们直接来看它的get()方法就行了。

RequestFutureTarget的get()方法中又调用了一个doGet()方法，而doGet()方法才是真正处理具体逻辑的地方。首先在doGet()方法中会判断当前是否是在子线程当中，如果不是的话会直接抛出一个异常。然后下面会判断下载是否已取消、或者已失败，如果是已取消或者已失败的话都会直接抛出一个异常。接下来会根据resultReceived这个变量来判断下载是否已完成，如果这个变量为true的话，就直接把结果进行返回。

那么如果下载还没有完成呢？我们继续往下看，接下来就进入到一个wait()当中，把当前线程给阻塞住，从而阻止代码继续往下执行。这也是为什么downloadOnly(int width, int height)方法要求必须在子线程当中使用，因为它会对当前线程进行阻塞，如果在主线程当中使用的话，那么就会让主线程卡死，从而用户无法进行任何其他操作。

那么现在线程被阻塞住了，什么时候才能恢复呢？答案在onResourceReady()方法中。可以看到，onResourceReady()方法中只有三行代码，第一行把resultReceived赋值成true，说明图片文件已经下载好了，这样下次再调用get()方法时就不会再阻塞线程，而是可以直接将结果返回。第二行把下载好的图片文件赋值到一个全局的resource变量上面，这样doGet()方法就也可以访问到它。第三行notifyAll一下，通知所有wait的线程取消阻塞，这个时候图片文件已经下载好了，因此doGet()方法也就可以返回结果了。

好的，这就是downloadOnly(int width, int height)方法的基本用法和实现原理



# 相关问题

<font color='orange'>Q：</font>



### 原理

图片加载框架：Glide实现原理

Glide：加载、缓存、LRU算法（LRUCache原理）

Glide如何加载GIF

Glide如何确定图片加载完毕？

Glide生命周期是如何绑定的？

### 缓存

Glide的缓存实现？

Glide的图片三级缓存

Glide缓存特点

Glide内存缓存如何控制大小？

Glide为我们做了哪些内存优化

LruCache的底层实现？

图片缓存框架设计

### 对比

Fresco与Glide的对比

Glide和Picasso有什么区别？

### 开放&拓展

Glide的优点

如何自己设计一个大图加载框架



# 参考

[Android图片加载框架最全解析（二），从源码的角度理解Glide的执行流程](https://blog.csdn.net/guolin_blog/article/details/53939176)

[深入理解Glide源码：三条主线分析 Glide 执行流程](https://juejin.cn/post/7028830670692548639)

[Android 【手撕Glide】--Glide缓存机制](https://www.jianshu.com/p/b85f89fce019)

[Glide 源码分析解读-基于最新版Glide 4.9.0](https://www.jianshu.com/p/9bb50924d42a)

[Glide 源码分析解读-缓存模块-基于最新版Glide 4.9.0](https://www.jianshu.com/p/62b7f990ee83)

[Carson带你学Android：手把手带你深入图片加载库Glide源码分析](https://www.jianshu.com/p/216df89bf59c)