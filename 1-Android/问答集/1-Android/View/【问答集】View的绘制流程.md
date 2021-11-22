#### 一、绘制流程

<font color='orange'>Q：View的绘制流程？</font>

主要就是讲一下measure、layout、draw分别干了什么。然后MeasureSpec发挥的作用。

见 **【总结篇】View的绘制流程.md**。

<font color='orange'>Q：View的第一次绘制是怎么调到的？</font>

measure,layout和draw.View的绘制流程是在Window添加过程中，ViewRootImpl类的setView方法开始的。

```java
ActivityThread.java
	@Override
    public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
            String reason) {
        final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);
        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();

            if (a.mVisibleFromClient) {
                if (!a.mWindowAdded) {
                    a.mWindowAdded = true;
                    wm.addView(decor, l);
                }
            }
        }        
    }
WindowManagerImpl.java
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();    
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                mContext.getUserId());
    }
WindowManagerGlobal.java
    public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
        ViewRootImpl root;
    
            root = new ViewRootImpl(view.getContext(), display);
            view.setLayoutParams(wparams);
    
            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);
    
            try {
                root.setView(view, wparams, panelParentView, userId);
            } 
        }
    }
```

是在handleResumeActivity()中通过wm.addView()触发的。最终调用到ViewRootImpl.setView()。

```java
    /**
     * We have one child
     */
    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
        synchronized (this) {
                // Schedule the first layout -before- adding to the window
                // manager, to make sure we do the relayout before receiving
                // any other events from the system.
                requestLayout();            
        }
    }       

    @Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }

    @UnsupportedAppUsage
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
```



<font color='orange'>Q：第一次绘制的消息是怎么发出来的？</font>



<font color='orange'>Q：View的后续绘制是怎么调用的？</font>



<font color='orange'>Q：后边的绘制消息是怎么循环的？</font>



<font color='orange'>Q：View的onMeasure，onLayout，onDraw都分别用来干什么，除了上面三个，还有哪些关键的方法？</font>



<font color='orange'>Q：onMeasure中不处理wrap_content，就会出现match_parent一样的效果的原因</font>

在onMeasure的过程中，在生成MS的时候，wrap_content会生成一个AT_MOST+size的MS，match_parent会生成一个EXACTLY+size的MS。在View默认的onMeasure方法中，对AT_MOST和EXACTLY的处理是一样的，都是使用的specSize，因此效果会一样。

<font color='orange'>Q：View的getwidth()和getMesuredWidth()有啥区别?</font>

getWidth()获取的是View当前真实的宽度，getMesuredWidth()获取的是当前View测量的宽度。getWidth()的值实在onLayout时赋值的，getMesuredWidth()的值是在完成onMeasure时赋值的。它们的大小不一定相等。为什么？这个还需要深入学一下。

<font color='orange'>Q：Measure、Layout、draw的流程、绘制顺序，基于这些说下TagLayout(FlowLayout)怎么写？</font>



<font color='orange'>Q：一个View被添加后第一个回调的方法是哪个?</font>

在ViewRootImpl中，当DecorView第一次添加时，会回调onAttachedToWindow(）。

```java
ViewRootImpl.java
private void performTraversals() {
。。。
if (mFirst) {
            host.dispatchAttachedToWindow(mAttachInfo, 0);
}
}
```

在ViewGroup中，会先回调View的onAttachedToWindow方法，然后回调onViewAdded()方法。

```java
    public void addView(View child, int index, LayoutParams params) {
...
        requestLayout();
        invalidate(true);
        addViewInner(child, index, params, false);
    }

    private void addViewInner(View child, int index, LayoutParams params,
            boolean preventRequestLayout) {
        if (ai != null && (mGroupFlags & FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW) == 0) {
            boolean lastKeepOn = ai.mKeepScreenOn;
            ai.mKeepScreenOn = false;
            // 这个最终会回调 onAttachedToWindow
            child.dispatchAttachedToWindow(mAttachInfo, (mViewFlags&VISIBILITY_MASK));

        }
        
		// 回调3
        dispatchViewAdded(child);
    }

    void dispatchAttachedToWindow(AttachInfo info, int visibility) {
        // 回调1
        onAttachedToWindow();

        // 回调2
        onVisibilityChanged(this, visibility);

        if ((mPrivateFlags&PFLAG_DRAWABLE_STATE_DIRTY) != 0) {
            // If nobody has evaluated the drawable state yet, then do it now.
            refreshDrawableState();
        }
        needGlobalAttributesUpdate(false);

        notifyEnterOrExitForAutoFillIfNeeded(true);
        notifyAppearedOrDisappearedForContentCaptureIfNeeded(true);
    }
```



<font color='orange'>Q：View的绘制原理？</font>



<font color='orange'>Q：View.inflater过程与异步inflater（东方头条）</font>



<font color='orange'>Q：inflater为什么比自定义View慢？（东方头条）</font>



<font color='orange'>Q：onMeasure怎么测量的？MeasureSpec的三种模式？</font>

以FrameLayout为例，由父View发起，先为子View计算一个MS，再测子View，然后再测父View，最后父View测量完成后，再对match_parent类型的子View再进行一次测量。

三种模式：EXACTLY、AT_MOST、UNSPECIFIED。首先在为子View计算MS的时候会使用到，具体是结合子View的LayoutParams（布局参数）来计算的，MS的结果一共存在9种情况。

<font color='orange'>Q：View的绘制流程，MeasureSpec(MS)知道吗？如何确定一个View的MS？那DecorView呢？</font>

MS是由父View为子View计算出来的一个测量约束。具体的计算是通过getChildMeasureSpec()方法，一共有9中情况。细细说一下。

<font color='orange'>Q：LayoutInflater#inflate的attrachToParenttrue是什么意思？</font>



<font color='orange'>Q：View的渲染过程？</font>



<font color='orange'>Q：View的刷新机制？</font>



<font color='orange'>Q：Android中View显示到设备上的流程。</font>



<font color='orange'>Q：Android渲染和刷新原理？</font>



<font color='orange'>Q：讲讲页面的刷新机制，GPU调试工具几个颜色值分别代码什么？</font>



<font color='orange'>Q：在oncreate里面可以得到View的宽高吗?</font>

不能。View是在handleResumeActivity()执行时，先执行onResume，然后才执行View的第一次绘制流程，因此在此之前都不能获取到View的真实宽高。

<font color='orange'>Q：在onResume中可以测量宽高么？</font>

不能。



#### 二、invalidate、postInvalidate、requestLayout

<font color='orange'>Q：invalidate的原理？</font>



<font color='orange'>Q：invalidate()和postInvalicate()区别</font>



<font color='orange'>Q：invalidate和requestlayout的区别</font>



<font color='orange'>Q：requestlayout的作用范围是多大</font>



<font color='orange'>Q：invalidate、postInvalidate、requestLayout的区别？</font>



<font color='orange'>Q：自定义View执行invalidate()方法,为什么有时候不会回调onDraw()？</font>

