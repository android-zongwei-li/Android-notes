在讲 View 的事件分发机制之前要首先了解一下 Activity 的组成，然后从源码的角度来分析 View 的事件分发机制。

### 1、源码解析 Activity 的构成

点击事件用 MotionEvent 来表示，当一个点击事件产生后，事件最先传递给 Activity，所以我们首先要了解一下 Activity 的构成。当我们写 Activity 时会调用 setContentView()方法来加载布局。现在来看看 setContentView()方法是怎么实现的，代码如下所示：

```java
    public void setContentView(View view) {
        getWindow().setContentView(view);
        initWindowDecorActionBar();
    }
```

这里调用了 getWindow().setContentView(layoutResID)，getWindow()指的是什么呢？接着往下看，getWindow()返回 mWindow：

```java
    public Window getWindow() {
        return mWindow;
    }
    
    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

        mWindow = new PhoneWindow(this, window, activityConfigCallback);
...
    }
    
```

而 getWindow()又指的是 PhoneWindow，因此 setContentView()最终调用的是 PhoneWindow的。接下来继续看看 PhoneWindow 的 setContentView()方法，代码如下所示：

```java
    @Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            // 1、
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```

挑关键的接着看，上面代码注释 1 处 installDecor()方法里面做了什么，代码如下所示：

```java
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            // 1、
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
        }
        ...
    }
```

紧接着查看上面代码注释 1 处的 generateDecor 方法里做了什么：

```java
    protected DecorView generateDecor(int featureId) {
...
        return new DecorView(context, featureId, this, getAttributes());
    }
```

这里创建了一个 DecorView，这个DecorView就是 Activity中的根 View。接着查看 DecorView 的源码，它继承自 FrameLayout。我们再回到 installDecor()方法中，查看 generateLayout(mDecor)做了什么：

```java
    protected ViewGroup generateLayout(DecorView decor) {
...
        // Inflate the window decor.
    	// 根据不同的情况加载不同的布局给 layoutResource
        int layoutResource;
        int features = getLocalFeatures();
        // System.out.println("Features: 0x" + Integer.toHexString(features));
        if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleIconsDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_title_icons;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
            // System.out.println("Title Icons!");
        } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
                && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
            // Special case for a window with only a progress bar (and title).
            // XXX Need to have a no-title version of embedded windows.
            layoutResource = R.layout.screen_progress;
            // System.out.println("Progress!");
        } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
            // Special case for a window with a custom title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogCustomTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {
                layoutResource = R.layout.screen_custom_title;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
        } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
            // If no other features and not embedded, only need a title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
                layoutResource = a.getResourceId(
                        R.styleable.Window_windowActionBarFullscreenDecorLayout,
                        R.layout.screen_action_bar);
            } else {
                // 注释 1、
                layoutResource = R.layout.screen_title;
            }
            // System.out.println("Title!");
        } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
            layoutResource = R.layout.screen_simple_overlay_action_mode;
        } else {
            // Embedded, so no decoration is needed.
            layoutResource = R.layout.screen_simple;
            // System.out.println("Simple!");
        }
...
        return contentParent;
    }
```

