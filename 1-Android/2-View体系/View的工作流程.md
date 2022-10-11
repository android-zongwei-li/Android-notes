> 知识路径：Android > 2-View体系
>
> version：2021/4/21
>
> review：2021/4/21
>
> 掌握程度：初学



View 的工作流程，指的就是 measure、layout 和 draw。其中，measure 用来测量 View 的宽和高，layout 用来确定 View 的位置，draw 则用来绘制 View。

### 1、View 的工作流程入口

在 `View的事件分发机制.md` 中，已经知道了Activity 的构成，以及 DecorView 的创建以及它加载的资源。这个时候 DecorView 的内容还无法显示，因为它还没有被加载到 Window 中。接下来我们来看看 DecorView 如何被加载到 Window 中。

#### 1.1、DecorView 被加载到 Window 中 

当 DecorView 创建完毕，要加载到 Window 中时，我们需要先了解一下 Activity 的创建过程。当我们调用 Activity 的 startActivity 方法时，最终是调用 ActivityThread 的 handleLaunchActivity方法来创建 Activity 的，代码如下所示：

```java
    @Override
    public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {
...
    	// 注释 1、
        final Activity a = performLaunchActivity(r, customIntent);
        
        return a;
    }
```

上面代码注释 1 处调用 performLaunchActivity 方法来创建 Activity，在这里面会调用到Activity 的 onCreate 方法，从而完成 DecorView 的创建。下面详细看一下从 performLaunchActivity 到 onCreate 的调用流程：

```java
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            // 注释 1、实例化 activity，内部是通过反射实例化
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
        } 

        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

            if (activity != null) {
                // window 初始化
                Window window = null;
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

小结：从 handleLaunchActivity 到 onCreate 的流程如下：

handleLaunchActivity

​	performLaunchActivity

​		activity = null

​		activity = mInstrumentation.newActivity

​		window = null

​		activity.attach

​		mInstrumentation.callActivityOnCreate

​			activity.performCreate

​				onCreate

在执行完 handleLaunchActivity 的整个流程后，系统会调用 handleResumeActivity 方法。下面来看下 handleResumeActivity 做了什么：

```java
    @Override
    public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
            String reason) {
		// step 1、最终执行 onResunme
        final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);
        if (r == null) {
            return;
        }
        final Activity a = r.activity;
        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            // step 2、获取（或者实例化）DecorView
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            // step 3、获取 WindowManager
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();
            a.mDecor = decor;
            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
            l.softInputMode |= forwardBit;
            if (r.mPreserveWindow) {
                a.mWindowAdded = true;
                r.mPreserveWindow = false;
                ViewRootImpl impl = decor.getViewRootImpl();
                if (impl != null) {
                    impl.notifyChildRebuilt();
                }
            }
            if (a.mVisibleFromClient) {
                if (!a.mWindowAdded) {
                    a.mWindowAdded = true;
                    // step 4、
                    wm.addView(decor, l);
                } else {
                    a.onWindowAttributesChanged(l);
                }
            }
    }
