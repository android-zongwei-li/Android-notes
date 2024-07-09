# 前言

`LeakCanary`是一个内存泄漏检测框架，有以下几个特点

1. 不需要手动初始化
2. 可自动检测内存泄漏并通过通知报警
3. 支持dump hprof 文件分析内存使用情况
4. 不能用于线上

本文主要梳理`LeakCanary`内存泄漏检测的主要流程并回答以下问题

1. 说一下`LeakCanary`检测内存泄漏的原理与基本流程
2. `LeakCanary`是如何初始化的？
3. 说一下`LeakCanary`是如何查找内存泄露的？
4. 为什么`LeakCanary`不能用于线上？

# LeakCanary 原理

LeakCanary 是通过在 Application 的 registerActivityLifecycleCallbacks 方法实现对 Activity 销毁监听的，该方法主要用来统一管理所有 Activity 的生命周期。所有 Activity 在销毁时在其 OnDestory 方法中都会回调 ActivityLifecycleCallbacks 的 onActivityDestroyed 方法，而 LeakCanary 要做的就是在该方法中调用 RefWatcher.watch 方法实现对 Activity 进行内存泄漏监控。 那么，LeakCanary 是如何判断某个 Activity 可能会发生内存泄漏呢？

答案是：WeakReference 和 ReferenceQueue，即 LeakCanary 利用了 Java 的 WeakReference 和 ReferenceQueue，通过将 Activity 包装到 WeakReference 中，被 WeakReference 包装过的 Activity 对象如果能够被回收，则说明引用可达，垃圾回收器就会将该 WeakReference 引用存放到 ReferenceQueue 中。假如我们要监视某个 Activity 对象，LeakCanary 就会去 ReferenceQueue 找这个对象的引用，如果找到了，说明该对象是引用可达的，能被 GC 回收，如果没有找到，说明该对象有可能发生了内存泄漏。最后，LeakCanary 会将 Java 堆转储到一个 .hprof 文件中，再使用 Shark（堆分析工具）析 .hprof 文件并定位堆转储中“滞留”的对象，并对每个"滞留"的对象找出 GC roots 的最短强引用路径，并确定是否是泄露，如果泄漏，建立导致泄露的引用链。最后，再将分析完毕的结果以通知的形式展现出来。

## 1. `LeakCanary`检测内存泄漏的原理与基本流程

### 1.1 内存泄漏的原理

内存泄漏的原因：不再需要的对象依然被引用，导致对象被分配的内存无法被回收。
例如：一个`Activity`实例对象在调用了`onDestory`方法后是不再被需要的，如果存储了一个引用`Activity`对象的静态域，将导致`Activity`无法被垃圾回收器回收。
引用链来自于垃圾回收器的可达性分析算法：当一个对象到`GC Roots` 没有任何引用链相连时，则证明此对象是不可用的。如图：
 ![img](images/LeakCanary原理/1)
 对象`object5`、`object6`、`object7` 虽然互相有关联，但是它们到 `GC Roots` 是不可达的，所以它们将会被判定为是可回收的对象。
 在`Java`语言中，可作为`GC Roots`的对象包括下面几种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象。
- 方法区中静态属性引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中JNI（即一般说的Native方法）引用的对象。

### 1.2 `LeakCanary`检测内存泄漏的基本流程

知道了内存泄漏的原理，我们可以推测到`LeakCanary`的基本流程大概是怎样的

1. 检测：在页面关闭后触发检测(不再需要的对象)
2. 收集可疑对象：触发`GC`，然后获取仍然存在的对象，这些是可能泄漏的
3. 分析：`dump heap`然后分析`hprof`文件，构建可能泄漏的对象与`GCRoot`间的引用链，如果存在则证明泄漏
4. 存储与通知：存储结果并使用通知提醒用户存在泄漏

总体流程图如下所示：
 ![img](images/LeakCanary原理/2)

1. `ObjectWatcher` 创建了一个`KeyedWeakReference`来监视对象.
2. 在后台线程中，延时检查引用是否已被清除，如果没有则触发`GC`
3. 如果引用一直没有被清除，它会`dumps the heap` 到一个`.hprof` 文件中，然后将`.hprof` 文件存储到文件系统。
4. 分析过程主要在`HeapAnalyzerService`中进行，`Leakcanary2.0`中使用`Shark`来解析`hprof`文件。
5. `HeapAnalyzer` 获取`hprof`中的所有`KeyedWeakReference`，并获取`objectId`
6. `HeapAnalyzer`计算`objectId`到`GC Root`的最短强引用链路径来确定是否有泄漏，然后构建导致泄漏的引用链。
7. 将分析结果存储在数据库中，并显示泄漏通知。

