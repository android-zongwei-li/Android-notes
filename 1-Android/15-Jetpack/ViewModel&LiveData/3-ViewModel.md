> 源码分析基于：2.3.0

## ViewModel

> The [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html) class is designed to store and manage UI-related data in a lifecycle conscious way.

简而言之，就是<u>在生命周期中管理数据</u>。说到生命周期，我们都知道 Android 中 `Activity` 和 `Fragment` 都有各自对应的生命周期，比如 `Activity`，它的生命周期如下图：

[![activity_lifecycle](https://leo-doc-img.oss-cn-hangzhou.aliyuncs.com/doc-img/activity_lifecycle.png?x-oss-process=style/doc-img)](https://leo-doc-img.oss-cn-hangzhou.aliyuncs.com/doc-img/activity_lifecycle.png?x-oss-process=style/doc-img)

通常我们会在 `onCreate()` 初始化数据，在 `onResume()` 中展示数据，`onDestroy()` 中释放数据，类似下面这些伪代码：

```java
onCreate() {
    initData();
}

onResume() {
    displayData()
}

onDestory() {
    recycleData();
}
```

如果没有在 `onDestory()` 中及时释放某些资源，可能还会导致内存泄漏，这是第一个问题。

第二个问题，Android 系统可能会在内存不足的情况下，回收了 `Activity`，导致 `Activity` 重建时数据会丢失，对于这种情况，Android 提供了 `onSaveInstanceState` 中保存数据，在 `onRestoreInstanceState` 和 `onCreate` 中获取。类似下面这些伪代码：

```
onSaveInstanceState(Bundle outState) {
 	outState.putData();   
}

onRestoreInstanceState(Bundle outState) {
    outState.getData();
}

onCreate(Bundle savedInstanceState) {
    if (savedInstanceState != null) {
        savedInstanceState.getData();
    }
}
```

这种方式除了重复的胶水代码以外，还存在 `Bundle` 存储只适用于支持序列化（Serializable 和 Parcelable）的少量数据，当然除了使用 Android SDK 提供的这种方案以外，我们也可以自己实现类似的方案，类似以下伪代码：

```
// Data Repository
Map<String,Data> map;

getData(String id) {
    if (map.get(id) == null){
        Data data = createData();
        map.put(id,data);
    }
    return map.get(id);
}

removeData(String id) {
    map.removeByKey(id);
}

// Activity
onCreate() {
    getData(this);
}

onDestory() {
    if (!isChangingConfigurations){
            removeData(this);
    }
}
```

看了上面解决思路后，我们再来看看 Google 提供的 `ViewModel`，它是如何解决上面提到的两个问题的。首先看下，`ViewModel`的典型用法，来源于官方文档：

首先定义一个 `MyViewModel` 继承于 `ViewModel`，这里使用 `LiveData` 作为数据源，这里我们只需要知道 `LiveData` 和 `RxJava` 是差不多的东西就可以了

```java
public class MyViewModel extends ViewModel {
    private MutableLiveData<List<User>> users;

    public LiveData<List<User>> getUsers(){
        if (users == null) {
            users = new MutableLiveData<>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // Do an asynchronous operation to fetch users.
    }
}
```

定义好 `ViewModel` 之后，我们在 `Activity` 中使用它：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		
     	// 1
        ViewModelProvider provider = new ViewModelProvider(this,          ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication()));
        // 2
        MyViewModel model = provider.get(MyViewModel.class);
        model.getUsers().observe(this, new Observer<List<User>>() {
            @Override
            public void onChanged(List<User> users) {
                // update UI
            }
        });
    }
}
```

在最新版本中，这种写法已经弃用：

```
model = ViewModelProviders.of(this).get(XXXViewModel.class);
```

现在需要先new ViewModelProvider对象，再使用这个provider去获取ViewModel。

#### 1、 new ViewModelProvider(）

需要将当前 `Activity` 或 `Fragment` 作为参数，这也是 `ViewModel` 将数据与生命周期结合起来的地方。那具体也是如何实现的呢？

```java
    public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
        this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                : NewInstanceFactory.getInstance());
    }

    public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        // 3 getViewModelStore
        this(owner.getViewModelStore(), factory);
    }

    public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
        mFactory = factory;
        mViewModelStore = store;
    }
