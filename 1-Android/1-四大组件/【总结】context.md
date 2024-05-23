> version：2022/07/
>
> review：



# 通过这篇文章你将学习到：

- Context 与 Activity 有什么关系？
- Context对于Activity有什么作用？
- 不同类型的Context的应用场景是什么？
- 关于Context的几个常见关键知识点

# 1. 二者有什么关系？

Context 是一个抽象类；Activity 继承了 Context。具体继承关系如下：

- Context的直接实现子类：ContextImp、ContextWrapper

- ContextWrapper的子类是常用的Activity、Service、Application

  ![img](https:////upload-images.jianshu.io/upload_images/944365-6fce9c4817e1e474.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

------

# 2. Context 对于Activity有什么作用

Activity 通过 Context接口 去访问Android系统的服务 & 资源，主要包括：

1. 获取应用相关信息
2. 获取系统/应用资源
3. 四大组件之间的交互
4. 文件相关
5. 数据库相关

具体如下：



```java
public abstract class Context {
    
    // 作用1：获取应用相关信息
    public abstract ApplicationInfo getApplicationInfo();
    public abstract String getPackageName();
    public abstract Looper getMainLooper();
    public abstract int checkPermission(@NonNull String permission, int pid, int uid);

    // 作用2：获取系统/应用资源
    // 如 AssetManager、PackageManager、Resources、System Service 以及 color、string、drawable 等
    public abstract AssetManager getAssets();
    public abstract Resources getResources();
    public abstract PackageManager getPackageManager();
    public abstract Context getApplicationContext();
    public abstract ClassLoader getClassLoader();
    public final @Nullable <T> T getSystemService(@NonNull Class<T> serviceClass) { ... }

    public final String getString(@StringRes int resId) { ... }
    public final int getColor(@ColorRes int id) { ... }
    public final Drawable getDrawable(@DrawableRes int id) { ... }
    public abstract Resources.Theme getTheme();
    public abstract void setTheme(@StyleRes int resid);
    public final TypedArray obtainStyledAttributes(@StyleableRes int[] attrs) { ... }

    // 作用3：四大组件之间的交互
    // 如启动 Activity、Broadcast、Service，获取 ContentResolver 等
    public abstract void startActivity(@RequiresPermission Intent intent);
    public abstract void sendBroadcast(@RequiresPermission Intent intent);
    public abstract Intent registerReceiver(@Nullable BroadcastReceiver receiver,
                                            IntentFilter filter);
    public abstract void unregisterReceiver(BroadcastReceiver receiver);
    public abstract ComponentName startService(Intent service);
    public abstract boolean stopService(Intent service);
    public abstract boolean bindService(@RequiresPermission Intent service,
            @NonNull ServiceConnection conn, @BindServiceFlags int flags);
    public abstract void unbindService(@NonNull ServiceConnection conn);
    public abstract ContentResolver getContentResolver();
    
    // 作用4：文件相关
    // 如：获取缓存文件夹、删除文件、SharedPreference 相关等
    public abstract File getSharedPreferencesPath(String name);
    public abstract File getDataDir();
    public abstract boolean deleteFile(String name);
    public abstract File getExternalFilesDir(@Nullable String type);
    public abstract File getCacheDir();
    ...
    public abstract SharedPreferences getSharedPreferences(String name, @PreferencesMode int mode);
    public abstract boolean deleteSharedPreferences(String name);

    // 作用5：数据库
    // 如打开数据库、删除数据库、获取数据库路径等
    public abstract SQLiteDatabase openOrCreateDatabase(...);
    public abstract boolean deleteDatabase(String name);
    public abstract File getDatabasePath(String name);
   
    ...
}
```

------

# 3. 不同类型的Context的应用场景是什么？

从上面可知，最终的Context类型主要包括：Activity、Service & Application，那么使用这三者的应用场景区别是什么呢

##### case1：与UI相关的场景，都使用Activity类型Context

因为是针对当前UI界面资源进行操作 & 继承自ContextThemeWrapper（可自定义主题样式），如show a dialog、 Layout Inflation等。

##### case2：生命周期较长的对象，都使用Application类型Context

因为Application Context的生命周期与应用保持一致，可避免出现Context引用的内存泄漏

##### 其余场景，三种类型 基本可视为共用。

------

# 4. 关于Context的几个常见关键知识点

### 4.1 为什么Activity通过ContextThemeWrapper间接继承ContextWrapper，而Service跟Application直接继承ContextWrapper呢？

答：ContextThemeWrapper类包含了与主题Theme相关的接口，而只有Activity才需要主题，Service跟Application不需要。以下是ContextThemeWrapper的源码



```java
public class ContextThemeWrapper extends ContextWrapper {
    private int mThemeResource;
    private Resources.Theme mTheme;
    private LayoutInflater mInflater;
    private Configuration mOverrideConfiguration;
    private Resources mResources;

    public ContextThemeWrapper() {
        super(null);
    }

    public ContextThemeWrapper(Context base, @StyleRes int themeResId) {
        super(base);
        mThemeResource = themeResId;
    }

    public ContextThemeWrapper(Context base, Resources.Theme theme) {
        super(base);
        mTheme = theme;
    }
    
   // 包含了大量跟主题相关的接口方法
    @Override
    protected void attachBaseContext(Context newBase) {
        super.attachBaseContext(newBase);
    }

    public void applyOverrideConfiguration(Configuration overrideConfiguration) {...}

    public Configuration getOverrideConfiguration() {...}

    @Override
    public AssetManager getAssets() {...}

    @Override
    public Resources getResources() {...}

    private Resources getResourcesInternal() {...}

    @Override
    public void setTheme(int resid) {...}

    @Override
    public int getThemeResId() {...}

    @Override
    public Resources.Theme getTheme() {...}

    @Override
    public Object getSystemService(String name) {...}

    protected void onApplyThemeResource(Resources.Theme theme, int resId, boolean first) {

    private void initializeTheme() {...}
}
```

------

### 4.2 getApplication() 与 getApplicationContext()有什么区别？

返回的是同一个Applicatoin对象，但作用域不同：

- getApplicatoin()的作用域：Activity、Service，但不能在BroadcastReceiver里使用；
- getApplicationContext()的作用域会广一些，如包括BroadcastReceiver等。

### 4.3 getApplicationContext()、getBaseContext()和Activity.this的区别是什么？

- getApplicationContext()：返回的是Application类型的Context
- Activity.this：返回的是当前Activity的Context
- getBaseContext()：返回构造函数指定的Context / setBaseContext()里传入的Context



# 一、什么是Context?

context的本意是上下文、环境的意思。在Android中，Context类中提供了一系列方法，用来获取应用资源和应用信息（比如getAssets、getResources、Sp文件等，package和application的信息）和类；另外还提供了组件之间交互的方法，比如启动Activity、Service、注册广播等方法；然后还提供了其他一些应用开发中常用的方法。

## 作用

提供了一系列常用的方法：

1、获取应用资源

2、组件交互

3、权限检查等其他方法。

## Context继承关系

![Context继承关系](images/%E3%80%90%E6%80%BB%E7%BB%93%E3%80%91context/image-20220922002752656.png)

![Context继承关系2](images/%E3%80%90%E6%80%BB%E7%BB%93%E3%80%91context/image-20220922012157676.png)

# 二、Context的创建

应用程序创建Context实例的情况有如下几种：

1) 创建Application 对象时， 而且整个App共一个Application对象
2) 创建Service对象时
3) 创建Activity对象时