## 2. `LeakCanary`是如何自动安装的？

`LeakCanary`的使用只需要添加依赖便可以自动初始化，是通过`ContentProvider`实现的

```kotlin
internal sealed class AppWatcherInstaller : ContentProvider() {
  internal class MainProcess : AppWatcherInstaller()

  internal class LeakCanaryProcess : AppWatcherInstaller()

  override fun onCreate(): Boolean {
    val application = context!!.applicationContext as Application
    AppWatcher.manualInstall(application)
    return true
  }
}  
```

当启动`App`时，一般启动顺序为：`Application`->`attachBaseContext` =====>`ContentProvider`->`onCreate` =====>`Application`->`onCreate`
 `ContentProvider`会在`Application.onCreate`前初始化，这样就调用到了`LeakCanary`的初始化方法，实现了免手动初始化。

### 2.1 跨进程初始化

注意，`AppWatcherInstaller`有两个子类，`MainProcess`与`LeakCanaryProcess`
其中默认使用`MainProcess`，会在`App`进程初始化
有时我们考虑到`LeakCanary`比较耗内存，需要在独立进程初始化
使用`leakcanary-android-process`模块的时候，会在一个新的进程中去开启`LeakCanary`

### 2.2 `LeakCanary2.0`手动初始化的方法

`LeakCanary`在检测内存泄漏时比较耗时，同时会打断`App`操作，在不需要检测时的体验并不太好
所以虽然`LeakCanary`可以自动初始化，但有时其实还是需要手动初始化

`LeakCanary`的自动初始化可以手动关闭

```xml
<?xml version="1.0" encoding="utf-8"?>
 <resources>
      <bool name="leak_canary_watcher_auto_install">false</bool>
 </resources>
```

1. 然后在需要初始化的时候，调用`AppWatcher.manualInstall`即可
2. 是否开始`dump`与分析开头：`LeakCanary.config = LeakCanary.config.copy(dumpHeap = false)`
3. 桌面图标开头：重写`R.bool.leak_canary_add_launcher_icon`或者调用`LeakCanary.showLeakDisplayActivityLauncherIcon(false)`

### 2.3 小结

`LeakCanary`利用`ContentProvier`进行了初始化。
`ContentProvier` 会在`Application.onCreate`之前被加载，`LeakCanary`在其`onCreate()`方法中调用了`AppWatcher.manualInstall`进行初始化。
这种写法虽然方便，免去了初始化的步骤，但是可能会带来启动耗时的问题，用户不能控制初始化的时机，这也是谷歌推出`StartUp`的原因
不过对于`LeakCanary`这个问题并不严重，因为它只在`Debug`阶段被依赖

## 3.`LeakCanary`如何检测内存泄漏?

### 3.1 首先我们来看下初始化时做了什么?

当初始化时，调用了`AppWatcher.manualInstall` ：

```kotlin
  @JvmOverloads
  fun manualInstall(
    application: Application,
    retainedDelayMillis: Long = TimeUnit.SECONDS.toMillis(5),
    watchersToInstall: List<InstallableWatcher> = appDefaultWatchers(application)
  ) {
    ....
    watchersToInstall.forEach {
      it.install()
    }
  }

  fun appDefaultWatchers(
    application: Application,
    reachabilityWatcher: ReachabilityWatcher = objectWatcher
  ): List<InstallableWatcher> {
    return listOf(
      ActivityWatcher(application, reachabilityWatcher),
      FragmentAndViewModelWatcher(application, reachabilityWatcher),
      RootViewWatcher(reachabilityWatcher),
      ServiceWatcher(reachabilityWatcher)
    )
  }
```

可以看出，初始化时安装了一些`Watcher`，即在默认情况下，只会观察 `Activity`，`Fragment`，`RootView`，`Service` 这些对象是否泄漏
如果需要观察其他对象，需要手动添加并处理

### 3.2 `LeakCanary`如何触发检测?

如上文所述，在初始化时会安装一些`Watcher`，以`ActivityWatcher`为例

```kotlin
class ActivityWatcher(
  private val application: Application,
  private val reachabilityWatcher: ReachabilityWatcher
) : InstallableWatcher {

  private val lifecycleCallbacks =
    object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
      override fun onActivityDestroyed(activity: Activity) {
        reachabilityWatcher.expectWeaklyReachable(
          activity, "${activity::class.java.name} received Activity#onDestroy() callback"
        )
      }
    }

  override fun install() {
    application.registerActivityLifecycleCallbacks(lifecycleCallbacks)
  }

  override fun uninstall() {
    application.unregisterActivityLifecycleCallbacks(lifecycleCallbacks)
  }
}
```

可以看到在`Activity.onDestory`时，就会触发检测内存泄漏

