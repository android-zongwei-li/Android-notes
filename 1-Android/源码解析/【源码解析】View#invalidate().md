#### invalidate()源码分析

它在触发绘制时，不仅绘制自己的View，还会去绘制parent View。



核心类

> View.java
>
> ViewGroup.java
>
> ViewRootImpl.java
>
> Choreographer.java



View.java

```java
public void invalidate() {
    invalidate(true);
}

@UnsupportedAppUsage
public void invalidate(boolean invalidateCache) {
    invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
}

void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
        boolean fullInvalidate) {
    if (mGhostView != null) {
        mGhostView.invalidate(true);
        return;
    }

    if (skipInvalidate()) {
        return;
    }

    // Reset content capture caches
    // here1:& + ~ 操作会将对应的bit位置为0。
    mPrivateFlags4 &= ~PFLAG4_CONTENT_CAPTURE_IMPORTANCE_MASK;
    mContentCaptureSessionCached = false;

    if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)) == (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)
            || (invalidateCache && (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == PFLAG_DRAWING_CACHE_VALID)
            || (mPrivateFlags & PFLAG_INVALIDATED) != PFLAG_INVALIDATED
            || (fullInvalidate && isOpaque() != mLastIsOpaque)) {
        if (fullInvalidate) {
            mLastIsOpaque = isOpaque();
            mPrivateFlags &= ~PFLAG_DRAWN;
        }

        // here2:将mPrivateFlags中的PFLAG_DIRTY对应的bit位置为1。
        // | 运算：将对应位置1。类似于将一个字段设为true。
        mPrivateFlags |= PFLAG_DIRTY;

        if (invalidateCache) {
            mPrivateFlags |= PFLAG_INVALIDATED;
            mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
        }

        // Propagate the damage rectangle to the parent view.
        final AttachInfo ai = mAttachInfo;
        final ViewParent p = mParent;
        if (p != null && ai != null && l < r && t < b) {
            final Rect damage = ai.mTmpInvalRect;
            damage.set(l, t, r, b);
            p.invalidateChild(this, damage);
        }

        // Damage the entire projection receiver, if necessary.
        if (mBackground != null && mBackground.isProjected()) {
            final View receiver = getProjectionReceiver();
            if (receiver != null) {
                receiver.damageInParent();
            }
        }
    }
}
```

主要做了几件事：

1、mGhostView

看是否有mGhostView，有的话，调用mGhostView的invalidate方法。

2、skipInvalidate()

看是否满足skip的条件，满足就不往下执行了。

```java
private boolean skipInvalidate() {
    return (mViewFlags & VISIBILITY_MASK) != VISIBLE && mCurrentAnimation == null &&
            (!(mParent instanceof ViewGroup) ||
                    !((ViewGroup) mParent).isViewTransitioning(this));
}
```

可以看到有以下条件：

1. 是否visible，是否可见
2. 是否在执行动画
3. 是否在做转换，isViewTransitioning

```java
boolean isViewTransitioning(View view) {
    return (mTransitioningViews != null && mTransitioningViews.contains(view));
}
```

这个方法用来判断是否在view invisible or gone继续执行invalidate，因为当前可能正在做动画转换，因此不得不继续执行invalidate。因为invalidate意味着draw。

ViewGroup.java

```java
public final void invalidateChild(View child, final Rect dirty) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null && attachInfo.mHardwareAccelerated) {
        // HW accelerated fast path
        onDescendantInvalidated(child, child);
        return;
    }

    ViewParent parent = this;
    if (attachInfo != null) {
...
        do {
            View view = null;
            if (parent instanceof View) {
                view = (View) parent;
            }
...
            parent = parent.invalidateChildInParent(location, dirty);
...
        } while (parent != null);
    }
}
```

到这里有两条路可以走：

