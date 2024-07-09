[TOC]





invalidate、postInvalicate、requestLayout的作用

invalidate 通常都是用来触发 onDraw() 执行的，即重绘。可以在 UI 线程调用，不能在子线程调用。

适用场景



# invalidate() 原理

invalidate()方法的执行可以分为三步，修改（添加、移除）标志位，计算dirty区域并交给父View，最终触发绘制。



## ViewRootImpl.performTraversals

requestLayout 和 invalidate 方法最后都会调用到 ViewRootImpl.performTraversals 方法开始 View 的更新，因此先看一下这个方法的部分代码：

```java
private void performTraversals() {
    // DecorView 实例
    final View host = mView;
    boolean layoutRequested = mLayoutRequested && (!mStopped || mReportNextDraw);
    if (layoutRequested) {
        // 此处可能会调用 performMeasure 方法
        windowSizeMayChange |= measureHierarchy(host, lp, res, desiredWindowWidth, desiredWindowHeight);
    }
    if (mFirst || windowShouldResize || insetsChanged || viewVisibilityChanged || params != null || mForceNextWindowRelayout) {
        // ...
        if (focusChangedDueToTouchMode || mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight() || contentInsetsChanged || updatedConfiguration) {
            int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
            int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
          
          // ... measure ...
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
            // 如果 lp.horizontalWeight 或 lp.verticalWeight 大于 0 则重新调用 performMeasure 测量
            // ...
            layoutRequested = true;
        }
    }
    final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
    if (didLayout) {
      // ... layout ...
        performLayout(lp, mWidth, mHeight);
    }
    boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() || !isViewVisible;
    if (!cancelDraw && !newSurface) {
      // ... draw ...
        performDraw();
    }
}
```

这个方法很长，抽取关键部分可以大致明白 performTraversals 里会根据一些判断条件来执行 View 的 Measure, Layout, Draw 三大流程。



## View.invalidate

```java
public void invalidate() {
    invalidate(true);
}

public void invalidate(boolean invalidateCache) {
    invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
}

void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache, boolean fullInvalidate) {
  // 判断是否需要跳过 invalidate
    if (skipInvalidate()) { 
        return;
    }

  // 判断是否需要重绘，如果满足以下任意一个条件则进行重绘：
	// View 已经绘制完成且具有边界
	// invalidateCache == true 且设置了 PFLAG_DRAWING_CACHE_VALID 标志位，即绘制缓存可用
	// 没有设置 PFLAG_INVALIDATED 标志位，即没有被重绘过
	// fullInvalidate == true 且在 透明 和 不透明 之间发生了变化
    if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)) == (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)
            || (invalidateCache && (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == PFLAG_DRAWING_CACHE_VALID)
            || (mPrivateFlags & PFLAG_INVALIDATED) != PFLAG_INVALIDATED
            || (fullInvalidate && isOpaque() != mLastIsOpaque)) { // 判断是否重绘
        if (fullInvalidate) {
            mLastIsOpaque = isOpaque(); // 重新设置 Opaque
            mPrivateFlags &= ~PFLAG_DRAWN; // 移除 PFLAG_DRAWN 标志位
        }
        mPrivateFlags |= PFLAG_DIRTY; // 设置 PFLAG_DIRTY 脏区标志位

        if (invalidateCache) {
            mPrivateFlags |= PFLAG_INVALIDATED; // 设置 PFLAG_INVALIDATED 标志位
            mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID; // 移除 PFLAG_DRAWING_CACHE_VALID 标志位
        }

        final AttachInfo ai = mAttachInfo;
        final ViewParent p = mParent;
        if (p != null && ai != null && l < r && t < b) {
            // damage 表示要重绘的脏区
            final Rect damage = ai.mTmpInvalRect;
            damage.set(l, t, r, b);
            p.invalidateChild(this, damage);
        }
        // ...
    }
}

// 判断是否要跳过 invalidate 过程，如果同时满足以下条件则跳过：
// 1、View 不可见
// 2、当前没有运行动画
// 3、父 View 不是 ViewGroup 类型或者父 ViewGoup 不处于过渡态
private boolean skipInvalidate() {
    return (mViewFlags & VISIBILITY_MASK) != VISIBLE && mCurrentAnimation == null &&
            (!(mParent instanceof ViewGroup) || !((ViewGroup) mParent).isViewTransitioning(this));
}
```

1、判断是否跳过此次 invalidate

2、判断是否需要重绘

在处理了一些标志位的逻辑后调用了父 View 的 invalidateChild 方法并将要重绘的区域 damage 传给父 View。于是接着看 ViewGroup.invalidateChild 方法：

```java
public final void invalidateChild(View child, final Rect dirty) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null && attachInfo.mHardwareAccelerated) {
        // 开启了硬件加速
        onDescendantInvalidated(child, child);
        return;
    }
    // 未开启硬件加速
    ViewParent parent = this;
    if (attachInfo != null) {
        // ...
        do {
            // ...
            parent = parent.invalidateChildInParent(location, dirty);
            // 重新设置脏区
            // ...
        } while (parent != null);
    }
}
```