```

##### 1.1、ViewModelStoreOwner接口

以下的Activity/Fragment都实现了此接口。

![image-20210329150730077](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210329150730077.png)

##### 1.2、Factory

`factory` 是 `ViewModel` 的工厂类，这里默认使用 `AndroidViewModelFactory` 最后返回 `ViewModelProvider`实例。

ViewModelProvider中提供了两个Factory的实现类，区别在于有无Application参数。都是通过反射的方式来实例化。

```java
public class ViewModelProvider {
	// 译：Factory的实现类负责modelClass（即ViewModel）的实例化
    public interface Factory {
        @NonNull
        <T extends ViewModel> T create(@NonNull Class<T> modelClass);
    }
    
    public static class NewInstanceFactory implements Factory {

        private static NewInstanceFactory sInstance;

        @NonNull
        static NewInstanceFactory getInstance() {
            if (sInstance == null) {
                sInstance = new NewInstanceFactory();
            }
            return sInstance;
        }

        @SuppressWarnings("ClassNewInstance")
        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            //noinspection TryWithIdenticalCatches
            try {
                return modelClass.newInstance();
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
    }
    
    public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {

        private static AndroidViewModelFactory sInstance;

        @NonNull
        public static AndroidViewModelFactory getInstance(@NonNull Application application) {
            if (sInstance == null) {
                sInstance = new AndroidViewModelFactory(application);
            }
            return sInstance;
        }

        private Application mApplication;

        public AndroidViewModelFactory(@NonNull Application application) {
            mApplication = application;
        }

        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
                //noinspection TryWithIdenticalCatches
                try {
                    return modelClass.getConstructor(Application.class).newInstance(mApplication);
                } catch (NoSuchMethodException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InstantiationException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InvocationTargetException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                }
            }
            return super.create(modelClass);
        }
    }
}
```

#### 2、ViewModelProvider.get()

```java
public class ViewModelProvider {
	private final ViewModelStore mViewModelStore;
    
	public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
    }
    
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        // 2.1
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            // 2.2
            if (mFactory instanceof OnRequeryFactory) {
                ((OnRequeryFactory) mFactory).onRequery(viewModel);
            }
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        if (mFactory instanceof KeyedFactory) {
            viewModel = ((KeyedFactory) mFactory).create(key, modelClass);
        } else {
            viewModel = mFactory.create(modelClass);
        }
        mViewModelStore.put(key, viewModel);
        return (T) viewModel;
    }
}
```

##### 2.1 ViewModelStore.get()

ViewModelStore是真正负责存储ViewModel的。

##### 2.2 OnRequeryFactory，KeyedFactory

也是`ViewModel` 的工厂类，在创建ViewModel的时候会传入参数key。

```java
    static class OnRequeryFactory {
        void onRequery(@NonNull ViewModel viewModel) {
        }
    }

    abstract static class KeyedFactory extends OnRequeryFactory implements Factory {

        @NonNull
        public abstract <T extends ViewModel> T create(@NonNull String key,
                @NonNull Class<T> modelClass);

        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            throw new UnsupportedOperationException("create(String, Class<?>) must be called on "
                    + "implementaions of KeyedFactory");
        }
    }
