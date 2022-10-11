> 源码分析基于：2.3.0

有很多项目都会使用 MVP 这种项目架构，使用 Presenter 来减轻 `Activity` 的负担，具体的 `MVP` 实现可以阅读 Google 推出的 [android-architecture](https://github.com/googlesamples/android-architecture)。

一般使用 MVP，都会遇到一个问题，如何将 Presenter 和 View 的生命周期进行绑定，常见的做法是，在 `Activity` 的生命周期中手动调用 Presenter 的回调方法，更复杂的做法可能需要在 Presenter 或者 View 维护一个操作栈，在指定生命周期中去执行操作。

## Lifecycle

[lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle) 是 Google 推出的用于响应 `Activity` 和 `Fragment` 生命周期改变的库。

**These components help you produce better-organized, and often lighter-weight code, that is easier to maintain.**

#### Lifecycle

`Lifecycle` 是一个包含比如 `Activity` 或 `Fragment` 生命周期状态的类，同时允许其他对象去订阅这些状态

`Lifecycle` 使用 Event 和 State 来管理生命周期状态的变化

##### Event

生命周期变化的事件

##### State

当前生命周期的状态

这两者的关系如下：

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic3.zhimg.com%2Fv2-924029f666eabc8be6cc328fef6f5e26_r.jpg&refer=http%3A%2F%2Fpic3.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1619660996&t=8a00f3d8e0af2b6884777acbe7897d9a)

#### 1、lifecycle基本使用

我们可以调用 `addObserver` 去注册一个监听者

```java
getLifecycle().addObserver(new PersonOne());
```

而 `PersonOne` 则需要实现 `LifecycleObserver` 接口，具体的回调方法则通过注解的形式：

```java
public class PersonOne implements LifecycleObserver {
    private static final String TAG = "PersonOne";

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onStart() {
        Log.d(TAG, "onStart");
    }
}
```

我们也可以调用 `getCurrentState()` 来获取 `Lifecycle` 当前的状态

#### 2、LifecycleOwner

上面我们简单介绍了 `Lifecyle` 和 `LifecycleObserver` 的作用和关系之后，再来看下 `getLifecycle` 这个方法：

```java
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    @NonNull
    Lifecycle getLifecycle();
}
```

简单来说，`LifecycleOwner` 用于提供 `Lifecycle`，`LifecycleObserver` 监听 `Lifecycle` 的状态变化，`Lifecycle` 则是 `Activity` 或 `Fragment` 生命周期状态的抽象。

可以看到 `Fragment` 和 `ComponentActivity` 等实现了 `LifecycleOwner` 接口，这里我们看下 `ComponentActivity` 的实现：

##### 2.1 ComponentActivity

```java
private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

@CallSuper                                                   
@Override                                                    
protected void onSaveInstanceState(Bundle outState) {
    // 这里需要注意，onSaveInstanceState 状态设置为 CREATED
    mLifecycleRegistry.markState(Lifecycle.State.CREATED);   
    super.onSaveInstanceState(outState);                     
}                                                            

@Override                         
public Lifecycle getLifecycle() { 
    return mLifecycleRegistry;    
}

@Override                                                        
@SuppressWarnings("RestrictedApi")                               
protected void onCreate(@Nullable Bundle savedInstanceState) {   
    super.onCreate(savedInstanceState);     
    // 2.2
    ReportFragment.injectIfNeededIn(this);                       
}
```

可以看到代码非常简洁，那它又是怎么去实现的呢？可以看到在 `onCreate` 中会调用 `ReportFragment.injectIfNeededIn`

##### 2.2 ReportFragment#injectIfNeededIn()

通过此方法，向Activity中注入（添加）了一个ReportFragment。

```java
    public static void injectIfNeededIn(Activity activity) {
        if (Build.VERSION.SDK_INT >= 29) {
            // On API 29+, we can register for the correct Lifecycle callbacks directly
            LifecycleCallbacks.registerIn(activity);
        }
        // Prior to API 29 and to maintain compatibility with older versions of
        // ProcessLifecycleOwner (which may not be updated when lifecycle-runtime is updated and
        // need to support activities that don't extend from FragmentActivity from support lib),
        // use a framework fragment to get the correct timing of Lifecycle events
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }
```

这里会添加一个 `ReportFragment` 如果有阅读过 Glide 源码的同学，应该会看到类似的实现：通过添加一个透明的 `Fragment` 来监听 `Activity` 的生命周期。

##### 2.3 ReportFragment