根据是否开启了硬件加速而走不同的逻辑。

### 软件绘制

关闭硬件加速时会循环调用 parent.invalidateChildInParent 方法并将返回值赋给 parent 直到其为 null 时退出循环，于是可以看看 ViewGroup.invalidateChildInParent 方法：

```java
public ViewParent invalidateChildInParent(final int[] location, final Rect dirty) {
    if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_DRAWING_CACHE_VALID)) != 0) {
        // 处理运算 dirty 区域
        // ...
        // 移除 PFLAG_DRAWING_CACHE_VALID 标志位
        mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
        return mParent;
    }
    return null;
}
```

invalidateChildInParent 方法主要是对子 View 传递的 dirty 区域进行处理与运算并返回 mParent 对象。因此在 invalidateChild 方法中会循环逐层调用父 View 的 invalidateChildInParent 方法，最终来到顶层的 DecorView.invalidateChild 方法，其 mParent 是 ViewRootImpl。

#### ViewRootImpl.invalidateChildInParent

```java
public ViewParent invalidateChildInParent(int[] location, Rect dirty) {
    checkThread(); // 检查当前线程是否是主线程
    // ...
    invalidateRectOnScreen(dirty);
    return null;
}

private void invalidateRectOnScreen(Rect dirty) {
    // ...
    scheduleTraversals();
}
```

可以看到 ViewRootImpl.invalidateChildInParent 方法返回 null，因此 ViewGroup.invalidateChild 会退出循环。