```

1. step 1

   ```java
       public ActivityClientRecord performResumeActivity(IBinder token, boolean finalStateRequest,
               String reason) {
           final ActivityClientRecord r = mActivities.get(token);
           if (r == null || r.activity.mFinished) {
               return null;
           }
           try {
   			// step 1
               r.activity.performResume(r.startsNotResumed, reason);
           }
           return r;
       }
   
   Activity#performResume
       final void performResume(boolean followedByPause, String reason) {
           ...
           // mResumed is set by the instrumentation
           mInstrumentation.callActivityOnResume(this);
   		...
       }
   
   Instrumentation#callActivityOnResume
       public void callActivityOnResume(Activity activity) {
           activity.onResume();
       ...
       }
   ```

   handleResumeActivity

   ​	performResumeActivity

   ​		r.activity.performResume

   ​			performResume

   ​				mInstrumentation.callActivityOnResume

   ​					activity.onResume()

2. step 2

   获取DecorView。

   在 onCreate 中，通常都会执行 setContentView，然后 installDecor。因此到这个步骤的时候，再去 getDecorView，就会直接返回 decorView 了。总之，无论如何，最晚在这个步骤，DecorView 就会被实例化。

   ```java
   PhoneWindow#getDecorView
   	@Override
   	public final @NonNull View getDecorView() {
       	if (mDecor == null || mForceDecorInstall) {
           	installDecor();
       	}
       	return mDecor;
   	}
   
   PhoneWindow#installDecor
       private void installDecor() {
           mForceDecorInstall = false;
           if (mDecor == null) {
               mDecor = generateDecor(-1);
           } else {
               mDecor.setWindow(this);
           }
           if (mContentParent == null) {
               mContentParent = generateLayout(mDecor);
               ...
           }
       }
   
   PhoneWindow#generateDecor
       protected DecorView generateDecor(int featureId) {
   		...
           return new DecorView(context, featureId, this, getAttributes());
       }
   ```

   handleResumeActivity

   ​	r.window.getDecorView

   ​		installDecor

   ​			generateDecor

   ​				new DecorView

3. step 3

   获取 WindowManager。

   ```java
   Activity#attach
   	final void attach(Context context, ActivityThread aThread,
               Instrumentation instr, IBinder token, int ident,
               Application application, Intent intent, ActivityInfo info,
               CharSequence title, Activity parent, String id,
               NonConfigurationInstances lastNonConfigurationInstances,
               Configuration config, String referrer, IVoiceInteractor voiceInteractor,
               Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
   
           mWindow = new PhoneWindow(this, window, activityConfigCallback);
           
       	// step 1、在这里
           mWindow.setWindowManager(
                   (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                   mToken, mComponent.flattenToString(),
                   (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
           if (mParent != null) {
               mWindow.setContainer(mParent.getWindow());
           }
           mWindowManager = mWindow.getWindowManager();
       }
   
   Window#setWindowManager
       public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
               boolean hardwareAccelerated) {
       ...
           mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
       }
   
   WindowManagerImpl#createLocalWindowManager
       public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
           return new WindowManagerImpl(mContext, parentWindow);
       }
   
   Window#getWindowManager
       public WindowManager getWindowManager() {
           return mWindowManager;
       }
   ```

   Activity#attach

   ​	mWindow = new PhoneWindow

   ​		mWindow.setWindowManager

   ​			mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager

   ​				new WindowManagerImpl

   ​	mWindowManager = mWindow.getWindowManager

   ​		mWindowManager

   下面是WindowManager、ViewManager、WindowManagerImpl之间的关系，以及和Window、PhoneWindow、Activity的关系：

   ```java
   public interface WindowManager extends ViewManager
   
   WindowManagerImpl implements WindowManager
   
       -----
   public abstract class Window
       
   public class PhoneWindow extends Window
       
   window = new PhoneWindow
       -----
   然后 window 中持有一个 WindowManager 变量，
       通过 setWindowManager、getWindowManager 设置和获取。
   ```

4. step 4

   调用 WindowManager 的 addView 方法，WindowManager 的实现类是 WindowManagerImpl，所以实际调用的是 WindowManagerImpl 的 addView 方法。

   ```java
       @Override
       public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
           applyDefaultToken(params);
           mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                   mContext.getUserId());
       }
   ```

   在 WindowManagerImpl 的 addView 方法中，又调用了 WindowManagerGlobal 的 addView方法，代码如下所示：

   ```java
       private final ArrayList<View> mViews = new ArrayList<View>();
       private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
   
   	public void addView(View view, ViewGroup.LayoutParams params,
               Display display, Window parentWindow, int userId) {
   
           ViewRootImpl root;
           View panelParentView = null;
   
           synchronized (mLock) {
               // step 1、
               root = new ViewRootImpl(view.getContext(), display);
   
               view.setLayoutParams(wparams);
   
               mViews.add(view);
               mRoots.add(root);
               mParams.add(wparams);
   
               // do this last because it fires off messages to start doing things
               try {
                   // step 2、
                   root.setView(view, wparams, panelParentView, userId);
               } catch (RuntimeException e) {
                   // BadTokenException or InvalidDisplayException, clean up.
                   if (index >= 0) {
                       removeViewLocked(index, true);
                   }
                   throw e;
               }
           }
       }
   
   ```

   在上面代码注释 1 处创建了 ViewRootImpl 实例，在注释 2 处调用了 ViewRootImpl 的 setView方法并将 DecorView 作为参数传进去，这样就把 DecorView 加载到了 Window 中。当然界面仍不会显示出什么来，因为 View 的工作流程还没有执行完，还需要经过 measure、layout 以及 draw才会把 View 绘制出来。

   特别说明，WindowManagerGlobal 是一个单例，管理者所有的 DecorView 和 ViewRootImpl。

到此 Activity 创建了 Window，Window 创建了 DecorView、设置了 WindowManager，DecorView  被添加到 WindowManager 中，实际是添加到 WindowManagerGlobal 中。在 WindowManagerGlobal 中，又把 DecorView  添加到了 ViewRootImpl 中。

#### 1.2、ViewRootlmpl 的 PerformTraveals 方法

前面讲到了将 DecorView 加载到 Window 中以后，还通过 ViewRootImpl 的 setView 方法加入到了 ViewRootImpl 中。ViewRootImpl 还有一个方法 performTraversals，这个方法使得 ViewTree 开始 View 的工作流程，代码如下所示：

```java
    private void performTraversals() {
        // cache mView since it is used so much below...
        // mView 就是 decorView
        final View host = mView;

        if (host == null || !mAdded)
...
            if (!mStopped || mReportNextDraw) {
...
                     // Ask host how big it wants to be
    				// step 1、
                    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
            }
        } 
        
        final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
        if (didLayout) {
            // step 2、
            performLayout(lp, mWidth, mHeight);
        }

        boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() || !isViewVisible;
		// step 3、
        if (!cancelDraw) {
            performDraw();
        } 
    }