因此应用程序App共有的Context数目公式为：

> 总Context实例个数 = Service个数 + Activity个数 + 1（Application对应的Context实例）

但是实际上，Activity本身是一个Context，但是其内部，还有一个base context，所以这个数量可能是 activity 数量*2？

TODO：结合Activity、Service的创建过程，看下上面这个问题。



Q：getApplication() 和 getApplicationContext() 的区别？

getApplication() 是 Activity 和 Service 中的方法，getApplicationContext() 是 ContextWrapper 中的。他们的生命周期是不同的，因为 getApplication() 是属于  Activity 和 Service 的，因此其生命周期是依赖于 Activity 和 Service 的。这对于广播接收器的注册是有影响的，我们用 getApplication() 注册，其生命周期和 Activity 与 Service 绑定，如果Activity销毁了，广播会自动注销。而如果用 getApplicationContext() 注册，则会与应用的生命周期绑定。



# 三、作用域

虽然Context神通广大，但并不是随便拿到一个Context实例就可以为所欲为，它的使用还是有一些规则限制的。由于Context的具体实例是由ContextImpl类去实现的，因此在绝大多数场景下，Activity、Service和Application这三种类型的Context都是可以通用的。不过有几种场景比较特殊，比如启动Activity，还有弹出Dialog。出于安全原因的考虑，Android是不允许Activity或Dialog凭空出现的，一个Activity的启动必须要建立在另一个Activity的基础之上，也就是以此形成的返回栈。而Dialog则必须在一个Activity上面弹出（除非是System Alert类型的Dialog），因此在这种场景下，我们只能使用Activity类型的Context，否则将会出错。