scheduleTraversals 方法在 Vsync 信号到来后便会执行 performTraversals 方法(参考 [Android-Choreographer工作原理](https://juejin.cn/post/6894206842277199880))，由于 mLayoutRequested 默认值是 false 且 invalidate 过程并没有给它赋值，于是不会调用 measureHierarchy 方法，而只有满足上面的条件 —— `mFirst || windowShouldResize || insetsChanged || viewVisibilityChanged || params != null ...` 才会调用 performMeasure 和 performLayout，否则只会调用 performDraw 去重新 draw，并在其中给 View 设置 PFLAG_DRAWN 标志位。

### 硬件绘制

开启了硬件绘制后会走 ViewGroup.onDescendantInvalidated 方法：

```java
public final void invalidateChild(View child, final Rect dirty) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null && attachInfo.mHardwareAccelerated) {
        // 开启了硬件加速
        onDescendantInvalidated(child, child);
        return;
    }
    // 未开启硬件加速
		// ...
}

public void onDescendantInvalidated(@NonNull View child, @NonNull View target) {
    // ...
    if (mParent != null) {
        mParent.onDescendantInvalidated(this, target);
    }
}
```

可以注意到，开启硬件加速后，dirty区域就不会传给父View了。

和软件绘制类似，接下来会逐级调用父 View 的 onDescendantInvalidated 方法，最后走到 ViewRootImpl.onDescendantInvalidated 方法：

```java
public void onDescendantInvalidated(@NonNull View child, @NonNull View descendant) {
    invalidate();
}

void invalidate() {
    mDirty.set(0, 0, mWidth, mHeight);
    if (!mWillDrawSoon) {
        scheduleTraversals();
    }
}
```

可以看到跟软件绘制类似调用了 scheduleTraversals 方法，在 Vsync 信号到来后执行 performTraversals 方法，由于 mLayoutRequested 默认值是 false 且 invalidate 过程并没有给它赋值，于是不会调用 measureHierarchy 方法，而只有满足上面的条件 —— `mFirst || windowShouldResize || insetsChanged || viewVisibilityChanged || params != null ...` 才会调用 performMeasure 和 performLayout，否则只会调用 performDraw 去重新 draw。

根据 [Android-Surface之创建流程及软硬件绘制](https://juejin.cn/post/6896382932639580167) 的解析，开启硬件加速后 performDraw 方法会通过 mAttachInfo.mThreadedRenderer.draw 来绘制，接着调用到 ThreadedRenderer.updateRootDisplayList 方法：

```java
private boolean performDraw() {
  //...
  boolean canUseAsync = draw(fullRedrawNeeded, usingAsyncReport && mSyncBuffer);
  //...
}

private boolean draw(boolean fullRedrawNeeded, boolean forceDraw) {
  	//...
		if (isHardwareEnabled()) {
				mAttachInfo.mThreadedRenderer.draw(mView, mAttachInfo, this);
		} else	{
		    if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset,
		        scalingRequired, dirty, surfaceInsets)) {
		    return false;
		}
    //...
}

// ThreadedRenderer.updateRootDisplayList
private void updateRootDisplayList(View view, DrawCallbacks callbacks) {
    updateViewTreeDisplayList(view);
    // ...
}

private void updateViewTreeDisplayList(View view) {
    view.mPrivateFlags |= View.PFLAG_DRAWN;
  
  // 在调用 View.invalidate 方法时对该 View 设置了 PFLAG_INVALIDATED 标志位，因此调用 invalidate 方法的 	// View 的 mRecreateDisplayList 属性为 true，其它 View 为 false。
    view.mRecreateDisplayList = (view.mPrivateFlags & View.PFLAG_INVALIDATED) == View.PFLAG_INVALIDATED;
    view.mPrivateFlags &= ~View.PFLAG_INVALIDATED;
    view.updateDisplayListIfDirty();
    view.mRecreateDisplayList = false;
}
```

接下来会调用到 view.updateDisplayListIfDirty 方法：

```java
public RenderNode updateDisplayListIfDirty() {
    // 调用 invalidate 时 PFLAG_DRAWING_CACHE_VALID 标志位已经被清除，因此该条件为 true
    if ((mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == 0 || !renderNode.isValid() || (mRecreateDisplayList)) {
        if (renderNode.isValid() && !mRecreateDisplayList) {
            // 不需要重绘
            mPrivateFlags |= PFLAG_DRAWN | PFLAG_DRAWING_CACHE_VALID;
            mPrivateFlags &= ~PFLAG_DIRTY_MASK;
            dispatchGetDisplayList();
            return renderNode; // no work needed
        }

        // 需要重绘
        mRecreateDisplayList = true;
        // 重绘...
    }
    return renderNode;
}
```

这里根据 mRecreateDisplayList 的值会判断是走 重绘 还是 dispatchGetDisplayList 的逻辑：

- 对于调用 View.invalidate 方法的 View 来说其 mRecreateDisplayList 值为 true，因此走重绘逻辑。
- 其余 View 如 DecorView 等会走 dispatchGetDisplayList 逻辑。

最开始的根 View 是 DecorView 类型，来看看 dispatchGetDisplayList 方法：

```java
protected void dispatchGetDisplayList() {
    for (int i = 0; i < count; i++) {
        final View child = children[i];
        if (((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null)) {
            recreateChildDisplayList(child);
        }
    }
    // ...
}

private void recreateChildDisplayList(View child) {
    child.mRecreateDisplayList = (child.mPrivateFlags & PFLAG_INVALIDATED) != 0;
    child.mPrivateFlags &= ~PFLAG_INVALIDATED;
    child.updateDisplayListIfDirty();
    child.mRecreateDisplayList = false;
}
```

可以看到 recreateChildDisplayList 和 updateViewTreeDisplayList 方法有些类似，都会设置 mRecreateDisplayList 的值，于是从 DecorView 开始会遍历其子 View 依次调用 updateDisplayListIfDirty 方法，对于没有设置 PFLAG_INVALIDATED 标志位的 View 会调用 dispatchGetDisplayList 接着往下分发，而设置了 PFLAG_INVALIDATED 标志位的 View 则会执行重绘逻辑，根据 [Android-View绘制原理](https://link.juejin.cn/?target=https%3A%2F%2Fljd1996.github.io%2F2020%2F09%2F09%2FAndroid-View%E7%BB%98%E5%88%B6%E5%8E%9F%E7%90%86%2F) 可知当该 View 执行 View.draw 开始重绘时，如果它是 ViewGroup 类型，则会调用 ViewGroup.dispatchDraw 方法分发 draw 事件，dispatchDraw 内部遍历子 View 然后调用 drawChild 方法：

```java
// ViewGroup
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
    return child.draw(canvas, this, drawingTime);
}

// View
boolean draw(Canvas canvas, ViewGroup parent, long drawingTime) {
    // 是否是硬件加速
    boolean drawingWithRenderNode = mAttachInfo != null && mAttachInfo.mHardwareAccelerated && hardwareAcceleratedCanvas;
    if (drawingWithRenderNode) {
        // 又会重复上面的逻辑，对于没有设置 PFLAG_INVALIDATED 标志位的 View 不会重绘
        renderNode = updateDisplayListIfDirty();
    }
    // ...
}
```

## 小结

调用 View.invalidate() 方法后会逐级往上调用父 View 的相关方法，最终在 Choreographer 的控制下调用 ViewRootImpl.performTraversals() 方法。由于 mLayoutRequested == false，因此只有满足 `mFirst || windowShouldResize || insetsChanged || viewVisibilityChanged || params != null ...` 等条件才会执行 measure 和 layout 流程，否则只执行 draw 流程，draw 流程的执行过程与是否开启硬件加速有关：

- 关闭硬件加速则从 DecorView 开始往下的所有子 View 都会被重新绘制。
- 开启硬件加速则只有调用 invalidate 方法的 View 才会重新绘制。

View 在绘制后会设置 PFLAG_DRAWN 标志位。



# requestLayout

## View.requestLayout







其他

SynchronizedPool

# 问题

invalidate的原理？                                       

invalidate和requestlayout的区别                            

requestlayout的作用范围是多大                             

invalidate()和postInvalicate()区别                         

invalidate、postInvalidate、requestLayout的区别？

自定义View执行invalidate()方法,为什么有时候不会回调onDraw()





# 参考

1、[比较一下requestLayout和invalidate方法](https://juejin.cn/post/6904518722564653070)

在其博客中还拓展了一些图形系统相关的知识。

2、