```

这里面主要执行了 3 个方法，分别是 performMeasure、performLayout 和 performDraw，在其方法的内部又会分别调用 View的 measure、layout 和 draw方法。需要注意的是performMeasure方法中需要传入两个参数，分别是 childWidthMeasureSpec 和 childHeightMeasureSpec。要了解这两个参数，需要了解 MeasureSpec。

### 2、MeasureSpec

MeasureSpec 是 View 的内部类，其封装了一个 View 的规格尺寸，包括 View 的宽和高的信息，它的作用是在 Measure 流程中，系统会将 View 的 LayoutParams 根据父容器所施加的规则转换成对应的 MeasureSpec，然后在 onMeasure 方法中根据这个 MeasureSpec 来确定 View 的宽和高。MeasureSpec 的代码如下所示：

```java
    public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

        /** @hide */
        @IntDef({UNSPECIFIED, EXACTLY, AT_MOST})
        @Retention(RetentionPolicy.SOURCE)
        public @interface MeasureSpecMode {}

        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        public static final int EXACTLY     = 1 << MODE_SHIFT;
        public static final int AT_MOST     = 2 << MODE_SHIFT;

        public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size,
                                          @MeasureSpecMode int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }

        @UnsupportedAppUsage
        public static int makeSafeMeasureSpec(int size, int mode) {
            if (sUseZeroUnspecifiedMeasureSpec && mode == UNSPECIFIED) {
                return 0;
            }
            return makeMeasureSpec(size, mode);
        }

        @MeasureSpecMode
        public static int getMode(int measureSpec) {
            //noinspection ResourceType
            return (measureSpec & MODE_MASK);
        }

        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

        static int adjust(int measureSpec, int delta) {
            final int mode = getMode(measureSpec);
            int size = getSize(measureSpec);
            if (mode == UNSPECIFIED) {
                // No need to adjust size for UNSPECIFIED mode.
                return makeMeasureSpec(size, UNSPECIFIED);
            }
            size += delta;
            if (size < 0) {
                Log.e(VIEW_LOG_TAG, "MeasureSpec.adjust: new size would be negative! (" + size +
                        ") spec: " + toString(measureSpec) + " delta: " + delta);
                size = 0;
            }
            return makeMeasureSpec(size, mode);
        }

        public static String toString(int measureSpec) {
            int mode = getMode(measureSpec);
            int size = getSize(measureSpec);

            StringBuilder sb = new StringBuilder("MeasureSpec: ");

            if (mode == UNSPECIFIED)
                sb.append("UNSPECIFIED ");
            else if (mode == EXACTLY)
                sb.append("EXACTLY ");
            else if (mode == AT_MOST)
                sb.append("AT_MOST ");
            else
                sb.append(mode).append(" ");

            sb.append(size);
            return sb.toString();
        }
    }
