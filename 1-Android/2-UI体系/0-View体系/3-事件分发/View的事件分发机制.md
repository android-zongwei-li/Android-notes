#一、什么是事件分发
######1、事件
用户通过屏幕与手机交互的时候，每一次点击、长按、移动等都是一个事件。

######2、事件分发机制
某一个事件从屏幕传递到各个View，由View来使用这一事件（消费事件）或者忽略这一事件（不消费事件），对这整个过程的控制就是事件分发机制。



#二、事件分发的对象是谁
系统把事件封装为MotionEvent对象，事件分发的过程就是MotionEvent分发的过程。

下面对**MotionEvent**进行简单介绍
######1、事件的类型
* 按下（ACTION_DOWN）
* 移动（ACTION_MOVE）
* 抬起（ACTION_UP）
* 取消（ACTION_CANCEL）

######2、事件序列
从手指按下屏幕开始，到手指离开屏幕所产生的一系列事件。



#三、传递层级
Activity -> Window -> DecorView -> ViewGroup -> View

1、涉及到的主要对象及顺序
* Activity 
* ViewGroup
* View



#四、Activity中的事件分发流程
######1、流程图
![Activity中的事件分发流程图](https://upload-images.jianshu.io/upload_images/9000209-cd17ad15eb333a60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######2、源码分析：
Activity.java
```
    public boolean dispatchTouchEvent(MotionEvent ev) {
//第一步：判断并调用onUserInteraction()。
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
//第二步：事件派发，返回值表示是否被消费
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
//第三步：没被消费，Activity中的onTouchEvent(ev)方法，进行处理。
        return onTouchEvent(ev);
    }

    public void onUserInteraction() {
    }

/**
*  当事件派发后，还没有被消费掉的话，就调用Activity#onTouchEvent方法进行处理。
**/
    public boolean onTouchEvent(MotionEvent event) {
//系统默认的处理是：
//如果shouldCloseOnTouch返回true，就finish掉当前的Activity，并返回true，表示事件消费掉了
//不然，就返回false，往下走
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }

//如果shouldCloseOnTouch返回false，返回false
        return false;
    }
```

Window.java
```
    private boolean mCloseOnTouchOutside = false;
    private boolean mSetCloseOnTouchOutside = false;

    public void setCloseOnTouchOutside(boolean close) {
        mCloseOnTouchOutside = close;
        mSetCloseOnTouchOutside = true;
    }

    public void setCloseOnTouchOutsideIfNotSet(boolean close) {
        if (!mSetCloseOnTouchOutside) {
            mCloseOnTouchOutside = close;
            mSetCloseOnTouchOutside = true;
        }
    }

    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
        final boolean isOutside =
                event.getAction() == MotionEvent.ACTION_DOWN && isOutOfBounds(context, event)
                || event.getAction() == MotionEvent.ACTION_OUTSIDE;
        if (mCloseOnTouchOutside && peekDecorView() != null && isOutside) {
            return true;
        }
        return false;
    }
```

在这做一个标注：Activity中的getWindow、onUserInteraction、onTouchEvent，以及setCloseOnTouchOutside方法都是public的。简单来说，这意味着在事件分发的过程中，我们是可以参与、写入自己的逻辑的。

######3、事件派发过程分析（从Activity到DecorView）：
在Activity#dispatchTouchEvent中的第二步，调用了下面的一行代码，对事件进行了分发：
```
getWindow().superDispatchTouchEvent(ev)
```
整个调用流程如下：
Activity.java
```
getWindow().superDispatchTouchEvent(ev)
```
Window.java
```
public abstract boolean superDispatchTouchEvent(MotionEvent event);
```
PhoneWindow.java (它是Window的唯一实现类)
```
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }
```
到这，可以看到事件已经被交给DecorView了。
DecorView.java
```
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return super.dispatchTouchEvent(event);
    }
```
可以看到，DecorView是调用父类的dispatchTouchEvent方法，对事件进行分发。

那它的父类是谁呢？
DecorView extends FrameLayout
FrameLayout extends ViewGroup
**最终调用的是ViewGroup#dispatchTouchEvent**

**到此，这一步的分发完成了：
事件从Activity传到Window，再从Window传到DecorView中。在DecorView中调用了父类ViewGroup#dispatchTouchEvent方法进行下一步的分发。**





#四、ViewGroup中的事件分发流程
首先看一下ViewGroup事件分发中涉及到的三个主要方法：
* public boolean dispatchTouchEvent(MotionEvent ev) {...}
事件进入ViewGroup中。
* public boolean onInterceptTouchEvent(MotionEvent ev) {...}
事件分发过程中调用：
如果返回true，表示当前的ViewGroup会拦截掉这个事件，也就是说不会再往下进行传递了；
如果返回false，表示当前的ViewGroup不会拦截掉这个事件。
* public boolean onTouchEvent(MotionEvent event) {...}
这个方法在ViewGroup的父类View中。

######1、流程图
![ViewGroup事件分发流程图](https://upload-images.jianshu.io/upload_images/9000209-1b664f95d266aec0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######2、ViewGroup#dispatchTouchEvent方法做了什么
1）去判断是否需要拦截事件
2）在当前ViewGroup中找到用户真正点击的View
3）分发事件到View上

######3、源码分析
```
    public boolean dispatchTouchEvent(MotionEvent ev) {
//与事件分发无关，无需关注，debug用的
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

//辅助功能选项，暂无需关注
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

//事件分发逻辑开始

//handled作为此方法的返回值，表示事件是否被消费
        boolean handled = false;
//第一步：判断是否符合安全策略
//安全策略：如果当前的View被其他视图遮挡了，并且当前的View被设置了——不在顶部时不响应事件，那么该方法就会返回false，也就不会向其分发事件了。这个方法的源码在后面分析
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

//处理最初的DOWN事件，这个处理并不会影响事件的分发，本次分析不作为重点
            if (actionMasked == MotionEvent.ACTION_DOWN) {
//取消并且清除一个触摸事件的目标链表
                cancelAndClearTouchTargets(ev);
//重置触摸状态
              resetTouchState();
            }

            // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                intercepted = true;
            }

//暂无需关注
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation.
//true：表示当前事件为取消事件
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;

//如果为true，表示当前的事件分发给多个子视图，因为多个子视图可能重叠在一起
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
//如果不是取消事件，并且当前不去拦截事件，那么进入到事件分发的逻辑中
            if (!canceled && !intercepted) {
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    // Clean up earlier touch targets for this pointer id in case they
                    // have become out of sync.
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
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
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // Did not find a child to receive the event.
                        // Assign the pointer to the least recently added target.
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
```

安全策略源码：
```
    public boolean onFilterTouchEventForSecurity(MotionEvent event) {
//如果当前View设置了：被遮挡时需要过滤此触摸事件。并且当前触摸事件处于被遮挡状态。
        if ((mViewFlags & FILTER_TOUCHES_WHEN_OBSCURED) != 0
                && (event.getFlags() & MotionEvent.FLAG_WINDOW_IS_OBSCURED) != 0) {
            // Window is obscured, drop this touch.
//过滤掉这个触摸事件，也就是不分发了
            return false;
        }
        return true;
    }
```