### 3.3 `LeakCanary`如何检测可能泄漏的对象?

从上面可以看出，`Activity`关闭后会调用到`ObjectWatcher.expectWeaklyReachable`

```kotlin
  @Synchronized override fun expectWeaklyReachable(
    watchedObject: Any,
    description: String
  ) {
    if (!isEnabled()) {
      return
    }
    removeWeaklyReachableObjects()
    val key = UUID.randomUUID()
      .toString()
    val watchUptimeMillis = clock.uptimeMillis()
    val reference =
      KeyedWeakReference(watchedObject, key, description, watchUptimeMillis, queue)
    SharkLog.d {
      "Watching " +
        (if (watchedObject is Class<*>) watchedObject.toString() else "instance of ${watchedObject.javaClass.name}") +
        (if (description.isNotEmpty()) " ($description)" else "") +
        " with key $key"
    }

    watchedObjects[key] = reference
    checkRetainedExecutor.execute {
      moveToRetained(key)
    }
  }

private fun removeWeaklyReachableObjects() {
    // WeakReferences are enqueued as soon as the object to which they point to becomes weakly
    // reachable. This is before finalization or garbage collection has actually happened.
    var ref: KeyedWeakReference?
    do {
      ref = queue.poll() as KeyedWeakReference?
      if (ref != null) {
        watchedObjects.remove(ref.key)
      }
    } while (ref != null)
  }  
```

1. 传入的观察对象都会被存储在`watchedObjects`中
2. 会为每个`watchedObject`生成一个`KeyedWeakReference`弱引用对象并与一个`queue`关联，当对象被回收时，该弱引用对象将进入`queue`当中
3. 在检测过程中，会调用多次`removeWeaklyReachableObjects`，将已回收对象从`watchedObjects`中移除
4. 如果`watchedObjects`中没有移除对象，证明它没有被回收，那么就会调用`moveToRetained`

### 3.4 `LeakCanary`触发堆快照，生成`hprof`文件

`moveToRetained`之后会调用到`HeapDumpTrigger.checkRetainedInstances`方法
`checkRetainedInstances()` 方法是确定泄露的最后一个方法了。
这里会确认引用是否真的泄露，如果真的泄露，则发起 `heap dump`，分析 `dump` 文件，找到引用链

```kotlin
private fun checkRetainedObjects() {
    var retainedReferenceCount = objectWatcher.retainedObjectCount

    if (retainedReferenceCount > 0) {
      gcTrigger.runGc()
      retainedReferenceCount = objectWatcher.retainedObjectCount
    }

    if (checkRetainedCount(retainedReferenceCount, config.retainedVisibleThreshold)) return

    val now = SystemClock.uptimeMillis()
    val elapsedSinceLastDumpMillis = now - lastHeapDumpUptimeMillis
    if (elapsedSinceLastDumpMillis < WAIT_BETWEEN_HEAP_DUMPS_MILLIS) {
      onRetainInstanceListener.onEvent(DumpHappenedRecently)
      ....
      return
    }

    dismissRetainedCountNotification()
    val visibility = if (applicationVisible) "visible" else "not visible"
    dumpHeap(
      retainedReferenceCount = retainedReferenceCount,
      retry = true,
      reason = "$retainedReferenceCount retained objects, app is $visibility"
    ) 
}

  private fun dumpHeap(
    retainedReferenceCount: Int,
    retry: Boolean,
    reason: String
  ) {
     ....
  	 heapDumper.dumpHeap()
  	 ....
     lastDisplayedRetainedObjectCount = 0
     lastHeapDumpUptimeMillis = SystemClock.uptimeMillis()
     objectWatcher.clearObjectsWatchedBefore(heapDumpUptimeMillis)
     HeapAnalyzerService.runAnalysis(
       context = application,
       heapDumpFile = heapDumpResult.file,
       heapDumpDurationMillis = heapDumpResult.durationMillis,
       heapDumpReason = reason
     )
 }
}
```

1. 如果`retainedObjectCount`数量大于0，则进行一次`GC`，避免额外的`Dump`
2. 默认情况下，如果`retainedReferenceCount<5`，不会进行`Dump`，节省资源
3. 如果两次`Dump`之间时间少于60s，也会直接返回，避免频繁`Dump`
4. 调用`heapDumper.dumpHeap()`进行真正的`Dump`操作
5. `Dump`之后，要删除已经处理过了的引用
6. 调用`HeapAnalyzerService.runAnalysis`对结果进行分析

### 3.5 `LeakCanary`如何分析`hprof`文件

分析`hprof`文件的工作主要是在`HeapAnalyzerService`类中完成的
关于`Hprof`文件的解析细节，就需要牵扯到`Hprof`二进制文件协议，通过阅读协议文档，`hprof`的二进制文件结构大概如下：
 ![img](images/LeakCanary原理/3)