```

从 MeasureSpec 的常量可以看出，它代表了 32 位的 int 值，其中高 2 位代表了 SpecMode， 低 30 位则代表 SpecSize。SpecMode 指的是测量模式，SpecSize 指的是测量大小。SpecMode 有 3 种模式，如下所示。

- UNSPECIFIED：未指定模式，View 想多大就多大，父容器不做限制，一般用于系统内部的测量。

- AT_MOST：最大模式，对应于 wrap_comtent 属性，子 View 的最终大小是父 View 指定的 SpecSize 值，并且子 View 的大小不能大于这个值。

- EXACTLY：精确模式，对应于 match_parent 属性和具体的数值，父容器测量出 View所需要的大小，也就是 SpecSize 的值。

对于每一个 View，都持有一个 MeasureSpec，而该 MeasureSpec 则保存了该 View 的尺寸规格。在 View 的测量流程中，通过 makeMeasureSpec 来保存宽和高的信息。通过 getMode 或 getSize得到模式和宽、高。MeasureSpec 是受自身 LayoutParams 和父容器的 MeasureSpec 共同影响的。作为顶层 View 的 DecorView 来说，其并没有父容器，那么它的 MeasureSpec 是如何得来的呢？为了解决这个疑问，我们再回到 ViewRootImpl 的 PerformTraveals 方法，如下所示：

```java
	private void performTraversals() {
            ...
            if (!mStopped || mReportNextDraw) {
                    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
                    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
            }
            ...
}

    private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }
```

getRootMeasureSpec方法的第一个参数 windowSize 指的是窗口的尺寸，所以对于DecorView来说，它的 MeasureSpec 由自身的 LayoutParams 和窗口的尺寸决定，这一点和普通 View 是不同的。接着往下看，就会看到根据自身的 LayoutParams 来得到不同的 MeasureSpec。

performMeasure 方法中需要传入两个参数，即 childWidthMeasureSpec和 childHeightMeasureSpec，下面看看 performMeasure 方法做了什么，代码如下所示：

```java
    private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
        try {
            mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
    }
```

果然是 View 的 measure 方法。接下来看一下 View 的 measure 流程。

### 3、View 的 measure 流程

measure 用来测量 View 的宽和高，它的流程分为 View 的 measure 流程和 ViewGroup 的measure 流程，只不过 ViewGroup 的 measure 流程除了要完成自己的测量，还要遍历地调用子元素的 measure()方法。

#### 3.1、View 的 measure 流程

```java
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
            onMeasure(widthMeasureSpec, heightMeasureSpec);
	}
看一下 View 的 onMeasure 方法：
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }

    public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }

这是用来设置 View 的宽、高的
    protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
        boolean optical = isLayoutModeOptical(this);
        if (optical != isLayoutModeOptical(mParent)) {
            Insets insets = getOpticalInsets();
            int opticalWidth  = insets.left + insets.right;
            int opticalHeight = insets.top  + insets.bottom;

            measuredWidth  += optical ? opticalWidth  : -opticalWidth;
            measuredHeight += optical ? opticalHeight : -opticalHeight;
        }
        setMeasuredDimensionRaw(measuredWidth, measuredHeight);
    }
```

SpecMode 是 View 的测量模式，而 SpecSize 是 View 的测量大小。在 getDefaultSize 方法中，根据不同的 SpecMode 值来返回不同的 result 值，也就是 SpecSize。 在 AT_MOST 和 EXACTLY 模式下，都返回 SpecSize 这个值，即 View 在这两种模式下的测量宽和高直接取决于 SpecSize。也就是说，对于一个直接继承自 View 的自定义 View 来说，它的 wrap_content 和 match_parent 属性的效果是一样的。因此如果要实现自定义 View 的 wrap_content，则要重写 onMeasure 方法，并对自定义 View 的 wrap_content 属性进行处理。而在 UNSPECIFIED 模式下返回的是 getDefaultSize 方法的第一个参数 size 的值，size 的值从 onMeasure 方法来看是 getSuggestedMinimumWidth方法或者 getSuggestedMinimumHeight 方法得到的。我们来查看 getSuggestedMinimumWidth 方法做了什么。

```java
    protected int getSuggestedMinimumWidth() {
        return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
    }
    
    protected int getSuggestedMinimumHeight() {
        return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
    }    
```

如果 View 没有设置背景，则取值为 mMinWidth，mMinWidth 是可以设置的，它对应于Android：minWidth 这个属性设置的值或者 View 的 setMinimumWidth 的值；如果不指定的话，则默认为 0。setMinimumWidth 方法如下所示：

```java
    public void setMinimumWidth(int minWidth) {
        mMinWidth = minWidth;
        requestLayout();
    }