![context作用域](images/%E3%80%90%E6%80%BB%E7%BB%93%E3%80%91context/image-20220922013438727.png)

从上图我们可以发现Activity所持有的Context的作用域最广，无所不能。因为Activity继承自ContextThemeWrapper，而Application和Service继承自ContextWrapper，很显然ContextThemeWrapper在ContextWrapper的基础上又做了一些操作使得Activity变得更强大，这里我就不再贴源码给大家分析了，有兴趣的童鞋可以自己查查源码。上图中的YES和NO我也不再做过多的解释了，这里我说一下上图中Application和Service所不推荐的两种使用情况。

1. 如果我们用ApplicationContext去启动一个LaunchMode为standard的Activity的时候会报错`android.util.AndroidRuntimeException: Calling startActivity from outside of an Activity context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?`

   这是因为非Activity类型的Context并没有所谓的任务栈，所以待启动的Activity就找不到栈了。解决这个问题的方法就是为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就为它创建一个新的任务栈，而此时Activity是以singleTask模式启动的。所有这种用Application启动Activity的方式不推荐使用，Service同Application。

2. 在Application和Service中去layout inflate也是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。所以这种方式也不推荐使用。

一句话总结：凡是跟UI相关的，都应该使用Activity做为Context来处理；其他的一些操作，Service,Activity,Application等实例都可以，当然了，注意Context引用的持有，防止内存泄漏。



## 如何获取Context?

通常我们想要获取Context对象，主要有以下四种方法

1. View.getContext,返回当前View对象的Context对象，通常是当前正在展示的Activity对象。
2. Activity.getApplicationContext,获取当前Activity所在的(应用)进程的Context对象，通常我们使用Context对象时，要优先考虑这个全局的进程Context。
3. ContextWrapper.getBaseContext():用来获取一个ContextWrapper进行装饰之前的Context，可以使用这个方法，这个方法在实际开发中使用并不多，也不建议使用。
4. Activity.this 返回当前的Activity实例，如果是UI控件需要使用Activity作为Context对象，但是默认的Toast实际上使用ApplicationContext也可以。

**getApplication()和getApplicationContext()**

上面说到获取当前Application对象用getApplicationContext，不知道你有没有联想到getApplication()，这两个方法有什么区别？相信这个问题会难倒不少开发者。

![](images/%E3%80%90%E6%80%BB%E7%BB%93%E3%80%91context/1240.jpeg)

程序是不会骗人的，我们通过上面的代码，打印得出两者的内存地址都是相同的，看来它们是同一个对象。其实这个结果也很好理解，因为前面已经说过了，Application本身就是一个Context，所以这里获取getApplicationContext()得到的结果就是Application本身的实例。那么问题来了，既然这两个方法得到的结果都是相同的，那么Android为什么要提供两个功能重复的方法呢？实际上这两个方法在作用域上有比较大的区别。getApplication()方法的语义性非常强，一看就知道是用来获取Application实例的，**但是这个方法只有在Activity和Service中才能调用的到**。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法了。

```java
public class MyReceiver extends BroadcastReceiver{
  @Override
  public void onReceive(Contextcontext,Intentintent){
    Application myApp= (Application)context.getApplicationContext();
  }
}
```



# 四、Context引起的内存泄露