解析流程如下所示:
 ![img](images/LeakCanary原理/4)

1. 解析文件头信息，得到解析开始位置
2. 根据头信息创建`Hprof`文件对象
3. 建内存索引
4. 使用`hprof`对象和索引构建`Graph`对象
5. 查找可能泄漏的对象与`GCRoot`间的引用链来判断是否存在泄漏(使用广度优先算法在`Graph`中查找)

`Leakcanary2.0`较之前的版本最大变化是改由`kotlin`实现以及开源了自己实现的`hprof`解析的代码，总体的思路是根据`hprof`文件的二进制协议将文件的内容解析成一个图的数据结构，然后广度遍历这个图找到最短路径，路径的起始就是`GCRoot`对象，结束就是泄漏的对象

具体分析可见：[Android内存泄漏检测之LeakCanary2.0（Kotlin版）的实现原理](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F360944586)

### 3.6 泄漏结果存储与通知

结果的存储与通知主要在`DefaultOnHeapAnalyzedListener`中完成

```kotlin
override fun onHeapAnalyzed(heapAnalysis: HeapAnalysis) {
    SharkLog.d { "\u200B\n${LeakTraceWrapper.wrap(heapAnalysis.toString(), 120)}" }

    val db = LeaksDbHelper(application).writableDatabase
    val id = HeapAnalysisTable.insert(db, heapAnalysis)
    db.releaseReference()
    ...

    if (InternalLeakCanary.formFactor == TV) {
      showToast(heapAnalysis)
      printIntentInfo()
    } else {
      showNotification(screenToShow, contentTitle)
    }
  }
```

1. 存储泄漏分析结果到数据库中
2. 展示通知，提醒用户去查看内存泄漏情况

## 4.为什么`LeakCanary`不能用于线上?

理解了`LeakCanary`判定对象泄漏后所做的工作后就知道，直接将`LeakCanary`应用于线上会有如下一些问题：

1. 每次内存泄漏以后，都会生成一个`.hprof`文件，然后解析，并将结果写入`.hprof.result`。增加手机负担，引起手机卡顿等问题。
2. 多次调用`GC`，可能会对线上性能产生影响
3. 同样的泄漏问题，会重复生成 `.hprof` 文件，重复分析并写入磁盘。
4. `.hprof`文件较大，信息回捞成问题。

了解了这些问题，我们可以尝试提出一些解决方案：

1. 可以根据手机信息来设定一个内存阈值 `M` ，当已使用内存小于 `M `时，如果此时有内存泄漏，只将泄漏对象的信息放入内存当中保存，不生成`.hprof`文件。当已使用大于 `M` 时，生成`.hprof`文件
2. 当引用链路相同时，可根据实际情况去重。
3. 不直接回捞`.hprof`文件，可以选择回捞分析的结果
4. 可以尝试将已泄漏对象存储在数据库中，一个用户同一个泄漏只检测一次，减少对用户的影响

以上想法并没有经过实际验证，仅供参考

## 总结

当引入`LeakCanary`后，会自动安装并且开始分析内存泄漏并报警，主要分为以下几步

1. 自动安装
2. 检测可能泄漏的对象
3. 堆快照，生成`hprof`文件
4. 分析`hprof`文件，生成引用链
5. 对泄漏进行分类并通知



# 问题

<font color='orange'>Q：内存泄漏检测框架：LeakCanary实现原理</font>

1. 自动安装
2. 检测可能泄漏的对象
3. 堆快照，生成`hprof`文件
4. 分析`hprof`文件，生成引用链
5. 对泄漏进行分类并通知

<font color='orange'>Q：leakCannary中如何判断一个对象是否被回收？</font>

通过 WeakReference + ReferenceQueue 来检测，当一个对象只被 WeakReference 持有时，此时触发 GC 后，会把 WeakReference 添加到 ReferenceQueue 中，所以可以在 GC 后检测 ReferenceQueue 中是否有 WeakReference 对象来判断监听对象是否被回收掉。

<font color='orange'>Q：LeakCanary核心原理源码</font>



<font color='orange'>Q：LeakCanray2.0为啥不需要在application里调install？</font>

初始化过程被移动到了`ContentProvider`中。由于`ContentProvider`在`Application`的`onCreate`方法之前就已经被创建和初始化，因此可以不在`Application` 中再调用初始化方法。

利用了`ContentProvider`的自动注册与初始化机制。好处：减少手动配置。



# 参考

[【带着问题学】关于LeakCanary2.0你应该知道的知识点](https://juejin.cn/post/6968084138125590541)