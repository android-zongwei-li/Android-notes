View 的工作流程，指的就是 measure、layout 和 draw。其中，measure 用来测量 View 的宽和高，layout 用来确定 View 的位置，draw 则用来绘制 View。

### 1、View 的工作流程入口

在 `View的事件分发机制.md` 中，已经知道了Activity 的构成，以及 DecorView 的创建以及它加载的资源。这个时候 DecorView 的内容还无法显示，因为它还没有被加载到 Window 中。接下来我们来看看 DecorView 如何被加载到 Window 中。

1.1、DecorView 被加载到 Window 中 

当 DecorView 创建完毕，要加载到 Window 中时，我们需要先了解一下 Activity 的创建过程。当我们调用 Activity 的 startActivity 方法时，最终是调用 ActivityThread 的 handleLaunchActivity方法来创建 Activity 的，代码如下所示：

```java
    @Override
    public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {
...
    	// 注释 1、
        final Activity a = performLaunchActivity(r, customIntent);

        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            reportSizeConfigurations(r);
            if (!r.activity.mFinished && pendingActions != null) {
                pendingActions.setOldState(r.state);
                pendingActions.setRestoreInstanceState(true);
                pendingActions.setCallOnPostCreate(true);
            }
        } else {
            // If there was an error, for any reason, tell the activity manager to stop us.
            try {
                ActivityTaskManager.getService()
                        .finishActivity(r.token, Activity.RESULT_CANCELED, null,
                                Activity.DONT_FINISH_TASK_WITH_ACTIVITY);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }

        return a;
    }
```

上面代码注释 1 处调用 performLaunchActivity 方法来创建 Activity，在这里面会调用到Activity 的 onCreate 方法，从而完成 DecorView 的创建。下面详细看一下从 performLaunchActivity 到 onCreate 的调用流程：

```java
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ActivityInfo aInfo = r.activityInfo;

        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }

        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            // 注释 1、实例化 activity，内部是通过反射实例化
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } 

        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

            if (activity != null) {
                Configuration config = new Configuration(mCompatConfiguration);
                // window 初始化
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }

                // Activity resources must be initialized with the same loaders as the
                // application context.
                appContext.getResources().addLoaders(
                        app.getResources().getLoaders().toArray(new ResourcesLoader[0]));

                appContext.setOuterContext(activity);
                // 注释 2、在此方法中，准备了各种 Activity 所需要的参数，集中传递给Activity
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);

                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }

                activity.mCalled = false;
                if (r.isPersistable()) {
                    // 注释 3、执行 onCreate
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
            }
            r.setState(ON_CREATE);

            synchronized (mResourcesManager) {
                mActivities.put(r.token, r);
            }
        }
        return activity;
    }

```

1. 注释1

   ```java
   Instrumentation#newActivity
   public Activity newActivity(ClassLoader cl, String className,
           Intent intent)
           throws InstantiationException, IllegalAccessException,
           ClassNotFoundException {
       String pkg = intent != null && intent.getComponent() != null
               ? intent.getComponent().getPackageName() : null;
       return getFactory(pkg).instantiateActivity(cl, className, intent);
   }
   
   AppComponentFactory#instantiateActivity
       public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
               @Nullable Intent intent)
               throws InstantiationException, IllegalAccessException, ClassNotFoundException {
           return (Activity) cl.loadClass(className).newInstance();
       }
   ```

   AppComponentFactory类负责多个组件的实例化工作。

   ![image-20210420181401675](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210420181401675.png)

2. 注释2

   在performLaunchActivity方法中，准备了各种 Activity 所需要的参数，通过attach集中传递给Activity

3. 注释3

   ```java
   Instrumentation#callActivityOnCreate
   public void callActivityOnCreate(Activity activity, Bundle icicle,
           PersistableBundle persistentState) {
       prePerformCreate(activity);
       // 注释 here
       activity.performCreate(icicle, persistentState);
       postPerformCreate(activity);
   }
   	
   Activity#performCreate
       final void performCreate(Bundle icicle, PersistableBundle persistentState) {
           dispatchActivityPreCreated(icicle);
           mCanEnterPictureInPicture = true;
           // initialize mIsInMultiWindowMode and mIsInPictureInPictureMode before onCreate
           final int windowingMode = getResources().getConfiguration().windowConfiguration
                   .getWindowingMode();
           mIsInMultiWindowMode = inMultiWindowMode(windowingMode);
           mIsInPictureInPictureMode = windowingMode == WINDOWING_MODE_PINNED;
           restoreHasCurrentPermissionRequest(icicle);
           if (persistentState != null) {
               // 注释 here
               onCreate(icicle, persistentState);
           } else {
               onCreate(icicle);
           }
           EventLogTags.writeWmOnCreateCalled(mIdent, getComponentName().getClassName(),
                   "performCreate");
           mActivityTransitionState.readState(icicle);
   
           mVisibleFromClient = !mWindow.getWindowStyle().getBoolean(
                   com.android.internal.R.styleable.Window_windowNoDisplay, false);
           mFragments.dispatchActivityCreated();
           mActivityTransitionState.setEnterActivityOptions(this, getActivityOptions());
           dispatchActivityPostCreated(icicle);
       }
   ```

   到此 onCreate 方法就执行到了。

   

   