```

如果 View 设置了背景，则取值为 max(mMinWidth, mBackground.getMinimumWidth())，也就是取 mMinWidth 和 mBackground.getMinimumWidth()之间的最大值。此前讲了 mMinWidth，下面看看 mBackground.getMinimumWidth()。这个 mBackground 是 Drawable 类型的，Drawable类的 getMinimumWidth 方法如下所示：

```java
    public int getMinimumWidth() {
        final int intrinsicWidth = getIntrinsicWidth();
        return intrinsicWidth > 0 ? intrinsicWidth : 0;
    }
```

intrinsicWidth 得到的是这个 Drawable 的固有宽度，如果固有宽度大于 0 则返回固有宽度，否则返回 0。总结一下，getSuggestedMinimumWidth 方法就是：如果 View 没有设置背景，则返回 mMinWidth；如果设置了背景，就返回 mMinWidth 和 Drawable 的最小宽度之间的最大值。

#### 3.2 ViewGroup 的 measure 流程

讲完了 View 的 measure 流程，接下来看看 ViewGroup 的 measure 流程。对于 ViewGroup，它不只要测量自身，还要遍历地调用子元素的 measure()方法。ViewGroup 中没有定义 onMeasure()方法，但却定义了 measureChildren()方法：

```java
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
```

遍历子元素并调用 measureChild 方法，measureChild 方法如下所示：

```java
    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

调用 child.getLayoutParams()方法来获得子元素的 LayoutParams 属性，获取子元素的MeasureSpec 并调用子元素的 measure()方法进行测量。getChildMeasureSpec()方法里写了什么呢？其代码如下：

```java
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                // step 1、
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

很显然，这是根据父容器的 MeasureSpec 模式再结合子元素的 LayoutParams 属性来得出的子元素的 MeasureSpec 属性。有一点需要注意的是，如果父容器的 MeasureSpec 属性为AT_MOST，子元素的 LayoutParams 属性为 WRAP_CONTENT，那根据上面代码注释 1 处的代码，我们会发现子元素的MeasureSpec属性也为AT_MOST，它的SpecSize值为父容器的SpecSize减去 padding 的值。换句话说，这和子元素设置 LayoutParams 属性为 MATCH_PARENT 效果是一样的。为了解决这个问题，需要在 LayoutParams 属性为 WRAP_CONTENT 时指定一下默认的宽和高。ViewGroup 并没有提供 onMeasure 方法，而是让其子类来各自实现测量的方法，究其原因就是 ViewGroup 有不同布局的需要，很难统一。接下来我们简单分析一下 ViewGroup 的子类 LinearLayout 的 measure 流程。现在先来看看它的 onMeasure 方法，代码如下所示：

```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }
```

这个方法的逻辑很简单，如果是垂直方向则调用 measureVertical 方法，否则就调用measureHorizontal 方法。接着分析垂直 measureVertical()方法的部分源码：

```java
    void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
        mTotalLength = 0;

        // See how tall everyone is. Also remember max width.
        for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                mTotalLength += measureNullChild(i);
                continue;
            }

            if (child.getVisibility() == View.GONE) {
               i += getChildrenSkipCount(child, i);
               continue;
            }

            nonSkippedChildCount++;
            if (hasDividerBeforeChildAt(i)) {
                mTotalLength += mDividerHeight;
            }

            final LayoutParams lp = (LayoutParams) child.getLayoutParams();

            totalWeight += lp.weight;

            final boolean useExcessSpace = lp.height == 0 && lp.weight > 0;
            if (heightMode == MeasureSpec.EXACTLY && useExcessSpace) {
                final int totalLength = mTotalLength;
                mTotalLength = Math.max(totalLength, totalLength + lp.topMargin + lp.bottomMargin);
                skippedMeasure = true;
            } else {
                if (useExcessSpace) {
                    lp.height = LayoutParams.WRAP_CONTENT;
                }
                final int usedHeight = totalWeight == 0 ? mTotalLength : 0;
                measureChildBeforeLayout(child, i, widthMeasureSpec, 0,
                        heightMeasureSpec, usedHeight);
                final int childHeight = child.getMeasuredHeight();
                if (useExcessSpace) {
                    lp.height = 0;
                    consumedExcessSpace += childHeight;
                }

                final int totalLength = mTotalLength;
                mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
                       lp.bottomMargin + getNextLocationOffset(child));

                if (useLargestChild) {
                    largestChildHeight = Math.max(childHeight, largestChildHeight);
                }
            }