1. ```
   onDescendantInvalidated
   ```

   ```java
   public void onDescendantInvalidated(@NonNull View child, @NonNull View target) {
   ...
       if (mParent != null) {
           mParent.onDescendantInvalidated(this, target);
       }
   }
   ```

   调用到父类：ViewRootImpl.java

   ```java
   @Override
   public void onDescendantInvalidated(@NonNull View child, @NonNull View descendant) {
       invalidate();
   }
   
   @UnsupportedAppUsage
   void invalidate() {
       mDirty.set(0, 0, mWidth, mHeight);
       if (!mWillDrawSoon) {
           scheduleTraversals();
       }
   }
   ```

2. ```
   parent = parent.invalidateChildInParent(location, dirty);
   ```

   ```java
   @Override
   public void invalidateChild(View child, Rect dirty) {
       invalidateChildInParent(null, dirty);
   }
   
   @Override
   public ViewParent invalidateChildInParent(int[] location, Rect dirty) {
       checkThread();
   ...
       invalidateRectOnScreen(dirty);
   
       return null;
   }
   
   private void invalidateRectOnScreen(Rect dirty) {
   ...
       if (!mWillDrawSoon && (intersected || mIsAnimating)) {
           scheduleTraversals();
       }
   }
   
   @UnsupportedAppUsage
   void scheduleTraversals() {
       if (!mTraversalScheduled) {
           mTraversalScheduled = true;
           mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
           // here1：将一个runnable（TraversalRunnable）给post出去
           mChoreographer.postCallback(
                   Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
           notifyRendererOfFramePending();
           pokeDrawLockIfNeeded();
       }
   }
   
       final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
   
       final class TraversalRunnable implements Runnable {
           @Override
           public void run() {
               doTraversal();
           }
       }
   ```

可以看到，不管哪种情况，最终都会调用`scheduleTraversals()`方法。

从`here1`就来到了`Choreographer.java` 编导类，它负责将这些事件msg按执行的时间进行排序。

```java
public void postCallback(int callbackType, Runnable action, Object token) {
    // here1：可以看到delayMillis为0
    postCallbackDelayed(callbackType, action, token, 0);
}

public void postCallbackDelayed(int callbackType,
        Runnable action, Object token, long delayMillis) {
    postCallbackDelayedInternal(callbackType, action, token, delayMillis);
}

private void postCallbackDelayedInternal(int callbackType,
        Object action, Object token, long delayMillis) {
    synchronized (mLock) {
        final long now = SystemClock.uptimeMillis();
        final long dueTime = now + delayMillis;
        // here2:这个action就是runnable，现在被传给了CallbackQueue，它是负责排序的。
        mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

        if (dueTime <= now) {
            // here3：因此走这里
            scheduleFrameLocked(now);
        } else {
            Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
            msg.arg1 = callbackType;
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, dueTime);
        }
    }
}
```

根据here1、here2、here3可知，走的是下面这个方法：

```java
private void scheduleFrameLocked(long now) {
    if (!mFrameScheduled) {
        mFrameScheduled = true;
        if (USE_VSYNC) {
            if (isRunningOnLooperThreadLocked()) {
                scheduleVsyncLocked();
            } else {
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtFrontOfQueue(msg);
            }
        } else {
            final long nextFrameTime = Math.max(
                    mLastFrameTimeNanos / TimeUtils.NANOS_PER_MS + sFrameDelay, now);
            // here1：这里
            Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, nextFrameTime);
        }
    }
}
```

看here1处，这里的判断条件：`if (USE_VSYNC)`我还不清楚为什么要是FALSE，我通过倒退，要走后面的draw方法，就得先发送：MSG_DO_FRAME消息。

