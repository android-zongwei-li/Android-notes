> 源码分析基于：2.3.0

## LiveData

`LiveData` 是一种可观察的数据持有类，下面是常用的写法：

```java
public class DataViewModel extends AndroidViewModel {
    LiveData<String> dataSource = new MutableLiveData<>();

    public DataViewModel(@NonNull Application application) {
        super(application);
    }

    public void postValue(String text) {
        ((MutableLiveData<String>)dataSource).postValue(text);
    }
}

ViewModelProvider provider = new ViewModelProvider(this, ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication()));
DataViewModel model = provider.get(DataViewModel.class);
model.dataSource.observe(this, new Observer<String>() {
    @Override
    public void onChanged(String s) {
        Toast.makeText(getApplicationContext(), s, Toast.LENGTH_LONG).show();
    }
});
```

一般情况下，`LiveData` 和 `ViewModel` 都是搭配使用的，可以在 `Activity` 和 `Fragment` 范围内提供复用，并处理因配置发生改变，而重建的情况。

> 关于 `ViewModel` 更详细的理解，可以阅读之前的文章：[Jetpack中的ViewModel](https://linxiaotao.github.io/2019/01/05/Jetpack中的ViewModel/)

如果了解过 [RxJava](https://github.com/ReactiveX/RxJava) 的同学，对这种**观察者**的设计模式应该有点了解，`LiveData` 也是同样的思想，不过和 RxJava 不同的是：`LiveData` 使用 `Lifecycle` 将数据源和 `Activity` 和 `Fragment` 的生命周期结合起来，避免内存泄漏等情况。这个特性也是本文的重点。

> 关于 Lifecycle 更详细的理解，可以阅读之前的文章：[Jetpack中的Lifecycle](https://linxiaotao.github.io/2019/01/14/Jetpack中的Lifecycle/)

建议在阅读以下内容时，先理解下 `Lifecycle`，因为 `LiveData` 能感应 `Activity` 和 `Fragment` 的生命周期就是基于 `Lifecycle` 实现的。

### 源码解析

#### 各个模块的作用

在理解源码之前，我们先理清组成 `LiveData` 各个模块的作用。

##### LiveData

`LiveData` 表示一个可观察的数据源。在 `LiveData` 的设计中，它被设计为一个纯粹的数据源，对外只提供了：

- observe(LifecycleOwner,Observer)

  表示注册一个观察者（`Observer`），这里同时需要提供一个 `LifecycleOwner` 用于提供 `Lifecycle`

- observeForever(Observer)

  功能和 `observe(LifecycleOwner,Observer)` 类似，但缺少感应生命周期的功能

- removeObserver(Observer) 和 removeObservers(LifecycleOwner)

  表示删除观察者，后者则是删除同个 `LifecycleOwner` 范围的 `Observer`

- getValue()

  表示获取当前值

- hasObservers()

  表示是否存在 `Observer`

- hasActiveObservers()

  表示是否存在活动的 `Observer`，至于怎么去定义为活动的，后面会讲到

`LiveData` 是一个纯粹的数据源，不包含添加数据的公开 API，只能通过使用 `MutableLiveData` 去添加数据

> 这是一种不错的设计理念，单个组件功能越纯粹，代码的耦合度就越低。

##### Observer

`Observer` 表示观察者，响应数据发生变化。API 也非常简单：`onChange(T)`，这里需要注意的是：`onChange()` 中数据可以为 `NULL`，这和 `RxJava2` 的设计是不一样的，这个很难说，哪个设计更优。但根据 [JakeWharton](https://github.com/JakeWharton) 的回答是：It’s not RxJava’s choice, it’s in the reactive streams spec.

> 关于 `RxJava2` 不能传递 `NULL` 的讨论，可以参考这篇[文章](https://github.com/ReactiveX/RxJava/issues/4644)

#### observe

调用 `LiveData.observe(LifecycleOwner,Observer)` 去注册一个订阅者：

```java
@MainThread                                                                                 
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
    // 注意，只能主线程调用
    assertMainThread("observe");                                                     
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {                       
        // ignore                                                                   
        return;                                                                     
    }
    // 使用 LifecycleBoundObserver 包装类
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
    // mObservers 为 Map<Observer,ObserverWrapper>
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);                   
    if (existing != null && !existing.isAttachedTo(owner)) {
        // 存在不同 LifecycleOwner 但相同 observer，抛异常
        // 确保observer只会观察一个LifecycleOwner
        throw new IllegalArgumentException("Cannot add the same observer"           
                + " with different lifecycles");                                     
    }
    
    if (existing != null) {                                                         
        return;                                                                     
    }                                                                                       
    owner.getLifecycle().addObserver(wrapper);                                       
}

// LifecycleBoundObserver
@Override                                     
boolean isAttachedTo(LifecycleOwner owner) {  
    return mOwner == owner;                   
}
```

`LifecycleBoundObserver` 实现了 `GenericLifecycleObserver` 接口，用于响应 `Activity` 和 `Fragment` 的生命周期

```java
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {   
    @NonNull                                                                         
    final LifecycleOwner mOwner;                                                                                                                                
    LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {    
        super(observer);                                                             
        mOwner = owner;                                                             
    }   
    
    @Override                                                                       
    boolean shouldBeActive() { 
        // 检测是否为活动状态
        return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);           
    }               
    
    @Override                                                                       
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {       
        Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
        if (currentState == DESTROYED) {
            removeObserver(mObserver);
            return;
        }
        Lifecycle.State prevState = null;
        while (prevState != currentState) {
            prevState = currentState;
            activeStateChanged(shouldBeActive());
            currentState = mOwner.getLifecycle().getCurrentState();
        }                       
    }                                                                               
    
    @Override                                                                       
    boolean isAttachedTo(LifecycleOwner owner) {
        // 判断是否为同个 LifecycleOwner
        return mOwner == owner;                                                     
    }                                                                                    
    @Override                                                                       
    void detachObserver() {
        // 1.removeObserver
        // 2. lifecycle state 没有在 started 之后，或者已经是 destoryed
        // 以上两种情况会调用 detachObserver
        // 从 Lifecycle 中移除 observer
        mOwner.getLifecycle().removeObserver(this);                                 
    }                                                                               
} 

private abstract class ObserverWrapper {                                             
    final Observer<? super T> mObserver;                                             
    boolean mActive;                                                                 
    int mLastVersion = START_VERSION;                                               
                                                                                     
    ObserverWrapper(Observer<? super T> observer) {                                 
        mObserver = observer;                                                       
    }                                                                               
                                                                                     
    abstract boolean shouldBeActive();                                               
                                                                                     
    boolean isAttachedTo(LifecycleOwner owner) {                                     
        return false;                                                               
    }                                                                                                                                                          
    void detachObserver() {}                                                         
                                                                                     
    void activeStateChanged(boolean newActive) {
        // 当活动状态发生变化时
        if (newActive == mActive) {                                                 
            return;                                                                 
        }                                                                            
        // immediately set active state, so we'd never dispatch anything to inactive 
        // owner                                                                     
        mActive = newActive;
        // 当前活动个数统计
        changeActiveCounter(mActive ? 1 : -1);
        
        if (mActive) { 
            // 当前为活动状态，分发当前 observer
            dispatchingValue(this);                                                 
        }                                                                           
    }                                                                               
}
```

当调用 `observe()` 会根据当前是否为活跃状态，分发当前值。

#### postValue 和 setValue

当在非主线程调用时，使用 `postValue()`，相关源码如下：

```java
protected void postValue(T value) {                                       
    boolean postTask;                                                     
    synchronized (mDataLock) {
        // 当前是否存在需要分发的值
        // 见下方代码***处
        postTask = mPendingData == NOT_SET;  
        // 多次调用，只会分发最后一次的值
        mPendingData = value;                                             
    }                                                                     
    if (!postTask) {
        // 已存在正在分发的任务
        return;                                                           
    }                                                                     
    ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);  
}
```

这里需要注意的是，当分发数据的任务还没结束时，多次调用 `postValue` 只会分发最后一次的值，同一时间只会存在一个 `PostValueRunnable`

```java
private final Runnable mPostValueRunnable = new Runnable() { 
    @Override                                                
    public void run() {                                      
        Object newValue;                                     
        synchronized (mDataLock) {                           
            newValue = mPendingData; 
            // 设置为 NOT_SET
            // *** 只有post出去的Runnable执行到此后，才可以再次post
            // 不然连续post只会更新mPendingData的值
            mPendingData = NOT_SET;
        }
        //noinspection unchecked 
        // 再调用 setValue
        setValue((T) newValue);                              
    }                                                        
}; 

@MainThread                                         
protected void setValue(T value) {
    // 只能在主线程调用
    assertMainThread("setValue"); 
    // 使用 version 记录
    mVersion++; 
    // 当前值
    mData = value;  
    // 分发
    dispatchingValue(null);                         
} 

void dispatchingValue(@Nullable ObserverWrapper initiator) {                         
    if (mDispatchingValue) {
        // 正在分发中（分发废除）
        mDispatchInvalidated = true;                                                 
        return; 
    }
    mDispatchingValue = true;
    do {         
        mDispatchInvalidated = false;
        if (initiator != null) {
            // 初始分发
            considerNotify(initiator);                                               
            initiator = null;                                                       
        } else {
            // 升序迭代，调用 considerNotify
            for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =  mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                considerNotify(iterator.next().getValue());                         
                if (mDispatchInvalidated) {
                    // 分发过程中，有新的数据需要分发
                    break;                                                           
                }
            }                                                                       
        }                                                                           
    } while (mDispatchInvalidated);//分发废除（无效）后，再次分发                      
    mDispatchingValue = false;                                                      
}
```

如果有阅读过 [Jetpack中的Lifecycle](https://linxiaotao.github.io/2019/01/14/Jetpack中的Lifecycle/) 这篇文章的话，可以很快看出，分发值的处理和 `Lifecycle` 有类似的地方，即在分发过程中，有新的值开始分发这种 case，都是采用中断当前的分发，推迟到下一次遍历时，分发新的值。

```
private void considerNotify(ObserverWrapper observer) {                             
    if (!observer.mActive) { 
        // 当前 observer 为非活动状态
        return;                                                                     
    }                                                                                 
    // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.  
    // we still first check observer.active to keep it as the entrance for events. So even if   
    // the observer moved to an active state, if we've not received that event, we better not   
    // notify for a more predictable notification order.
    // 可能存在已经是非活动状态，但没有接收到的 case
    // 检测是否为活动状态
    if (!observer.shouldBeActive()) {
        // 非活动状态，mActive 和 shouldBeActive 不一致，通知 activeStateChanged
        observer.activeStateChanged(false);                                         
        return;                                                                     
    }                                                                               
    if (observer.mLastVersion >= mVersion) { 
        // 当重新变为活动状态时，会分发当前值，避免重复值
        return;                                                                     
    }                                                                               
    observer.mLastVersion = mVersion;                                               
    //noinspection unchecked                                                         
    observer.mObserver.onChanged((T) mData);                                         
}
```

需要注意的是，`setValue` 和 `postValue` 方法都是 `protected`，所以当我们想实现一个 LiveData 时，需要使用 MutableLiveData，源码很简单，就是将这两个方法修改为 `public`。

> 这种实现思路有利于数据的不可变性。上游使用 MutableLiveData 分发数据，下游使用 LiveData 消费数据。

如果使用过 RxJava 类似库的同学可能会觉得跟 LiveData 很像，同样是监听者模式。不一样的是，RxJava 提供了很多操作符，类似 `map`、`zip` 等，还有线程调度的 `observeOn` 和 `subscribeOn`。LiveData 没有线程调度的功能，它数据分发只在主线程上进行，没有线程锁等开销。除此之前，LiveData 只提供了 `map` 和 `switchMap` 的实现，但提供了 `MediatorLiveData` 去实现自己需要的操作符。

#### Transformations（注：这个还没学，不太懂，需要再查资料）

Transformations 中有 `map` 和 `switchMap` 的实现。

```java
@MainThread  
public static <X, Y> LiveData<Y> map(                              
        @NonNull LiveData<X> source,                               
        @NonNull final Function<X, Y> mapFunction) {
    final MediatorLiveData<Y> result = new MediatorLiveData<>();   
    result.addSource(source, new Observer<X>() {                   
        @Override                                                  
        public void onChanged(@Nullable X x) {
            // 对源数据进行转化
            result.setValue(mapFunction.apply(x));                 
        }                                                          
    });                                                            
    return result;                                                 
} 

@MainThread  
public static <X, Y> LiveData<Y> switchMap(                            
        @NonNull LiveData<X> source,                                   
        @NonNull final Function<X, LiveData<Y>> switchMapFunction) {   
    final MediatorLiveData<Y> result = new MediatorLiveData<>();       
    result.addSource(source, new Observer<X>() {                       
        LiveData<Y> mSource;                                           
                                                                       
        @Override                                                      
        public void onChanged(@Nullable X x) {
            // 将发射出来的每个数据都转换成新的 LiveData，再接收
            LiveData<Y> newLiveData = switchMapFunction.apply(x);      
            if (mSource == newLiveData) {                              
                return;                                                
            }                                                          
            if (mSource != null) {                                     
                result.removeSource(mSource);                          
            }                                                          
            mSource = newLiveData;                                     
            if (mSource != null) {                                     
                result.addSource(mSource, new Observer<Y>() {          
                    @Override                                          
                    public void onChanged(@Nullable Y y) {             
                        result.setValue(y);                            
                    }                                                  
                });                                                    
            }                                                          
        }                                                              
    });                                                                
    return result;                                                     
}
```

MediatorLiveData 可以监听多个 LiveData，这里我们简单看下它的源码：

```java
@MainThread                                                                         
public <S> void addSource(@NonNull LiveData<S> source, @NonNull Observer<? super S> onChanged) {  
    Source<S> e = new Source<>(source, onChanged);                                   
    Source<?> existing = mSources.putIfAbsent(source, e);                           
    if (existing != null && existing.mObserver != onChanged) {                       
        throw new IllegalArgumentException(                                         
                "This source was already added with the different observer");       
    }                                                                               
    if (existing != null) {                                                         
        return;                                                                     
    }                                                                                             
    if (hasActiveObservers()) { 
        // 存在活动的 observer，开始监听 source livedata
        e.plug();                                                                   
    }                                                                              
}                                                                                   

@MainThread                                                     
public <S> void removeSource(@NonNull LiveData<S> toRemote) {   
    Source<?> source = mSources.remove(toRemote);               
    if (source != null) { 
        // 删除监听	
        source.unplug();                                        
    }                                                           
}                                                               

@CallSuper                                                         
@Override                                                          
protected void onActive() {                                        
    for (Map.Entry<LiveData<?>, Source<?>> source : mSources) { 
        // 重新监听
        source.getValue().plug();                                  
    }                                                              
}                                                                  
                                                                   
@CallSuper                                                         
@Override                                                          
protected void onInactive() {                                      
    for (Map.Entry<LiveData<?>, Source<?>> source : mSources) {
        // 暂时删除监听
        source.getValue().unplug();                                
    }                                                              
}                                                                  

private static class Source<V> implements Observer<V> {                    
    final LiveData<V> mLiveData;                                           
    final Observer<? super V> mObserver;                                   
    int mVersion = START_VERSION;                                          
                                                                           
    Source(LiveData<V> liveData, final Observer<? super V> observer) {     
        mLiveData = liveData;                                              
        mObserver = observer;                                              
    }                                                                      
                                                                           
    void plug() { 
        // 这里可以使用 observeForever，因为会在 onActive 和 onInactive 两个方法中处理，能响应页面的生命周期
        mLiveData.observeForever(this);                                    
    }                                                                      
                                                                           
    void unplug() {                                                        
        mLiveData.removeObserver(this);                                    
    }                                                                      
                                                                           
    @Override                                                              
    public void onChanged(@Nullable V v) {                                 
        if (mVersion != mLiveData.getVersion()) {                          
            mVersion = mLiveData.getVersion();                             
            mObserver.onChanged(v);                                        
        }                                                                  
    }                                                                      
}
```

### 小结

LiveData 可以算是 RxJava 的简化版本，它最大的好处就是和 Lifecycle 结合起来使用，不需要手动处理监听。第二个是，将数据分发操作封闭在主线程，减少线程同步锁的损耗，更适合 Android 日常开发场景。

