> version：2021/9/13
>
> review：



invalidate()方法的执行可以分为两步，标记dirty区域，触发绘制。

> 基于 Android 30



#### 一、dirty区域标记

```java
View.java
	// 1、invalidate的范围是整个view。
	// 2、如果此view可见，那后续后调用onDraw()。
	public void invalidate() {
        invalidate(true);
    }

    public void invalidate(boolean invalidateCache) {
        // 四个参数分别是：左、上、宽、高
        // left、top、width、height
        invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
    }
// 分析：invalidateInternal的参数
    void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
            boolean fullInvalidate) {
        // 这次主要分析 dirty 区域的计算过程，其他代码去除了
        。。。
        	// 1、添加标记：PFLAG_DIRTY
            mPrivateFlags |= PFLAG_DIRTY;

            if (invalidateCache) {
                // 2、添加标记：PFLAG_INVALIDATED
                mPrivateFlags |= PFLAG_INVALIDATED;
                // 3、去除标记：PFLAG_DRAWING_CACHE_VALID
                mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
            }

            // Propagate the damage rectangle to the parent view.
            final AttachInfo ai = mAttachInfo;
            final ViewParent p = mParent;
            if (p != null && ai != null && l < r && t < b) {
                final Rect damage = ai.mTmpInvalRect;
                // 4、传入 相对于父布局的坐标 给ViewParent，
                // ViewParent的实现类有ViewGroup和ViewRootImpl（最终的parent）
                damage.set(l, t, r, b);
                p.invalidateChild(this, damage);
            }
。。。
        }
    }
```

分析：invalidateInternal的参数

1. mLeft、mRight、mTop、mBottom：这四个参数值由其父view和自身决定。

   mLeft：父View 左边缘 到此View 左边缘 的距离，单位像素px

   mRight：父View 左边缘 到此View 右边缘 的距离，单位px

   mTop：从该视图的父视图的 上边缘 到该视图 上边缘 的距离，单位px

   mBottom：从该视图的父视图的 上边缘 到该视图的 下边缘 的距离

2. boolean invalidateCache ：是否使此View的图形缓存无效。一般为true。如果调用的是invalidate()，那么恒为true。

3. boolean fullInvalidate：这个view都无效。如果调用的是invalidate()，那么恒为true。

ViewGroup#invalidateChild()

```java
ViewGroup.java
    public final void invalidateChild(View child, final Rect dirty) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null && attachInfo.mHardwareAccelerated) {
            // HW accelerated fast path
            onDescendantInvalidated(child, child);
            return;
        }

        ViewParent parent = this;
        if (attachInfo != null) {
            // If the child is drawing an animation, we want to copy this flag onto
            // ourselves and the parent to make sure the invalidate request goes
            // through
            final boolean drawAnimation = (child.mPrivateFlags & PFLAG_DRAW_ANIMATION) != 0;

            // Check whether the child that requests the invalidate is fully opaque
            // Views being animated or transformed are not considered opaque because we may
            // be invalidating their old position and need the parent to paint behind them.
            Matrix childMatrix = child.getMatrix();
            // Mark the child as dirty, using the appropriate flag
            // Make sure we do not set both flags at the same time

            if (child.mLayerType != LAYER_TYPE_NONE) {
                mPrivateFlags |= PFLAG_INVALIDATED;
                mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
            }

            final int[] location = attachInfo.mInvalidateChildLocation;
            location[CHILD_LEFT_INDEX] = child.mLeft;
            location[CHILD_TOP_INDEX] = child.mTop;
            if (!childMatrix.isIdentity() ||
                    (mGroupFlags & ViewGroup.FLAG_SUPPORT_STATIC_TRANSFORMATIONS) != 0) {
                RectF boundingRect = attachInfo.mTmpTransformRect;
                boundingRect.set(dirty);
                Matrix transformMatrix;
                if ((mGroupFlags & ViewGroup.FLAG_SUPPORT_STATIC_TRANSFORMATIONS) != 0) {
                    Transformation t = attachInfo.mTmpTransformation;
                    boolean transformed = getChildStaticTransformation(child, t);
                    if (transformed) {
                        transformMatrix = attachInfo.mTmpMatrix;
                        transformMatrix.set(t.getMatrix());
                        if (!childMatrix.isIdentity()) {
                            transformMatrix.preConcat(childMatrix);
                        }
                    } else {
                        transformMatrix = childMatrix;
                    }
                } else {
                    transformMatrix = childMatrix;
                }
                transformMatrix.mapRect(boundingRect);
                dirty.set((int) Math.floor(boundingRect.left),
                        (int) Math.floor(boundingRect.top),
                        (int) Math.ceil(boundingRect.right),
                        (int) Math.ceil(boundingRect.bottom));
            }

            do {
                View view = null;
                if (parent instanceof View) {
                    view = (View) parent;
                }

                if (drawAnimation) {
                    if (view != null) {
                        view.mPrivateFlags |= PFLAG_DRAW_ANIMATION;
                    } else if (parent instanceof ViewRootImpl) {
                        ((ViewRootImpl) parent).mIsAnimating = true;
                    }
                }

                // If the parent is dirty opaque or not dirty, mark it dirty with the opaque
                // flag coming from the child that initiated the invalidate
                if (view != null) {
                    if ((view.mPrivateFlags & PFLAG_DIRTY_MASK) != PFLAG_DIRTY) {
                        view.mPrivateFlags = (view.mPrivateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DIRTY;
                    }
                }

                parent = parent.invalidateChildInParent(location, dirty);
                if (view != null) {
                    // Account for transform on current parent
                    Matrix m = view.getMatrix();
                    if (!m.isIdentity()) {
                        RectF boundingRect = attachInfo.mTmpTransformRect;
                        boundingRect.set(dirty);
                        m.mapRect(boundingRect);
                        dirty.set((int) Math.floor(boundingRect.left),
                                (int) Math.floor(boundingRect.top),
                                (int) Math.ceil(boundingRect.right),
                                (int) Math.ceil(boundingRect.bottom));
                    }
                }
            } while (parent != null);
        }
    }    
```