...
        if (useLargestChild &&
                (heightMode == MeasureSpec.AT_MOST || heightMode == MeasureSpec.UNSPECIFIED)) {
            mTotalLength = 0;

            for (int i = 0; i < count; ++i) {
                final View child = getVirtualChildAt(i);
                if (child == null) {
                    mTotalLength += measureNullChild(i);
                    continue;
                }

                if (child.getVisibility() == GONE) {
                    i += getChildrenSkipCount(child, i);
                    continue;
                }

                final LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams)
                        child.getLayoutParams();
                // Account for negative margins
                final int totalLength = mTotalLength;
                mTotalLength = Math.max(totalLength, totalLength + largestChildHeight +
                        lp.topMargin + lp.bottomMargin + getNextLocationOffset(child));
            }
        }

        // Add in our padding
        mTotalLength += mPaddingTop + mPaddingBottom;

        int heightSize = mTotalLength;
    }
```

这里定义了 mTotalLength 用来存储 LinearLayout 在垂直方向的高度，然后遍历子元素，根据子元素的 MeasureSpec 模式分别计算每个子元素的高度。如果是 WRAP_CONTENT，则将每个子元素的高度和 margin 垂直高度等值相加并赋值给 mTotalLength。当然，最后还要加上垂直方向 padding 的值。如果布局高度设置为 MATCH_PARENT 或者具体数值，则和 View 的测量方法是一样的。measure 流程就讲到这里了，接下来讲解 View 的 layout 和 draw 流程。

#### 3.3  View 的 layout 流程

layout 方法的作用是确定元素的位置。ViewGroup 中的 layout 方法用来确定子元素的位置，View 中的 layout 方法则用来确定自身的位置。首先我们看看 View 的 layout 方法：

```java
    public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        // step 1、
        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);

            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }

            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        final boolean wasLayoutValid = isLayoutValid();

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;

        if (!wasLayoutValid && isFocused()) {
            mPrivateFlags &= ~PFLAG_WANTS_FOCUS;
            if (canTakeFocus()) {
                // We have a robust focus, so parents should no longer be wanting focus.
                clearParentsWantFocus();
            } else if (getViewRootImpl() == null || !getViewRootImpl().isInLayout()) {
                // This is a weird case. Most-likely the user, rather than ViewRootImpl, called
                // layout. In this case, there's no guarantee that parent layouts will be evaluated
                // and thus the safest action is to clear focus here.
                clearFocusInternal(null, /* propagate */ true, /* refocus */ false);
                clearParentsWantFocus();
            } else if (!hasParentWantsFocus()) {
                // original requestFocus was likely on this view directly, so just clear focus
                clearFocusInternal(null, /* propagate */ true, /* refocus */ false);
            }
            // otherwise, we let parents handle re-assigning focus during their layout passes.
        } else if ((mPrivateFlags & PFLAG_WANTS_FOCUS) != 0) {
            mPrivateFlags &= ~PFLAG_WANTS_FOCUS;
            View focused = findFocus();
            if (focused != null) {
                // Try to restore focus as close as possible to our starting focus.
                if (!restoreDefaultFocus() && !hasParentWantsFocus()) {
                    // Give up and clear focus once we've reached the top-most parent which wants
                    // focus.
                    focused.clearFocusInternal(null, /* propagate */ true, /* refocus */ false);
                }
            }
        }

        if ((mPrivateFlags3 & PFLAG3_NOTIFY_AUTOFILL_ENTER_ON_LAYOUT) != 0) {
            mPrivateFlags3 &= ~PFLAG3_NOTIFY_AUTOFILL_ENTER_ON_LAYOUT;
            notifyEnterOrExitForAutoFillIfNeeded(true);
        }

        notifyAppearedOrDisappearedForContentCaptureIfNeeded(true);
    }
```

layout 方法的 4 个参数 l、t、r、b 分别是 View 从左、上、右、下相对于其父容器的距离。

step 1、来查看 setFrame 方法里做了什么，代码如下所示：

```java
    protected boolean setFrame(int left, int top, int right, int bottom) {
        boolean changed = false;

        if (DBG) {
            Log.d(VIEW_LOG_TAG, this + " View.setFrame(" + left + "," + top + ","
                    + right + "," + bottom + ")");
        }

        if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
            changed = true;

            // Remember our drawn bit
            int drawn = mPrivateFlags & PFLAG_DRAWN;

            int oldWidth = mRight - mLeft;
            int oldHeight = mBottom - mTop;
            int newWidth = right - left;
            int newHeight = bottom - top;
            boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);

            // Invalidate our old position
            invalidate(sizeChanged);

            mLeft = left;
            mTop = top;
            mRight = right;
            mBottom = bottom;
            mRenderNode.setLeftTopRightBottom(mLeft, mTop, mRight, mBottom);
