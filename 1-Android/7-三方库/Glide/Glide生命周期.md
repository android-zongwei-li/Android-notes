# 前言

- 正由于他的功能强大，所以它的源码非常复杂，这导致很多人望而却步，本人尝试将 `Glide` 的功能进行分解，并单独针对每个功能进行源码分析，从而降低`Glide`源码的复杂度。

> 接下来，我将推出一系列关于 `Glide`的功能源码分析，有兴趣可以继续关注

- 今天，我将主要针对**`Glide`的图片缓存功能** 进行流程 & 源码分析 ，希望你们会喜欢。

------

# 储备知识

阅读本文前，请务必先阅读Glide的图片加载功能源码分析：[Carson带你学Android：图片加载库Glide源码分析](https://www.jianshu.com/p/c3a5518b58b2)

------

# 源码分析入口：with()

该方法在图片加载库Glide加载图片的源码分析中曾进行详细说明，具体请看：[Carson带你学Android：图片加载库Glide源码分析](https://www.jianshu.com/p/c3a5518b58b2)

- 定义：`Glide` 类中的静态方法，根据传入 不同参数 进行 方法重载
- 作用：得到一个`RequestManager`对象后，根据传入`with()`方法的参数 **将Glide图片加载的生命周期与Activity/Fragment的生命周期进行绑定，从而实现自动执行请求，暂停操作**
- 具体源码分析



```java
public class Glide {
    ...

    // with()重载种类非常多，根据传入的参数可分为：
    // 1. 非Application类型的参数（Activity & Fragment  ）
    // 2. Application类型的参数（Context）
    // 下面将详细分析

// 参数1：Application类型
 public static RequestManager with(Context context) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        // 步骤1：调用RequestManagerRetriever类的静态get()获得RequestManagerRetriever对象 - 单例实现
        return retriever.get(context);
        // 步骤2：调用RequestManagerRetriever实例的get()获取RequestManager对象 & 绑定图片加载的生命周期 ->>分析1
    }

// 参数2：非Application类型（Activity & Fragment ）
    public static RequestManager with(Activity activity) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(activity);
    }

    public static RequestManager with(Fragment fragment) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(fragment);

  @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public static RequestManager with(android.app.Fragment fragment) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(fragment);
    }

    }
}

<-- 分析1：RequestManagerRetriever对象的实例 get（）-->
// 作用：
  // 1. 获取RequestManager对象
  // 2. 将图片加载的生命周期与Activity/Fragment的生命周期进行绑定

  public class RequestManagerRetriever implements Handler.Callback {
      ...

    // 实例的get（）重载种类很多，参数分为：（与with（）类似）
    // 1. Application类型（Context）
    // 2. 非Application类型（Activity & Fragment ）- >>分析3
    // 下面会详细分析

// 参数1：Application类型（Context） 
   public RequestManager get(Context context) {
        return getApplicationManager(context);
        // 调用getApplicationManager（）最终获取一个RequestManager对象 ->>分析2
        // 因为Application对象的生命周期即App的生命周期
        // 所以Glide加载图片的生命周期是自动与应用程序的生命周期绑定，不需要做特殊处理（若应用程序关闭，Glide的加载也会终止）
    }

// 参数2：非Application类型（Activity & Fragment  ）
// 将Glide加载图片的生命周期与Activity生命周期同步的具体做法：向当前的Activity添加一个隐藏的Fragment
// 原因：因Fragment的生命周期 与 Activity 的是同步的，通过添加隐藏的Fragment 从而监听Activity的生命周期，从而实现Glide加载图片的生命周期与Activity的生命周期 进行同步。
 @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public RequestManager get(Activity activity) {
        if (Util.isOnBackgroundThread() || Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
            return get(activity.getApplicationContext());
        } else {

            assertNotDestroyed(activity);
             //判断activity是否已经销毁

            android.app.FragmentManager fm = activity.getFragmentManager();
            // 获取FragmentManager 对象
            
            return fragmentGet(activity, fm);
           // 通过fragmentGet返回RequestManager。
           // 1. 创建Fragment
           // 2. 向当前的Activity中添加一个隐藏的Fragment
           // 3. 将RequestManager与该隐藏的Fragment进行绑定
        }
    }

    public RequestManager get(FragmentActivity activity) {
      // 逻辑同上，此处不作过多描述
      ...

    }

    public RequestManager get(Fragment fragment) {
        // 逻辑同上，此处不作过多描述
      ...
    }

}

<-- 分析2：getApplicationManager（context）-->
private RequestManager getApplicationManager(Context context) {

      ...

        Glide glide = Glide.get(context);
        // 通过单例模式创建Glide实例 ->>分析3
        applicationManager =
            new RequestManager(
                glide, new ApplicationLifecycle(), new EmptyRequestManagerTreeNode());
      }
    }
  }
  return applicationManager;
}

<-- 分析3：Glide.get(context) -->
public static Glide get(Context context) {
  if (glide == null) {
  // 单例模式的体现
    synchronized (Glide.class) {
      if (glide == null) {
        Context applicationContext = context.getApplicationContext();
        
        List<GlideModule> modules = new ManifestParser(applicationContext).parse();
        // 解析清单文件配置的自定义GlideModule的metadata标签，返回一个GlideModule集合

        GlideBuilder builder = new GlideBuilder(applicationContext);
        // 步骤1：创建GlideBuilder对象
        for (GlideModule module : modules) {
          module.applyOptions(applicationContext, builder);
        }
        glide = builder.createGlide();
        // 步骤2：根据GlideBuilder对象创建Glide实例
        // GlideBuilder会为Glide设置一默认配置，如：Engine，RequestOptions，GlideExecutor，MemorySizeCalculator
       
        for (GlideModule module : modules) {
          module.registerComponents(applicationContext, glide.registry);
           // 步骤3：利用GlideModule 进行延迟性的配置和ModelLoaders的注册
        }
      }
    }
  }
  return glide;
}
// 回到分析2原处


<--分析4： -->
 RequestManager fragmentGet(Context context, android.app.FragmentManager fm) {
     
        RequestManagerFragment current = getRequestManagerFragment(fm);
        // 获取RequestManagerFragment
        // 作用：利用Fragment进行请求的生命周期管理 -->分析5
        RequestManager requestManager = current.getRequestManager();

        // 若requestManager 为空，即首次加载初始化requestManager 
        if (requestManager == null) {
            // 从分析6回来时请看这里
            // 创建RequestManager传入Lifecycle实现类，如ActivityFragmentLifecycle ->>分析7
            requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
            current.setRequestManager(requestManager);
            // 调用setRequestManager设置到RequestManagerFragment 
        }
        return requestManager;
    }

<--分析5 -->
 RequestManagerFragment getRequestManagerFragment(final android.app.FragmentManager fm) {

                current = new RequestManagerFragment();
                // 获取RequestManagerFragment 对象 -->分析6
                pendingRequestManagerFragments.put(fm, current);
              
            }
        }
        return current;
    }

<--分析6 -->
public class RequestManagerFragment extends Fragment {
// 继承Fragment
// 在其生命周期onStart()、onStop()、onDestory()，分别调用了ActivityFragmentLifecycle 相应的方法
// ActivityFragmentLifecycle实现了Lifecycle 接口,通过addListener（LifecycleListener listener）回调相应（LifecycleListener的 onStart(),onStop(),onDestory()）周期方法


    private final ActivityFragmentLifecycle lifecycle;
  
    @Override
    public void onStart() {
        super.onStart();
        //关联lifecycle相应onStart方法
        lifecycle.onStart();
    }

    @Override
    public void onStop() {
        super.onStop();
         //关联lifecycle相应onStop方法
        lifecycle.onStop();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
         //关联lifecycle相应onDestroy方法
        lifecycle.onDestroy();
    }
}

// 请回到分析4

<-- 分析7 -->

// 作用：实现了LifeCycleListener接口，绑定Activity/Fragment生命周期，对请求进行暂停，恢复，清除操作



public class RequestManager implements LifecycleListener {

    private final Lifecycle lifecycle;

// 将刚创建Ffragment的lifeCycle传入，并将RequestManager这个listener添加到lifeCycle中，从而实现绑定。

 public RequestManager(Context context, Lifecycle lifecycle, RequestManagerTreeNode treeNode) {
        this(context, lifecycle, treeNode, new RequestTracker(), new ConnectivityMonitorFactory());
    }

    RequestManager(Context context, final Lifecycle lifecycle, RequestManagerTreeNode treeNode,
            RequestTracker requestTracker, ConnectivityMonitorFactory factory) {
        this.context = context.getApplicationContext();
        this.lifecycle = lifecycle;
        this.treeNode = treeNode;

        this.requestTracker = requestTracker;
        // 跟踪请求取消，重启，完成，失败
        
        this.glide = Glide.get(context);
        // 通过Glide的静态方法获取Glide实例（单例模式）
        this.optionsApplier = new OptionsApplier();

//通过工厂类ConnectivityMonitorFactory的build方法获取ConnectivityMonitor （一个用于监控网络连接事件的接口）
        ConnectivityMonitor connectivityMonitor = factory.build(context,
                new RequestManagerConnectivityListener(requestTracker));

        // If we're the application level request manager, we may be created on a background thread. In that case we
        // cannot risk synchronously pausing or resuming requests, so we hack around the issue by delaying adding
        // ourselves as a lifecycle listener by posting to the main thread. This should be entirely safe.
        if (Util.isOnBackgroundThread()) {
            new Handler(Looper.getMainLooper()).post(new Runnable() {
                @Override
                public void run() {
                    lifecycle.addListener(RequestManager.this);
                }
            });
        } else {
        //设置监听
            lifecycle.addListener(this);
        }
        lifecycle.addListener(connectivityMonitor);
    }
    /**
     * Lifecycle callback that registers for connectivity events (if the android.permission.ACCESS_NETWORK_STATE
     * permission is present) and restarts failed or paused requests.
     */
    @Override
    public void onStart() {
        // onStart might not be called because this object may be created after the fragment/activity's onStart method.
        resumeRequests();
    }

    /**
     * Lifecycle callback that unregisters for connectivity events (if the android.permission.ACCESS_NETWORK_STATE
     * permission is present) and pauses in progress loads.
     */
    @Override
    public void onStop() {
        pauseRequests();
    }

    /**
     * Lifecycle callback that cancels all in progress requests and clears and recycles resources for all completed
     * requests.
     */
    @Override
    public void onDestroy() {
        requestTracker.clearRequests();
    }
}
```

------

# 总结

- Gilde的生命周期管理在于：with()中，根据传入with()方法的参数 将Glide图片加载的生命周期与Activity/Fragment的生命周期进行绑定，从而实现自动执行请求，暂停操作；
- RequestManagerFragment：连接生命周期方法；
- RequestManager：该类实现了LifeCycleListener接口，绑定Activity/Fragment生命周期，实现生命周期中请求方法，对请求进行暂停，恢复，清除操作；
- RequestManagerRetriever：将RequestManager和自定义Fragment（如RequestManagerFragment）绑定，从而实现在生命周期管理回调。



# 参考

[Android：深入了解图片加载库Glide的生命周期管理(源码分析）](https://www.jianshu.com/p/349d0e20245f)