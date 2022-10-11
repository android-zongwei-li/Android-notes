```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button btnOnClick = findViewById(R.id.btn_on_click);
        btnOnClick.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 1、
                startActivity(new Intent(MainActivity.this, BActivity.class));
            }
        });
    }
}
```



#### 一、Activity 中启动 Activity 的流程

Activity.java

```java
    @Override
    public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }
    
    @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        ...
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
			// 1.0、
            startActivityForResult(intent, -1);
        }
    }    

	// 1.2、
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode) {
        startActivityForResult(intent, requestCode, null);
    }
	// 1.3、
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        if (mParent == null) {
            // 1.3.1、mParent == null
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
                mStartedActivity = true;
            }

            cancelInputsAndStartExitTransition(options);
        } else {
            // 1.3.2、mParent != null
            if (options != null) {
                mParent.startActivityFromChild(this, intent, requestCode, options);
            } else {
                mParent.startActivityFromChild(this, intent, requestCode);
            }
        }
    }

    @Deprecated
    public void startActivityFromChild(@NonNull Activity child, @RequiresPermission Intent intent,
            int requestCode) {
        startActivityFromChild(child, intent, requestCode, null);
    }

    @Deprecated
    public void startActivityFromChild(@NonNull Activity child, @RequiresPermission Intent intent,
            int requestCode, @Nullable Bundle options) {
        options = transferSpringboardActivityOptions(options);
        Instrumentation.ActivityResult ar =
            mInstrumentation.execStartActivity(
                this, mMainThread.getApplicationThread(), mToken, child,
                intent, requestCode, options);
        if (ar != null) {
            mMainThread.sendActivityResult(
                mToken, child.mEmbeddedID, requestCode,
                ar.getResultCode(), ar.getResultData());
        }
        cancelInputsAndStartExitTransition(options);
    }
```

1、无论 mParent 是否为 null，代码最终都会执行：

```java
    Instrumentation.ActivityResult ar =
        mInstrumentation.execStartActivity(
            this, mMainThread.getApplicationThread(), mToken, this,
            intent, requestCode, options);
```

并且传递的参数也是一样的，有：

1. this/child：当前执行启动的 Activity。

2. mMainThread.getApplicationThread()：mAppThread，mMainThread 是 ActivityThread。

3. mToken：

   如下，mMainThread、mToken 都是 attach 的时候传入的。

   ```java
   Activity#attach
   	final void attach(Context context, ActivityThread aThread,
               Instrumentation instr, IBinder token, int ident,
               Application application, Intent intent, ActivityInfo info,
               CharSequence title, Activity parent, String id,
               NonConfigurationInstances lastNonConfigurationInstances,
               Configuration config, String referrer, IVoiceInteractor voiceInteractor,
               Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
           mMainThread = aThread;
           mInstrumentation = instr;
           mToken = token;
       }
   
   这个方法是在 ActivityThread#performLaunchActivity 中调用的：
       private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
       activity.attach(appContext, this, getInstrumentation(), r.token,
               r.ident, app, r.intent, r.activityInfo, title, r.parent,
               r.embeddedID, r.lastNonConfigurationInstances, config,
               r.referrer, r.voiceInteractor, window, r.configCallback,
               r.assistToken);
   	}
   ```

   > 以后还要分析 performLaunchActivity 的调用栈，以及 ActivityClientRecord 的创建过程。分析完这两点就可以清晰知道启动时传递的参数意义了。

4. requestCode：-1

5. mInstrumentation：这个在下一步（二）中要使用，首先也是从 Activity#attach 中传入的。我们来看下它的初始化：

   ```java
   ActivityThread#attach   
       private void attach(boolean system, long startSeq) {
           if (!system) {
           } else {
                   mInstrumentation = new Instrumentation();
                   mInstrumentation.basicInit(this);
           }
       }
   
   ActivityThread#handleBindApplication
       private void handleBindApplication(AppBindData data) {
           // Instrumentation info affects the class loader, so load it before
           // setting up the app context.
           final InstrumentationInfo ii;
           if (data.instrumentationName != null) {
               try {
                   ii = new ApplicationPackageManager(
                           null, getPackageManager(), getPermissionManager())
                           .getInstrumentationInfo(data.instrumentationName, 0);
               } 
               mInstrumentationPackageName = ii.packageName;
               mInstrumentationAppDir = ii.sourceDir;
               mInstrumentationSplitAppDirs = ii.splitSourceDirs;
               mInstrumentationLibDir = getInstrumentationLibrary(data.appInfo, ii);
               mInstrumentedAppDir = data.info.getAppDir();
               mInstrumentedSplitAppDirs = data.info.getSplitAppDirs();
               mInstrumentedLibDir = data.info.getLibDir();
           } else {
               ii = null;
           }
   
           // Continue loading instrumentation.
           if (ii != null) {
               ApplicationInfo instrApp;
               try {
                   instrApp = getPackageManager().getApplicationInfo(ii.packageName, 0,
                           UserHandle.myUserId());
               } catch (RemoteException e) {
                   instrApp = null;
               }
               if (instrApp == null) {
                   instrApp = new ApplicationInfo();
               }
               ii.copyTo(instrApp);
               instrApp.initForUser(UserHandle.myUserId());
               final LoadedApk pi = getPackageInfo(instrApp, data.compatInfo,
                       appContext.getClassLoader(), false, true, false);
               final ContextImpl instrContext = ContextImpl.createAppContext(this, pi,
                       appContext.getOpPackageName());
   
               try {
                   final ClassLoader cl = instrContext.getClassLoader();
                   mInstrumentation = (Instrumentation)
                       cl.loadClass(data.instrumentationName.getClassName()).newInstance();
               } else {
               mInstrumentation = new Instrumentation();
               mInstrumentation.basicInit(this);
           }
       }
   ```

   

#### 二、Instrumentation 中启动 Activity

现在起，启动 Activity 的任务交给了 Instrumentation。

```java
Instrumentation.ActivityResult ar =
    mInstrumentation.execStartActivity(
        this, mMainThread.getApplicationThread(), mToken, this,
        intent, requestCode, options);
```