```java
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }

    @Override
    public void onPause() {
        super.onPause();
        dispatch(Lifecycle.Event.ON_PAUSE);
    }

    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
        // just want to be sure that we won't leak reference to an activity
        mProcessListener = null;
    }
    
    private void dispatch(@NonNull Lifecycle.Event event) {
        if (Build.VERSION.SDK_INT < 29) {
            // Only dispatch events from ReportFragment on API levels prior
            // to API 29. On API 29+, this is handled by the ActivityLifecycleCallbacks
            // added in ReportFragment.injectIfNeededIn
            dispatch(getActivity(), event);
        }
    }
    
    static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
        if (activity instanceof LifecycleRegistryOwner) {
            // 3
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }

        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }    
```

在 `Activity` 的各种生命周期回调方法中，调用 `handleLifecycleEvent()` 分发 `Lifecycle.Event`

##### 3、handleLifecycleEvent

`LifecycleRegistry` 是 `Lifecycle` 的实现类，看下 `handleLifecycleEvent` 的实现：

```java
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {    
	enforceMainThreadIfNeeded("handleLifecycleEvent");                           
    moveToState(event.getTargetState());                     
}              

private void moveToState(State next) {                                
    if (mState == next) {                                             
        return;                                                       
    }                                                                 
    mState = next;                                                    
    if (mHandlingEvent || mAddingObserverCounter != 0) {
        // 正在处理事件中或者正在处理添加 Observer 中
        mNewEventOccurred = true;                                     
        // we will figure out what to do on upper level.              
        return;                                                       
    }
    // 标记正在处理事件
    mHandlingEvent = true;
    // 同步状态
    sync();                                                           
    mHandlingEvent = false;                                           
}

// happens only on the top of stack (never in reentrance),                       
// so it doesn't have to take in account parents                                 
private void sync() {
    // 使用弱应用持有 LifecycleOwner，也是为了防止 Activity/Fragment 内存泄漏
    LifecycleOwner lifecycleOwner = mLifecycleOwner.get();                       
    
    if (lifecycleOwner == null) {
        throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already" + "garbage collected. It is too late to change lifecycle state.");   
        return;                                                                  
    }                                                                                         
    while (!isSynced()) {
        // 如果还没完成同步
        mNewEventOccurred = false;                                               
        // no need to check eldest for nullability, because isSynced does it for us.
        // 使用 eldest(start) 判断是否需要回退 
        // 使用 newest(end) 判断是否需要前进，刚添加的 observer 一般为初始化状态
        if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
            // 最先添加的 observer 的状态大于当前状态，回退
            // 3.1
            backwardPass(lifecycleOwner);                                       
        }                                                                                     
        Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();           
        if (!mNewEventOccurred && newest != null                                 
                && mState.compareTo(newest.getValue().mState) > 0) {
            // 最新添加的 observer 如果状态一致，则可以乐观地表示在它之前添加的 observer 状态也是一致的
            // mNewEventOccurred 表示有新的事件发生，则放弃这次同步，延迟到下一次
            // 3.2
            forwardPass(lifecycleOwner);                                         
        }                                                                         
    }                                                                             
    mNewEventOccurred = false;                                                   
}

private boolean isSynced() {                                                     
    if (mObserverMap.size() == 0) {                                               
        return true;                                                             
    }
    // eldest 最先添加的，newest 最新添加的
    State eldestObserverState = mObserverMap.eldest().getValue().mState;         
    State newestObserverState = mObserverMap.newest().getValue().mState;
    // 判断状态是否一致
    return eldestObserverState == newestObserverState && mState == newestObserverState; 
}
```

使用 `sync()` 同步状态，这里分为两种情况，一种是需要回退状态（backward），另外一种则是需要前进（forward），其中 `backward` 代码如下：

##### 3.1 backward

```java
private void backwardPass(LifecycleOwner lifecycleOwner) {
    // 使用 eldest(start) 判断是否需要回退 
    // 降序迭代，end -> start
    Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =  
            mObserverMap.descendingIterator();                                   
    while (descendingIterator.hasNext() && !mNewEventOccurred) {
        // mNewEventOccurred 判断是否有新事件分发
        Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next(); 
        ObserverWithState observer = entry.getValue();                           
        while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred       
                && mObserverMap.contains(entry.getKey()))) {
            // 回退
            // 举个例子：observer.state 为 RESUMED
            // mState 为 CREATED
            // downEvent(observer.state) 为 ON_PAUSE
            Event event = Event.downFrom(observer.mState);
            // getStateAfter(event) 为 STARTED
            // 最终：RESUMED -> STARTED，在下一次同步中再同步为 CREATED
            // pushParentState 和 popParentState 则是将 state 暂存在 List 中，这个作用我们会在 addObserver 中讲
            pushParentState(getStateAfter(event));
            // 分发事件
            observer.dispatchEvent(lifecycleOwner, event);                       
            popParentState();                                                     
        }                                                                         
    }                                                                             
}
	// Event#downFrom
    public static Event downFrom(@NonNull State state) {
        switch (state) {
            case CREATED:
                return ON_DESTROY;
            case STARTED:
                return ON_STOP;
            case RESUMED:
                return ON_PAUSE;
            default:
                return null;
        }
    }

	// ObserverWithState.java
    void dispatchEvent(LifecycleOwner owner, Event event) {
        State newState = event.getTargetState();
        mState = min(mState, newState);
        mLifecycleObserver.onStateChanged(owner, event);
        mState = newState;
    }
```