PhoneWindow 的 generateLayout()方法比较长，这里只截取了一小部分关键的代码，其主要内容就是根据不同的情况加载不同的布局给 layoutResource。现在查看上面代码注释 1 处的布局R.layout.screen_title，这个文件在 frameworks，它的代码如下所示：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:fitsSystemWindows="true">
    <!-- Popout bar for action modes -->
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
        android:layout_width="match_parent" 
        android:layout_height="?android:attr/windowTitleSize"
        style="?android:attr/windowTitleBackgroundStyle">
        <TextView android:id="@android:id/title" 
            style="?android:attr/windowTitleStyle"
            android:background="@null"
            android:fadingEdge="horizontal"
            android:gravity="center_vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </FrameLayout>
    <FrameLayout android:id="@android:id/content"
        android:layout_width="match_parent" 
        android:layout_height="0dip"
        android:layout_weight="1"
        android:foregroundGravity="fill_horizontal|top"
        android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```

上面的 ViewStub 是用来显示 Actionbar 的。下面的两个 FrameLayout：一个是 title，用来显示标题；另一个是 content，用来显示内容。看到上面的源码，大家就知道了一个 Activity 包含一个 Window 对象，这个对象是由 PhoneWindow 来实现的。PhoneWindow 将 DecorView 作为整个应用窗口的根 View，而这个 DecorView 又将屏幕划分为两个区域：一个是 TitleView，另一个是 ContentView，而我们平常做应用所写的布局正是展示在 ContentView 中的，如图所示。

![image-20210420162341247](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210420162341247.png)

### 2、源码解析 View 的事件分发机制

当我们点击屏幕时，就产生了点击事件，这个事件被封装成了一个类：MotionEvent。而当这个 MotionEvent 产生后，那么系统就会将这个 MotionEvent 传递给 View 的层级，MotionEvent 在 View 中的层级传递过程就是点击事件分发。在了解了什么是事件分发后，我们还需要了解事件分发的 3 个重要方法，它们分别是：

• dispatchTouchEvent(MotionEvent ev)：用来进行事件的分发。

• onInterceptTouchEvent(MotionEvent ev)：用来进行事件的拦截，在 dispatchTouchEvent()中调用，需要注意的是 View 没有提供该方法。

• onTouchEvent(MotionEvent ev)：用来处理点击事件，在 dispatchTouchEvent()方法中进行调用。

1. ##### View 的事件分发机制

   当点击事件产生后，事件首先会传递给当前的 Activity ， 这会调用 Activity 的dispatchTouchEvent()方法，当然具体的事件处理工作都是交由 Activity 中的 PhoneWindow 来完成的，然后 PhoneWindow 再把事件处理工作交给 DecorView，之后再由 DecorView 将事件处理工作交给根 ViewGroup。所以，我们从 ViewGroup 的 dispatchTouchEvent()方法开始分析，代码如下所示：

   ```java
       @Override
       public boolean dispatchTouchEvent(MotionEvent ev) {
   ...
           boolean handled = false;
           if (onFilterTouchEventForSecurity(ev)) {
               final int action = ev.getAction();
               final int actionMasked = action & MotionEvent.ACTION_MASK;
   
               // Handle an initial down.
               if (actionMasked == MotionEvent.ACTION_DOWN) {
                   cancelAndClearTouchTargets(ev);
                   resetTouchState();
               }
   
               // Check for interception.
               final boolean intercepted;
               // 注释 1、
               if (actionMasked == MotionEvent.ACTION_DOWN
                       || mFirstTouchTarget != null) {
                   //注释 2、
                   final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                   if (!disallowIntercept) {
                       intercepted = onInterceptTouchEvent(ev);
                       ev.setAction(action); 
                   } else {
                       intercepted = false;
                   }
               } else {
                   intercepted = true;
               }
   ...
           return handled;
       }
   ```

   首先判断事件是否为 DOWN 事件，如果是，则进行初始化，resetTouchState 方法中会把 mFirstTouchTarget 的值置为 null。这里为什么要进行初始化呢？原因就是一个完整的事件序列是以 DOWN 开始，以 UP 结束的。所以如果是 DOWN 事件，那么说明这是一个新的事件序列，故而需要初始化之前的状态。

   接着往下看，上面代码注释 1 处的条件如果满足，则执行下面的句子，mFirstTouchTarget 的意义是：当前 ViewGroup 是否拦截了事件，如果拦截了，mFirstTouchTarget=null；如果没有拦截并交由子 View 来处理，则 mFirstTouchTarget != null。假设当前的 ViewGroup 拦截了此事件，mFirstTouchTarget != null 则为 false，如果这时触发 ACTION_DOWN 事件 ，则会执行 onInterceptTouchEvent(ev) 方 法 ； 如果触发的是 ACTION_MOVE、ACTION_UP 事件，则不再执行 onInterceptTouchEvent(ev)方法，而是直接设置 intercepted = true，此后的一个事件序列均由这个 ViewGroup 处理。

   再往下看，上面代码注释 2 处又出现了一个 FLAG_DISALLOW_INTERCEPT 标志位，它主要是禁止 ViewGroup 拦截除了 DOWN 之外的事件，一般通过子 View 的 requestDisallowInterceptTouchEvent 来设置。

   所以总结一下就是，当 ViewGroup 要拦截事件的时候，那么后续的事件序列都将交给它处理，而不用再调用 onInterceptTouchEvent()方法了。所以，onInterceptTouchEvent()方法并不是每次事件都会调用的。 接下来查看 onInterceptTouchEvent()方法：

   ```java
       public boolean onInterceptTouchEvent(MotionEvent ev) {
           // 注释 1、
           if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
                   && ev.getAction() == MotionEvent.ACTION_DOWN
                   && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
                   && isOnScrollbarThumb(ev.getX(), ev.getY())) {
               return true;
           }
           return false;
       }
   ```

   默认情况下，onInterceptTouchEvent()只在满足注释 1 下的条件时，才会返回 true；其他情况都为 false，不进行拦截。如果想要让 ViewGroup 拦截事件，那么应该在自定义的 ViewGroup 中重写这个方法。接着来看看 dispatchTouchEvent()方法剩余的部分源码，如下所示：

   ```java
       public boolean dispatchTouchEvent(MotionEvent ev) {
       ...
       final View[] children = mChildren;
       // 注释 1、
       for (int i = childrenCount - 1; i >= 0; i--) {
           final int childIndex = getAndVerifyPreorderedIndex(
                   childrenCount, i, customOrder);
           final View child = getAndVerifyPreorderedView(
                   preorderedList, children, childIndex);
           // 注释 2、
           if (!child.canReceivePointerEvents()
                   || !isTransformedTouchPointInView(x, y, child, null)) {
               continue;
           }
           newTouchTarget = getTouchTarget(child);
           if (newTouchTarget != null) {
               newTouchTarget.pointerIdBits |= idBitsToAssign;
               break;
           }
           resetCancelNextUpFlag(child);
           // 注释 3、
           if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
               mLastTouchDownTime = ev.getDownTime();
               if (preorderedList != null) {
                   for (int j = 0; j < childrenCount; j++) {
                       if (children[childIndex] == mChildren[j]) {
                           mLastTouchDownIndex = j;
                           break;
                       }
                   }
               } else {
                   mLastTouchDownIndex = childIndex;
               }
               mLastTouchDownX = ev.getX();
               mLastTouchDownY = ev.getY();
               newTouchTarget = addTouchTarget(child, idBitsToAssign);
               alreadyDispatchedToNewTouchTarget = true;
               break;
           }
           // The accessibility focus didn't handle the event, so clear
           // the flag and do a normal dispatch to all children.
           ev.setTargetAccessibilityFocus(false);
       }
       ...
       }
   ```

   在上面代码注释 1 处我们看到了 for 循环。首先遍历 ViewGroup 的子元素，判断子元素是否能够接收到点击事件，如果子元素能够接收到点击事件，则交由子元素来处理。需要注意这个 for 循环是倒序遍历的，即从最上层的子 View 开始往内层遍历。接着往下看注释 2 处的代码，其意思是判断触摸点位置是否在子 View 的范围内或者???(还不懂)。如果均不符合则执行 continue 语句，表示这个子 View 不符合条件，开始遍历下一个子 View。接下来查看注释 3 处的 dispatchTransformedTouchEvent 方法做了什么，代码如下所示：

   ```java
       private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
               View child, int desiredPointerIdBits) {
           final boolean handled;
   
           final int oldAction = event.getAction();
           if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
               event.setAction(MotionEvent.ACTION_CANCEL);
               if (child == null) {
                   handled = super.dispatchTouchEvent(event);
               } else {
                   handled = child.dispatchTouchEvent(event);
               }
               event.setAction(oldAction);
               return handled;
           }
   ...
       }
   ```

   如果有子 View，则调用子 View 的 dispatchTouchEvent(event)方法。如果 ViewGroup 没有子View，则调用 super.dispatchTouchEvent(event)方法。ViewGroup 是继承 View 的，再来查看 View 的 dispatchTouchEvent(event)：

   ```java
       public boolean dispatchTouchEvent(MotionEvent event) {
   ...
           boolean result = false;
           if (onFilterTouchEventForSecurity(event)) {
               //noinspection SimplifiableIfStatement
               ListenerInfo li = mListenerInfo;
               if (li != null && li.mOnTouchListener != null
                       && (mViewFlags & ENABLED_MASK) == ENABLED
                       && li.mOnTouchListener.onTouch(this, event)) {
                   result = true;
               }
   
               if (!result && onTouchEvent(event)) {
                   result = true;
               }
           }
           return result;
       }
   ```

   我们看到如果 OnTouchListener 不为 null 并且 onTouch 方法返回 true，则表示事件被消费，就不会执行 onTouchEvent(event)；否则就会执行 onTouchEvent(event)。可以看出 OnTouchListener 中的 onTouch()方法优先级要高于 onTouchEvent(event)方法。下面再来看看 onTouchEvent()方法的部分源码：

   ```java
       public boolean onTouchEvent(MotionEvent event) {
           final int action = event.getAction();
   
           final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                   || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                   || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
   
           if ((viewFlags & ENABLED_MASK) == DISABLED) {
               if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                   setPressed(false);
               }
               mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
               // A disabled view that is clickable still consumes the touch
               // events, it just doesn't respond to them.
               return clickable;
           }
           if (mTouchDelegate != null) {
               if (mTouchDelegate.onTouchEvent(event)) {
                   return true;
               }
           }
   
           if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
               switch (action) {
                   case MotionEvent.ACTION_UP:
   ...
                       boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                       if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                           boolean focusTaken = false;
                           if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                               focusTaken = requestFocus();
                           }
   
                           if (prepressed) {
                               setPressed(true, x, y);
                           }
   
                           if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                               removeLongPressCallback();
                               if (!focusTaken) {
                                   if (mPerformClick == null) {
                                       mPerformClick = new PerformClick();
                                   }
                                   if (!post(mPerformClick)) {
                                       performClickInternal();
                                   }
                               }
                           }
               return true;
           }
           return false;
       }
   ```

   从上面的代码可以看到，只要 View 的 CLICKABLE 和 LONG_CLICKABLE 有一个为 true，那么onTouchEvent()就会返回true消耗这个事件。CLICKABLE和LONG_CLICKABLE代表 View 可以被点击和长按点击，可以通过 View 的 setClickable 和 setLongClickable 方法来设置，也可以通过 View 的 setOnClickListenter 和 setOnLongClickListener 来设置，它们会自动将 View 设置为 CLICKABLE 和 LONG_CLICKABLE。 接着在 ACTION_UP 事件中会调用 performClickInternal()方法：

   ```java
       private boolean performClickInternal() {
           notifyAutofillManagerOnClick();
   
           return performClick();
       }
       
       public boolean performClick() {
           notifyAutofillManagerOnClick();
   
           final boolean result;
           final ListenerInfo li = mListenerInfo;
           // 注释 1、
           if (li != null && li.mOnClickListener != null) {
               playSoundEffect(SoundEffectConstants.CLICK);
               li.mOnClickListener.onClick(this);
               result = true;
           } else {
               result = false;
           }
   
           sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
   
           notifyEnterOrExitForAutoFillIfNeeded(true);
   
           return result;
       }
   ```

   从上面代码注释 1 处可以看出，如果 View 设置了点击事件 OnClickListener，那么它的onClick()方法就会被执行。View 事件分发机制的源码分析就讲到这里了，接下来介绍点击事件分发的传递规则。

2. ##### 点击事件分发的传递规则

   由前面事件分发机制的源码分析可知点击事件分发的这 3 个重要方法的关系，用伪代码来简单表示：

   ```java
   public boolean dispatchTouchEvent(MotionEvent ev) {
   boolean result=false；
   if(onInterceptTouchEvent(ev)){
    result=super.onTouchEvent(ev)；
    }else{
    result=child.dispatchTouchEvent(ev)；
   }
   return result；
   ```

   onInterceptTouchEvent 方法和 onTouchEvent 方法都在 dispatchTouchEvent 方法中调用。现在我们根据这段伪代码来分析一下点击事件分发的传递规则。

   首先讲一下点击事件由上而下的传递规则，当点击事件产生后会由 Activity 来处理，传递给 PhoneWindow，再传递给 DecorView，最后传递给顶层的 ViewGroup。一般在事件传递中只考虑 ViewGroup 的 onInterceptTouchEvent 方法，因为一般情况下我们不会重写 dispatchTouchEvent()方法。对于根 ViewGroup，点击事件首先传递给它的 dispatchTouchEvent()方法，如果该 ViewGroup 的 onInterceptTouchEvent()方法返回 true，则表示它要拦截这个事件，这个事件就会交给它的 onTouchEvent()方法处理；如果 onInterceptTouchEvent()方法返回 false，则表示它不拦截这个事件，则这个事件会交给它的子元素的 dispatchTouchEvent()来处理，如此反复下去。如果传递给底层的 View，View 是没有子 View 的，就会调用 View 的 dispatchTouchEvent()方法，一般情况下最终会调用 View 的 onTouchEvent()方法。

   事件由上而下传递返回值的规则如下：为 true，则拦截，不继续向下传递；为 false，则不拦截，继续向下传递。

   点击事件由下而上的传递。当点击事件传给底层的 View 时，如果其 onTouchEvent()方法返回 true，则事件由底层的 View 消耗并处理；如果返回 false 则表示该 View不做处理，则传递给父 View 的 onTouchEvent()处理；如果父 View 的 onTouchEvent()仍旧返回 false，则继续传递给该父 View 的父 View 处理，如此反复下去。

   事件由下而上传递返回值的规则如下：为 true，则处理了，不继续向上传递；为 false，则不处理，继续向上传递。





参考：

1、《Android进阶之光》

