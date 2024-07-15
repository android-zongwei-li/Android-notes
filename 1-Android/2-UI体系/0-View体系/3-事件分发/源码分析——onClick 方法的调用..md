##### 开始：

首先 debug 运行到 onClick 中，查看其调用栈，如下：

##### step1：实现 OnClickListener 接口，并把实例传给 View > mListenerInfo > mOnClickListener 持有。

首先来看以下 onClick() 方法在 View.java 中的定义：

View.java

```java
    public interface OnClickListener {
        void onClick(View v);
    }
    
	// 1、实现接口，通过此方法传入 View 中
    public void setOnClickListener(@Nullable OnClickListener l) {
        if (!isClickable()) {
            setClickable(true);
        }
        // 1.1、将 l 实例交由 mOnClickListener 持有
        // 1.2、mOnClickListener 是 mListenerInfo 中的一个变量
        getListenerInfo().mOnClickListener = l;
    }    

    static class ListenerInfo {
        public OnClickListener mOnClickListener;		
	}

    ListenerInfo getListenerInfo() {
        if (mListenerInfo != null) {
            return mListenerInfo;
        }
        mListenerInfo = new ListenerInfo();
        return mListenerInfo;
    }
```

分析：

1、我们实现了一个 OnClickListener 接口，并把实例通过 setOnClickListener() 方法传入 View 中。

1.1、我们要知道 View 中维护着一个变量 mListenerInfo，它的实例化只有一个地方，即 getListenerInfo() 中。

##### step2：View#performClick() --> onClick()

View.java

```java
    public boolean performClick() {
        // We still need to call this method to handle the cases where performClick() was called
        // externally, instead of through performClickInternal()
        notifyAutofillManagerOnClick();

        final boolean result;
        
        // 2.0、
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            
            // 2.1、
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

2.0、

2.1、执行 mOnClickListener 实例的 onCLick 方法。并且把当前对象即 View 自身作为参数传入 onClick 中。

##### step3：View#performClickInternal() --> performClick()

```java
    private boolean performClickInternal() {
        // Must notify autofill manager before performing the click actions to avoid scenarios where
        // the app has a click listener that changes the state of views the autofill service might
        // be interested on.
        notifyAutofillManagerOnClick();

        // 3.0
        return performClick();
    }
```

3.0、

##### step4：View$PerformClick#run() ---> performClickInternal()

```java
    private final class PerformClick implements Runnable {
        @Override
        public void run() {
            recordGestureClassification(TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__SINGLE_TAP);
            // 4.0
            performClickInternal();
        }
    }
```

4.0、现在重要的就是找到 PerformClick 执行的地方，怎么找呢？

首先可以看到 PerformClick 类是一个 private 的，因此之间 ctrl + F，找到它。

```java
	private PerformClick mPerformClick;
	
    public boolean onTouchEvent(MotionEvent event) {
...
	if (mPerformClick == null) {
    	mPerformClick = new PerformClick();
	}
	if (!post(mPerformClick)) {
    	performClickInternal();
	}
        ...
    }
```

##### step5：View#post()

```java
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().post(action);
        return true;
    }
```



ViewRootImpl.java

```java
    final ViewRootHandler mHandler = new ViewRootHandler();
    
	public ViewRootImpl(Context context, Display display, IWindowSession session,
            boolean useSfChoreographer) {
        // 
        mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, display, this, mHandler, this,
                context);        
    }

	private void performTraversals() {
        final View host = mView;
        
        
        host.dispatchAttachedToWindow(mAttachInfo, 0);
	}
```