总结下 `backwardPass()` 的逻辑：将较大的状态逐步回退。为什么说是逐步呢？比如 `observer.state` 是 `RESUMED`，当前状态是 `CREATED`，那这里会分两次回退，分别为：`RESUMED -> STARTED` 和 `STARTED -> CREATED`

`forwardPass()` 逻辑类似，则不分析了。

#### 4、addObserver

`addObserver()` 是添加 `Observer` 的方法，源码如下：

```java
@Override                                                                         
public void addObserver(@NonNull LifecycleObserver observer) {
	enforceMainThreadIfNeeded("addObserver");
    // 如果不是 DESTROYED，则从 INITIALIZED 开始分发
    State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
    // ObserverWithState 用于分发事件给 observer
    ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);   
    ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);                                                                                
    if (previous != null) { 
        // 唯一性
        return;                                                                   
    }                                                                                     
    LifecycleOwner lifecycleOwner = mLifecycleOwner.get();                       
    if (lifecycleOwner == null) {  
        // mLifecycleOwner 为弱引用
        // it is null we should be destroyed. Fallback quickly                   
        return;                                                                   
    }                                                                           
    
    // isReentrance 表示是否在分发事件时新添加了 observer
    // 举个例子：在 observer 在 onStart() 中又调用了 addObserver()
    boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent; 
    // 计算需要分发的状态
    // 4.1
    State targetState = calculateTargetState(observer);                           
    mAddingObserverCounter++; 
    // 将事件逐步分发到 targetState
    while ((statefulObserver.mState.compareTo(targetState) < 0                   
            && mObserverMap.contains(observer))) {
        // 如果 statefulObserver.state 小于 targetState
        pushParentState(statefulObserver.mState);
        // 如果 state 为 STARTED，则 upFrom(state) 则为 ON_RESUME
        final Event event = Event.upFrom(statefulObserver.mState);
        if (event == null) {
            throw new IllegalStateException("no event up from " + statefulObserver.mState);
        }        
        statefulObserver.dispatchEvent(lifecycleOwner, event); 
        popParentState();                                                         
        // mState / subling may have been changed recalculate                     
        targetState = calculateTargetState(observer);                             
    }                                                                                 
    if (!isReentrance) {                                                         
        // we do sync only on the top level.
        // 当前为重入，则不进行同步
        sync();                                                                   
    }                                                                             
    mAddingObserverCounter--;                                                     
}

public static Event upFrom(@NonNull State state) {
    switch (state) {
        case INITIALIZED:
            return ON_CREATE;
        case CREATED:
            return ON_START;
        case STARTED:
            return ON_RESUME;
        default:
            return null;
    }
}                                                                   
```

##### 4.1 calculateTargetState

```java
private State calculateTargetState(LifecycleObserver observer) {
    // 获取上一个添加的 observer
    Entry<LifecycleObserver, ObserverWithState> previous = mObserverMap.ceil(observer);       
    State siblingState = previous != null ? previous.getValue().mState : null;
    // mParentStates 是个 List，它的添加和删除分别由 pushParentState() 和 popParentState()，它们是成对出现的，在 dispatchEvent 的前后
    // 在这种 case 下，会存在 parentState：在 dispatchEvent 时，又调用了 addObserver()，即上面说的 isReentrance
    State parentState = !mParentStates.isEmpty() ? mParentStates.get(mParentStates.size() - 1)
            : null;
    // 这里的计算是获取更合适的状态
    // 考虑以下这种 case：某个 observer 在 onStart() 中再调用 addObserver，那这个 observer 理应使用 STARTED 状态分发，而当前状态即 mState 可能是 RESUMED，再在 sync() 中进行同步
    return min(min(mState, siblingState), parentState);                           
}
```

`addObserver()` 主要考虑了 Reentrance 的情况，即在 `observer` 的事件分发中，又添加了新的 `observer` 的情况。

#### 5、ProcessLifecycleOwner