```

在说ViewModelStore之前先来总结一下：

将以上两段代码合起来看，其实逻辑是比较清晰的：

[![ViewModelProvider#create](https://leo-doc-img.oss-cn-hangzhou.aliyuncs.com/doc-img/viewmodel/ViewModelProvider%23create.png?x-oss-process=style/doc-img)](https://leo-doc-img.oss-cn-hangzhou.aliyuncs.com/doc-img/viewmodel/ViewModelProvider%23create.png?x-oss-process=style/doc-img)

Factor 的实现可以通过反射来实现，比如默认的 `AndroidViewModelFactory` 会优先调用使用 `Application` 作为参数的构造方法，来创建实例。所以，如果自定义的 `ViewModel` 构造方法有其他参数，就需要自定义 `Factor`

#### 3 ViewModelStore

 `ViewModelStore` 是 `Activity` 重建时还能拥有之前数据的保障。

```java
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    Set<String> keys() {
        return new HashSet<>(mMap.keySet());
    }

    /**
     *  Clears internal storage and notifies ViewModels that they are no longer used.
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.clear();
        }
        mMap.clear();
    }
}
```

`ViewModelStore` 的源码很短，可以看到其实就是使用 `HashMap` 作为数据载体。既然有使用，那就需要清理操作，可以看到有个 `clear` 它会清除 `HashMap` 中的缓存数据。我们首先看下 `ViewModelProvider` 中的 `mViewModelStore` 是在哪里赋值的：

```java
public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) { 
    // 4
    this(owner.getViewModelStore(), factory);                                  
}                                                                                     
public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {      
    mFactory = factory;                                                         
    this.mViewModelStore = store;                                               
}
```

通过 `ViewModelStoreOwner.getViewModelStore` 获取 `ViewModelStore` 实例对象，而 `ViewModelStoreOwner` 实际就是我们调用 `new ViewModelProvider` 中传递进来的的 `FragmentActivity` 和 `Fragment`。

#### 4、owner.getViewModelStore()

看一下ComponentActivity的实现：

```java
    private ViewModelStore mViewModelStore;

	public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        if (mViewModelStore == null) {
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                // Restore the ViewModelStore from NonConfigurationInstances
                mViewModelStore = nc.viewModelStore;
            }
            if (mViewModelStore == null) {
                mViewModelStore = new ViewModelStore();
            }
        }
        return mViewModelStore;
    }
```

通过new ViewModelStore()进行实例化，并持有mViewModelStore对象。

#### 5 数据保持与恢复

```java
    public final Object onRetainNonConfigurationInstance() {
        Object custom = onRetainCustomNonConfigurationInstance();

        ViewModelStore viewModelStore = mViewModelStore;
        if (viewModelStore == null) {
            // No one called getViewModelStore(), so see if there was an existing
            // ViewModelStore from our last NonConfigurationInstance
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                viewModelStore = nc.viewModelStore;
            }
        }

        if (viewModelStore == null && custom == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.custom = custom;
        nci.viewModelStore = viewModelStore;
        return nci;
    }
```

> 在`ComponentActivity` 中会通过 `onRetainNonConfigurationInstance` 和 `getLastNonConfigurationInstance` 去保持 `Activity` 因为屏幕旋转等配置发生改变而导致重建时，数据的唯一性。

恢复则使用 **4** 中的getViewModelStore()方法获取ViewModelStore，进而恢复数据。

### 小结

现在我们来比较下自定义实现的方案和 `ViewModel` ，可以发现其实核心思想是共通的，首先我们需要一个保存于 `Activity` 和 `Fragment` 生命周期之外的存储空间，在 `ViewModel` 中是 `ViewModelStore`，其次我们需要在 `Activity` 和 `Fragment`对应的生命周期中，去初始化和清理这个 `ViewModelStore`



涉及到的类：

ViewModel，Factory，ViewModelStore，ComponentActivity，ViewModelProvider。

Factory：实例化ViewModel

ViewModelStore：存储ViewModel

ComponentActivity：持有一个ViewModelStore。实现了ViewModelStoreOwner接口，用于获取ViewModelStore。在`onRetainNonConfigurationInstance` 方法中保留ViewModelStore，使用`getLastNonConfigurationInstance`获取上次保留的ViewModelStore。

ViewModelProvider：用于操作ViewModel，Factory，ViewModelStore，ComponentActivity这四者。构造方法从ComponentActivity中获取ViewModelStore；get方法用于从ViewModelStore中获取ViewModel，若不存在，则通过Factory创建。