泄漏的本质是，context被比它生命周期更长的对象所持有，导致context不能被销毁。

Context并不能随便乱用，用的不好有可能会引起内存泄露的问题，下面就示例两种错误的引用方式。

## 1、**错误的单例模式**

```java
public class Singleton {
    private static Singleton instance;
    private Context mContext;

    private Singleton(Context context) {
        this.mContext = context;
    }

    public static Singleton getInstance(Context context) {
        if (instance == null) {
            instance = new Singleton(context);
        }
        return instance;
    }
}
```

这是一个非线程安全的单例模式，instance作为静态对象，其生命周期要长于普通的对象，其中也包含Activity，假如Activity A去getInstance获得instance对象，传入this，常驻内存的Singleton保存了你传入的Activity A对象，并一直持有，即使Activity被销毁掉，但因为它的引用还存在于一个Singleton中，就不可能被GC掉，这样就导致了内存泄漏。

## 2、**View持有Activity引用**

```java
public class MainActivity extends Activity {
    private static Drawable mDrawable;

    @Override
    protected void onCreate(Bundle saveInstanceState) {
        super.onCreate(saveInstanceState);
        setContentView(R.layout.activity_main);
        ImageView iv = new ImageView(this);
        mDrawable = getResources().getDrawable(R.drawable.ic_launcher);
        iv.setImageDrawable(mDrawable);
    }
}
```

有一个静态的Drawable对象，当ImageView设置这个Drawable时，ImageView保存了mDrawable的引用，而ImageView传入的this是MainActivity的mContext，因为被static修饰的mDrawable是常驻内存的，MainActivity是它的间接引用，MainActivity被销毁时，也不能被GC掉，所以造成内存泄漏。

## 3、内部类持有Activity引用

下面是一个Handle导致Activity泄漏的示例：

```java
public class SampleActivity extends Activity {

  private final Handler mLeakyHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ...
    }
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // 延时10分钟发送一个消息
    mLeakyHandler.postDelayed(new Runnable() {
      @Override
      public void run() { }
    }, 60 * 10 * 1000);

    // 返回前一个Activity
    finish();
  }
}
```

解决：

```java
public class SampleActivity extends Activity {
    /**
    * 匿名类的静态实例不会隐式持有他们外部类的引用
    */
    private static final Runnable sRunnable = new Runnable() {
            @Override
            public void run() {
            }
        };

    private final MyHandler mHandler = new MyHandler(this);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 延时10分钟发送一个消息.
        mHandler.postDelayed(sRunnable, 60 * 10 * 1000);

        // 返回前一个Activity
        finish();
    }

    /**
    * 静态内部类的实例不会隐式持有他们外部类的引用。
    */
    private static class MyHandler extends Handler {
        private final WeakReference<SampleActivity> mActivity;

        public MyHandler(SampleActivity activity) {
            mActivity = new WeakReference<SampleActivity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            SampleActivity activity = mActivity.get();

            if (activity != null) {
                // ...
            }
        }
    }
}
```

4、



**正确使用Context**

一般Context造成的内存泄漏，几乎都是当Context销毁的时候，却因为被引用导致销毁失败，而Application的Context对象可以理解为随着进程存在的，所以我们总结出使用Context的正确姿势：

1. 当Application的Context能搞定的情况下，并且生命周期长的对象，优先使用Application的Context。
2. 不要让生命周期长于Activity的对象持有到Activity的引用。
3. 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用，如果使用静态内部类，将外部实例引用作为弱引用持有。





# FAQ：常见问题

Q：谈一下你对Android中的context的理解

Q：在一个应用程序中有多少个context实例？



<font color='orange'>Q：Application Context和Activity Context的区别</font>

<font color='orange'>Q：Application、Activity、Service中context的区别？能否启动一个Activity、Dialog?</font>

Activity继承自ContextThemeWrapper，Application 和 Service 继承自 ContextWrapper。

ContextThemeWrapper 类中新增了theme的相关的内容，这个是Activity独有的。

启动的内容见：作用域一节。

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 参考

[Carson带你学Android：如何理解 Context 对于Activity的意义？](https://www.jianshu.com/p/e70822bc29fb)