示例：

在这样一个布局中，Button是居中的。

![image-20210913232729903](images/invalidate()、RequestLayout()/image-20210913232729903.png)

已知：

dirty = 0，0 - 242,132，父布局宽高为 150dp，150dp，设备密度为：

wm 取到的 density 实际上是 dpi，转换后得到Android中的density（一般命名为scale）

density = dpi / 160 = 2.75

因此父布局的宽高（单位像素）为：

width = density * 150dp + 0.5f = 2.75 * 150 = 413px

height = density * 150dp + 0.5f = 2.75 * 150 = 413px

所以现在的Button相对于父布局的坐标是：

left =（413 - 242）/ 2 = 85.5

top = (413 - 132) / 2 = 140.5

那下一次dirty计算出来的结果推测是：

dirty = 85.5，140.5 - 242,132





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







#### invalidate()源码分析

它在触发绘制时，不仅绘制自己的View，还会去绘制parent View。

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





# 问题

<font color='orange'>Q：invalidate的原理？</font>

invalidate() 用于触发 onDraw() 的执行，只能在 UI 线程调用。

`invalidate`在Android中的执行流程是一个涉及视图重绘请求的过程。以下是`invalidate`方法执行流程的详细概述：

1. 触发`invalidate`

当视图的某个部分需要更新显示时（如内容变化、状态更新等），会调用该视图的`invalidate`方法。

`invalidate`方法可以由开发者直接调用，也可以由系统函数间接调用（如`setVisibility()`, `setEnabled()`, `setSelected()`等）。

2. 调用`invalidate(boolean invalidateCache)`

在`View`类中，`invalidate()`方法通常会调用`invalidate(boolean invalidateCache)`方法，并传入`true`作为参数，表示需要同时无效化绘制缓存。

3. 判断是否需要重绘

在`invalidate(boolean invalidateCache)`方法中，首先会调用`skipInvalidate()`方法来判断该视图是否不需要重绘。这通常基于视图的可见性和动画状态。如果视图不可见或未进行动画，则直接返回，不执行后续的重绘流程。

4. 设置重绘标志

如果视图需要重绘，则会设置相应的标志位（如`DIRTY`），表示该视图已被标记为“脏”，需要在下一次绘制周期中重新绘制。

如果`invalidateCache`为`true`，则还会无效化绘制缓存，以确保使用最新的内容进行绘制。

5. 向上传递重绘请求

接下来，`invalidate`流程会向上传递到视图的父视图。这是通过调用父视图的`invalidateChild(View child, Rect dirty)`方法实现的。