```java
private final class FrameHandler extends Handler {
    public FrameHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSG_DO_FRAME:
                doFrame(System.nanoTime(), 0);
                break;
            case MSG_DO_SCHEDULE_VSYNC:
                doScheduleVsync();
                break;
            case MSG_DO_SCHEDULE_CALLBACK:
                doScheduleCallback(msg.arg1);
                break;
        }
    }
}

    @UnsupportedAppUsage
    void doFrame(long frameTimeNanos, int frame) {
...
            // here1：在这里
            doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
...
    }

void doCallbacks(int callbackType, long frameTimeNanos) {
...
        // here1：这里
        callbacks = mCallbackQueues[callbackType].extractDueCallbacksLocked(
                now / TimeUtils.NANOS_PER_MS);
...
            for (CallbackRecord c = callbacks; c != null; c = c.next) {
                c.run(frameTimeNanos);
            }
...
}

public CallbackRecord extractDueCallbacksLocked(long now) {
    CallbackRecord callbacks = mHead;
    if (callbacks == null || callbacks.dueTime > now) {
        return null;
    }

    CallbackRecord last = callbacks;
    CallbackRecord next = last.next;
    while (next != null) {
        if (next.dueTime > now) {
            last.next = null;
            break;
        }
        last = next;
        next = next.next;
    }
    mHead = next;
    return callbacks;
}
```

下面就开始执行这个Runnable的run方法了：

```java
final class TraversalRunnable implements Runnable {
    @Override
    public void run() {
        doTraversal();
    }
}

void doTraversal() {
    if (mTraversalScheduled) {
        mTraversalScheduled = false;
...
        performTraversals();
...
    }
}

private void performTraversals() {
    ...
        if (!mStopped || mReportNextDraw) {
            boolean focusChangedDueToTouchMode = ensureTouchModeLocally(
...
                 // Ask host how big it wants to be
                performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
...
                if (measureAgain) {
                    if (DEBUG_LAYOUT) Log.v(mTag,
                            "And hey let's measure once more: width=" + width
                            + " height=" + height);
                    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
                }
            }
        }
    } 
...

    if (didLayout) {
        performLayout(lp, mWidth, mHeight);
...
    }
...

    if (!cancelDraw) {
        performDraw();
    }
...
}
```

依次执行了这三个方法：

`performMeasure()`、`performLayout()`、`performDraw()`

这几个方法最终会调用到 View 的 `onMeasure()`、`onLayout()`、`onDraw()`方法。

下面看一下这几组方法：

ViewRootImpl.java

```java
// Q1：mView是什么？    
View mView;

private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
...
        mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
...
}

    private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
            int desiredWindowHeight) {
...
        final View host = mView;
        if (host == null) {
            return;
        }
...
            host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
...
    }

    private void performDraw() {
...
            boolean canUseAsync = draw(fullRedrawNeeded);
...
    }

    private boolean draw(boolean fullRedrawNeeded) {
...
                if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset,
                        scalingRequired, dirty, surfaceInsets)) {
                    return false;
                }
...
    }

    private boolean drawSoftware(Surface surface, AttachInfo attachInfo, int xoff, int yoff,
            boolean scalingRequired, Rect dirty, Rect surfaceInsets) {

        // Draw with software renderer.
        final Canvas canvas;
...
            mView.draw(canvas);
...
    }
```

View.java

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
...
            onMeasure(widthMeasureSpec, heightMeasureSpec);
...
}

    public void layout(int l, int t, int r, int b) {
...
            onLayout(changed, l, t, r, b);
...
    }

    public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         *      7. If necessary, draw the default focus highlight
         */

        // Step 1, draw the background, if needed
        int saveCount;

        drawBackground(canvas);

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            drawAutofilledHighlight(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // Step 7, draw the default focus highlight
            drawDefaultFocusHighlight(canvas);

            if (isShowingLayoutBounds()) {
                debugDrawFocus(canvas);
            }

            // we're done...
            return;
        }

        // Step 2, save the canvas' layers

        // Step 3, draw the content
        onDraw(canvas);

        // Step 4, draw the children
        dispatchDraw(canvas);

        // Step 5, draw the fade effect and restore layers

        // Step 6, draw decorations (foreground, scrollbars)
        onDrawForeground(canvas);

        // Step 7, draw the default focus highlight
        drawDefaultFocusHighlight(canvas);

        if (isShowingLayoutBounds()) {
            debugDrawFocus(canvas);
        }
    }
```