`ProcessLifecycleOwner` 提供应用进程的生命周期。跟 `Activity` 和 `Fragment` 的生命周期不一样的是：

- `ON_CREATE` 只会分发一次
- `ON_DESTROY` 则不会被分发
- `ON_START` 和 `ON_RESUME` 在第一个 `Activity` 的时候分发
- `ON_PAUSE` 和 `ON_STOP` 则在最后一个 `Activity` 的时候**延迟**分发，用于防止因为配置改变，而导致 `Activity` 重建

下面我们来分析下 `ProcessLifecycleOwner` 的源码，首先看下 `init()`：

```java
// 单例
private static final ProcessLifecycleOwner sInstance = new ProcessLifecycleOwner();

static void init(Context context) {
    // 调用 attach
    sInstance.attach(context);      
}                                   

void attach(Context context) {                                                   
    mHandler = new Handler();
    // 同样是使用 LifecycleRegistry 来处理
    // 先分发 ON_CREATE，而且只会分发一次
    mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);                   
    Application app = (Application) context.getApplicationContext();
    app.registerActivityLifecycleCallbacks(new EmptyActivityLifecycleCallbacks() {      
        public void onActivityPreCreated(@NonNull Activity activity,
                @Nullable Bundle savedInstanceState) {
            activity.registerActivityLifecycleCallbacks(new EmptyActivityLifecycleCallbacks() {
                @Override
                public void onActivityPostStarted(@NonNull Activity activity) {
                    activityStarted();
                }
                @Override
                public void onActivityPostResumed(@NonNull Activity activity) {
                    activityResumed();
                }
            });
        }
        
        @Override                                                                 
        public void onActivityCreated(Activity activity, Bundle savedInstanceState) {   
 			if (Build.VERSION.SDK_INT < 29) {
                ReportFragment.get(activity).setProcessListener(mInitializationListener);
            } 
        }                                                                               
                                                                                        
        @Override                                                                       
        public void onActivityPaused(Activity activity) {                               
            activityPaused();                                                           
        }                                                                               
                                                                                        
        @Override                                                                       
        public void onActivityStopped(Activity activity) {                              
            activityStopped();                                                          
        }                                                                               
    });                                                                                 
}

ActivityInitializationListener mInitializationListener =      
        new ActivityInitializationListener() {                
            @Override                                         
            public void onCreate() {                          
            }                                                 
                                                              
            @Override                                         
            public void onStart() {                           
                activityStarted();                            
            }                                                 
                                                              
            @Override                                         
            public void onResume() {                          
                activityResumed();                            
            }                                                 
        };
                                
// ground truth counters                         
private int mStartedCounter = 0;                 
private int mResumedCounter = 0;                 
                                                 
private boolean mPauseSent = true;               
private boolean mStopSent = true;                

void activityStarted() {
    // 计数
    mStartedCounter++;                                            
    if (mStartedCounter == 1 && mStopSent) {
        // 第一次调用，分发 ON_START
        // mStopSent 为 true 的情况：
        // 1. 默认为 true
        // 2. dispatchStopIfNeeded 中设置
        // 防止因为配置改变，Activity 创建而导致重新分发
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START); 
        mStopSent = false;                                        
    }                                                             
}

 void activityResumed() {                                               
     mResumedCounter++;                                                 
     if (mResumedCounter == 1) {                                        
         if (mPauseSent) {
             // 第一次调用分发 ON_RESUME
             mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME); 
             mPauseSent = false;                                        
         } else { 
             // 配置改变，Activity 创建，删除延迟的 pause runnable
             mHandler.removeCallbacks(mDelayedPauseRunnable);           
         }                                                              
     }                                                                  
 }
```

上面是初始事件分发流程，下面我们来看下 `ON_PAUSE` 和 `ON_STOP` 的分发：

```java
@VisibleForTesting                               
static final long TIMEOUT_MS = 700; //mls 

// 在 onActivityPaused 中调用
void activityPaused() {
    // 计数
    mResumedCounter--;                                              
    if (mResumedCounter == 0) {
        // 延迟分发
        mHandler.postDelayed(mDelayedPauseRunnable, TIMEOUT_MS);    
    }                                                               
}
 
private Runnable mDelayedPauseRunnable = new Runnable() { 
    @Override                                             
    public void run() { 
        // 延迟分发
        dispatchPauseIfNeeded();                          
        dispatchStopIfNeeded();                           
    }                                                     
};                                                        

void activityStopped() { 
    // 计数
    mStartedCounter--;                                              
    dispatchStopIfNeeded();                                         
}                                                                   
                                                                    
void dispatchPauseIfNeeded() {
    if (mResumedCounter == 0) {
        // 计数
        mPauseSent = true;
        // 分发 ON_PAUSE
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);   
    }                                                               
}                                                                   
                                                                    
void dispatchStopIfNeeded() {                                       
    if (mStartedCounter == 0 && mPauseSent) {
        // 计数为 0，同时已经分发了 ON_PAUSE
        // 分发 ON_STOP
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);    
        mStopSent = true;                                           
    }                                                               
}
```