在这个过程中，如果开启了硬件加速，则可能会采用快速路径来无效化整个视图树，而不是仅仅无效化一个特定的区域。

如果没有开启硬件加速，则会计算需要重绘的区域（即“dirty rect”），并将其传递给父视图。

6. 视图树的重新绘制

最终，这些重绘请求会传递到根视图（如`DecorView`），并由`ViewRootImpl`处理。

`ViewRootImpl`会安排在下一个绘制帧中重新执行绘制流程，包括测量（Measure）、布局（Layout）和绘制（Draw）步骤。

在绘制步骤中，所有被标记为“脏”的视图都会重新调用它们的`onDraw`方法来绘制新的内容。

综上所述，`invalidate`的执行流程是一个从视图自身开始，向上传递到根视图，并最终由`ViewRootImpl`安排重新绘制的过程。这个过程中涉及到了视图的可见性检查、重绘标志的设置、重绘请求的向上传递以及视图树的重新绘制等多个步骤。

<font color='orange'>Q：invalidate()和postInvalicate()区别</font>

invalidate() 只能在UI线程调用。

postInvalicate() 可以在子线程中使用。

都可以用来触发重绘。

<font color='orange'>Q：invalidate和requestlayout的区别</font>

- **功能区别**：`invalidate`主要用于请求视图内容的重绘，而`requestLayout`主要用于请求视图大小和位置的重新计算。
- **性能影响**：`invalidate`是轻量级的，因为它只涉及到绘制过程；而`requestLayout`是重量级的，因为它会触发整个视图树的布局过程。
- **使用场景**：根据你是否改变了视图的内容、大小或位置来选择使用哪个方法。如果只是内容变化，使用`invalidate`；如果大小或位置变化，使用`requestLayout`。

<font color='orange'>Q：requestlayout的作用范围是多大</font>

`requestLayout`的作用范围涉及整个视图树中与当前调用`requestLayout`的视图直接或间接相关的部分。具体来说，当在Android开发中调用一个视图的`requestLayout`方法时，会发生以下过程：

**作用流程**

1. **标记当前视图**：首先，系统会为当前调用`requestLayout`的视图设置一个标记（如`PFLAG_FORCE_LAYOUT`），表明这个视图需要重新布局。
2. **向上传递请求**：然后，这个请求会向上传递到该视图的父视图。父视图同样会检查自己是否需要重新布局，如果需要，则设置相应的标记，并继续向上传递请求。这个过程会一直持续到根视图（如`DecorView`），或者遇到已经处于布局流程中的视图树为止。
3. **处理布局请求**：当根视图（或能够处理布局请求的视图）接收到这些请求时，它会触发整个视图树的测量（Measure）、布局（Layout）和绘制（Draw）过程。这个过程中，所有设置了重新布局标记的视图都会根据新的布局参数重新计算其大小和位置。

**作用范围**

- **直接作用**：直接调用`requestLayout`的视图本身会触发重新布局。
- **间接作用**：由于布局请求会向上传递，因此所有直接或间接依赖于该视图的父视图和子视图（如果它们的布局受到该视图变化的影响）都可能参与到重新布局的过程中。

**注意事项**

- **性能影响**：由于`requestLayout`可能会触发整个视图树的重新布局，因此它可能会对应用的性能产生影响。特别是在视图树较大或布局过程较复杂的情况下，频繁调用`requestLayout`可能会导致性能问题。
- **异步处理**：`requestLayout`的调用是异步的，即它不会立即触发布局过程。布局过程会在下一个绘制帧之前进行。
- **正确使用**：为了优化性能，应仅在确实需要时才调用`requestLayout`。如果只需要更新视图的内容而不需要改变其大小和位置，则应考虑使用`invalidate`方法。

综上所述，`requestLayout`的作用范围是整个视图树中与当前调用它的视图直接或间接相关的部分，具体取决于视图之间的依赖关系和布局参数的变化。

<font color='orange'>Q：自定义View执行invalidate()方法，为什么有时候不会回调onDraw()？</font>

invalidate()在执行会根据一些条件来判断是否触发重绘，比如在



# 参考

[比较一下requestLayout和invalidate方法](https://juejin.cn/post/6904518722564653070)，在其博客中还拓展了一些图形系统相关的知识。