...
        }
        return changed;
    }
```

setFrame 方法用传进来的 l、t、r、b 这 4 个参数分别初始化 mLeft、mTop、mRight、mBottom这 4 个值，这样就确定了该 View 在父容器中的位置。在调用 setFrame 方法后，会调用 onLayout方法：

```java
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    }
```

onLayout 方法是一个空方法，这和 onMeasure 方法类似。确定位置时根据不同的控件有不同的实现，所以在 View 和 ViewGroup 中均没有实现 onLayout 方法。既然这样，我们下面就来查看 LinearLayout 的 onLayout 方法：

```java
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if (mOrientation == VERTICAL) {
            layoutVertical(l, t, r, b);
        } else {
            layoutHorizontal(l, t, r, b);
        }
    }
```

与 onMeasure 方法类似，根据方向来调用不同的方法。这里仍旧查看垂直方向的 layoutVertical 方法，如下所示：

```java
    void layoutVertical(int left, int top, int right, int bottom) {
...
        for (int i = 0; i < count; i++) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                childTop += measureNullChild(i);
            } else if (child.getVisibility() != GONE) {
                final int childWidth = child.getMeasuredWidth();
                final int childHeight = child.getMeasuredHeight();

                final LinearLayout.LayoutParams lp =
                        (LinearLayout.LayoutParams) child.getLayoutParams();

                int gravity = lp.gravity;
                if (gravity < 0) {
                    gravity = minorGravity;
                }
                final int layoutDirection = getLayoutDirection();
                final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
                switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
                    case Gravity.CENTER_HORIZONTAL:
                        childLeft = paddingLeft + ((childSpace - childWidth) / 2)
                                + lp.leftMargin - lp.rightMargin;
                        break;

                    case Gravity.RIGHT:
                        childLeft = childRight - childWidth - lp.rightMargin;
                        break;

                    case Gravity.LEFT:
                    default:
                        childLeft = paddingLeft + lp.leftMargin;
                        break;
                }

                if (hasDividerBeforeChildAt(i)) {
                    childTop += mDividerHeight;
                }

                childTop += lp.topMargin;
                setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                        childWidth, childHeight);
                childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

                i += getChildrenSkipCount(child, i);
            }
        }
    }
```

这个方法会遍历子元素并调用 setChildFrame 方法。其中 childTop 值是不断累加的，这样子元素才会依次按照垂直方向一个接一个排列下去而不会是重叠的。接着看 setChildFrame 方法：

```java
    private void setChildFrame(View child, int left, int top, int width, int height) {
        child.layout(left, top, left + width, top + height);
    }
```

在setChildFrame 方法中调用子元素的 layout 方法来确定自己的位置。

#### 3.4 View 的 draw 流程

View 的 draw 流程很简单，下面先来看看 View 的 draw 方法。

官方注释清楚地说明了每一步的做法，它们分别是：

（1）如果需要，则绘制背景。 

（2）保存当前 canvas 层。 

（3）绘制 View 的内容。 

（4）绘制子 View。 

（5）如果需要，则绘制 View 的褪色边缘，这类似于阴影效果。 

（6）绘制装饰，比如滚动条。

其中第 2 步和第 5 步可以跳过，所以这里不做分析，重点分析其他步骤。

##### 步骤 1：绘制背景

```java
    private void drawBackground(Canvas canvas) {
        final Drawable background = mBackground;
        if (background == null) {
            return;
        }

        setBackgroundBounds();
...
        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        // note 1、
        if ((scrollX | scrollY) == 0) {
            background.draw(canvas);
        } else {
            canvas.translate(scrollX, scrollY);
            background.draw(canvas);
            canvas.translate(-scrollX, -scrollY);
        }
    }
```

note 1、绘制背景考虑了偏移参数 scrollX 和 scrollY。如果有偏移值不为 0，则会在偏移后的 canvas 绘制背景。

##### 步骤 3：绘制 View 的内容

步骤 3 调用了 View 的 onDraw 方法。这个方法是一个空实现，因为不同的 View 有着不同的内容，这需要我们自己去实现，即在自定义 View 中重写该方法来实现：

```java
    protected void onDraw(Canvas canvas) {
    }
```

##### 步骤 4：绘制子 View

步骤 4 调用了 dispatchDraw 方法，这个方法也是一个空实现，如下所示：

```java
    protected void dispatchDraw(Canvas canvas) {

    }