到这里，我们已经将 `ProcessLifecycleOwner` 的流程分析完了，那 `ProcessLifecycleOwner.init()` 是在那里调用的呢？

其实是用了一种比较巧妙的方法，在 `lifecycle-process` 这个包下的 `AndroidManifest.xml` 文件中，可以看到有如下配置：

```xml
<application>
        <provider
            android:name="androidx.lifecycle.ProcessLifecycleOwnerInitializer"
            android:authorities="${applicationId}.lifecycle-process"
            android:exported="false"
            android:multiprocess="true" />
</application>
```

其中 `ProcessLifecycleOwnerInitializer` 是一个 `ContentProvider` 即利用 `ContentProvider` 来实现自动初始化

> ContentProvider 的 `onCreate` 方法会在应用启动时候调用
>
> Implement this to initialize your content provider on startup. This method is called for all registered content providers on the application main thread at application launch time. It must not perform lengthy operations, or application startup will be delayed.

```java
@Override                                     
public boolean onCreate() {                   
    LifecycleDispatcher.init(getContext());   
    ProcessLifecycleOwner.init(getContext()); 
    return true;                              
}
```

可以看到这里调用了两个初始化方法，其中 `ProcessLifecycleOwner.init()` 我们已经分析了，再看看 `LifecycleDispatcher.init()`

```java
static void init(Context context) {                                                 
    if (sInitialized.getAndSet(true)) {                                             
        return;                                                                     
    }                                                                               
    ((Application) context.getApplicationContext())                                 
            .registerActivityLifecycleCallbacks(new DispatcherActivityCallback());  
}                                                                                   
                                                                                    
@SuppressWarnings("WeakerAccess")                                                   
@VisibleForTesting                                                                  
static class DispatcherActivityCallback extends EmptyActivityLifecycleCallbacks {   
                                                                                    
    @Override                                                                       
    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {   
        ReportFragment.injectIfNeededIn(activity);                                  
    }                                                                               
                                                                                    
    @Override                                                                       
    public void onActivityStopped(Activity activity) {                              
    }                                                                               
                                                                                    
    @Override                                                                       
    public void onActivitySaveInstanceState(Activity activity, Bundle outState) {   
    }                                                                               
}
```

初始化的逻辑比较简单，即在 `Activity` 创建时，调用 `ReportFragment.injectIfneededIn()` ，其实我们在 `Activity` 的 `Lifecycle` 处理中，也见到这个方法：

```
// ComponentActivity.java
@Override                                                        
@SuppressWarnings("RestrictedApi")                               
protected void onCreate(@Nullable Bundle savedInstanceState) {   
    super.onCreate(savedInstanceState);                          
    ReportFragment.injectIfNeededIn(this);                       
}

// ReportFragment.java
public static void injectIfNeededIn(Activity activity) {                                 
    // ProcessLifecycleOwner should always correctly work and some activities may not extend  
    // FragmentActivity from support lib, so we use framework fragments for activities   
    android.app.FragmentManager manager = activity.getFragmentManager();                 
    if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
        // 重复添加判断
        manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();   
        // Hopefully, we are the first to make a transaction.                             
        manager.executePendingTransactions();                                             
    }                                                                                     
}
```

这里可以理解为双重保障吧，可能存在不继承 `ComponentActivity` 的 `Activity`。

### 小结

分析了 `Lifecycle` 的整个流程，可以发现其实逻辑还是比较简单的，实现上也是参考了其他开源库的做法，比如 `ReportFragment` 通过添加一个透明的 `Fragment` 去感知 `Activity` 的生命周期，`Glide` 也是这么做的。还有使用 `ContentProvider` 去实现在 `Application` 创建时自动初始化，也是一个不错的想法。

`Lifecycle` 的源码比我一开始想象的复杂，不是在于它的逻辑，而是在 `sync()` 这一块，通过 `Listener` 执行状态回调是一个非常常见的做法，但是有很多需要考虑的 case，举个例子，在分发回调时，有新的状态发生，那么应该怎么去处理。或者，在回调方法中，又添加了新的 `Listener`，那应该怎么处理。

学习源码，不仅仅是学习实现原理，还可以学习一个健壮的库是如何处理各种场景下的 case 的。