```

ViewGroup 重写了这个方法，紧接着看看 ViewGroup 的 dispatchDraw 方法：

```java
    @Override
    protected void dispatchDraw(Canvas canvas) {
...
        for (int i = 0; i < childrenCount; i++) {
            while (transientIndex >= 0 && mTransientIndices.get(transientIndex) == i) {
                final View transientChild = mTransientViews.get(transientIndex);
                if ((transientChild.mViewFlags & VISIBILITY_MASK) == VISIBLE ||
                        transientChild.getAnimation() != null) {
                    more |= drawChild(canvas, transientChild, drawingTime);
                }
                transientIndex++;
                if (transientIndex >= transientCount) {
                    transientIndex = -1;
                }
            }
...
    }
```

源码很长，这里截取了关键的部分，在 dispatchDraw 方法中对子类 View 进行遍历，并调用 drawChild 方法：

```java
    protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        return child.draw(canvas, this, drawingTime);
    }
```

这里主要调用了 View 的 draw 方法，代码如下所示：

```java
    boolean draw(Canvas canvas, ViewGroup parent, long drawingTime) {
...
    // note 1、
        if (!drawingWithDrawingCache) {
            if (drawingWithRenderNode) {
                mPrivateFlags &= ~PFLAG_DIRTY_MASK;
                ((RecordingCanvas) canvas).drawRenderNode(renderNode);
            } else {
                // Fast path for layouts with no backgrounds
                if ((mPrivateFlags & PFLAG_SKIP_DRAW) == PFLAG_SKIP_DRAW) {
                    mPrivateFlags &= ~PFLAG_DIRTY_MASK;
                    dispatchDraw(canvas);
                } else {
                    draw(canvas);
                }
            }
        } else if (cache != null) {
            mPrivateFlags &= ~PFLAG_DIRTY_MASK;
            if (layerType == LAYER_TYPE_NONE || mLayerPaint == null) {
                // no layer paint, use temporary paint to draw bitmap
                Paint cachePaint = parent.mCachePaint;
                if (cachePaint == null) {
                    cachePaint = new Paint();
                    cachePaint.setDither(false);
                    parent.mCachePaint = cachePaint;
                }
                cachePaint.setAlpha((int) (alpha * 255));
                canvas.drawBitmap(cache, 0.0f, 0.0f, cachePaint);
            } else {
                // use layer paint to draw the bitmap, merging the two alphas, but also restore
                int layerPaintAlpha = mLayerPaint.getAlpha();
                if (alpha < 1) {
                    mLayerPaint.setAlpha((int) (alpha * layerPaintAlpha));
                }
                canvas.drawBitmap(cache, 0.0f, 0.0f, mLayerPaint);
                if (alpha < 1) {
                    mLayerPaint.setAlpha(layerPaintAlpha);
                }
            }
        }

        return more;
    }
```

源码很长，我们挑重点的看。在上面代码注释 1 处判断是否有缓存，如果没有则正常绘制，

如果有则利用缓存显示。

##### 步骤 6：绘制装饰

绘制装饰的方法为 View 的 onDrawForeground 方法：

```java
    public void onDrawForeground(Canvas canvas) {
        onDrawScrollIndicators(canvas);
        onDrawScrollBars(canvas);

        final Drawable foreground = mForegroundInfo != null ? mForegroundInfo.mDrawable : null;
        if (foreground != null) {
            if (mForegroundInfo.mBoundsChanged) {
                mForegroundInfo.mBoundsChanged = false;
                final Rect selfBounds = mForegroundInfo.mSelfBounds;
                final Rect overlayBounds = mForegroundInfo.mOverlayBounds;

                if (mForegroundInfo.mInsidePadding) {
                    selfBounds.set(0, 0, getWidth(), getHeight());
                } else {
                    selfBounds.set(getPaddingLeft(), getPaddingTop(),
                            getWidth() - getPaddingRight(), getHeight() - getPaddingBottom());
                }

                final int ld = getLayoutDirection();
                Gravity.apply(mForegroundInfo.mGravity, foreground.getIntrinsicWidth(),
                        foreground.getIntrinsicHeight(), selfBounds, overlayBounds, ld);
                foreground.setBounds(overlayBounds);
            }

            foreground.draw(canvas);
        }
    }
```

很明显这个方法用于绘制 ScrollBar 以及其他的装饰，并将它们绘制在视图内容的上层。

