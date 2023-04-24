> version：2022/06/04
>
> review：



目录

[TOC]



# 1、View 的滑动

不管是哪种滑动方式，其基本原理都是类似的：当点击事件传到 View 时，系统记下**触摸点的坐标**，手指移动时系统记下**移动后触摸的坐标**并算出**偏移量**，并**通过偏移量来修改 View 的坐标**。实现 View 滑动有很多种方法，下面介绍 6 种滑动方法，分别是 layout()、offsetLeftAndRight() 与 offsetTopAndBottom()、LayoutParams、动画、scollTo 与 scollBy，以及 Scroller。

## 1.1 layout() 方法

View 在绘制的时候会调用 onLayout() 方法来设置显示的位置，因此可以通过修改 View 的 left、top、right、bottom 这 4 个属性来控制 View 的坐标。首先我们要自定义一个View，在 onTouchEvent() 方法中获取触摸点的坐标 ：

```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // 获取手指触摸点的横坐标和纵坐标
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = x;
                lastY = y;
                break;
...
        }
        return true;
    }
```

接下来我们在 ACTION_MOVE 事件中计算偏移量，再调用 layout()方法重新放置这个自定义 View 的位置即可。

```java
    case MotionEvent.ACTION_MOVE:
        // 计算移动的距离
        int offsetX = x - lastX;
        int offsetY = y - lastY;
        // 调用 layout 方法来重新放置它的位置
        layout(getLeft() + offsetX, getTop() + offsetY,
                getRight() + offsetX, getBottom() + offsetY);
        break;
```

在每次移动时都会调用 layout()方法对屏幕重新布局，从而达到移动 View 的效果。自定义 View，CustomView 的全部代码如下所示：

```java
public class FingerView extends View {
    private int lastX;
    private int lastY;

    public FingerView(Context context) {
        super(context);
    }

    public FingerView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public FingerView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
    }
    
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // 获取手指触摸点的横坐标和纵坐标
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算移动的距离
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                // 调用 layout 方法来重新放置它的位置
                layout(getLeft() + offsetX, getTop() + offsetY,
                        getRight() + offsetX, getBottom() + offsetY);
                break;
        }
        return true;
    }
}
```

布局中使用：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.lizw.viewsystemdemo.CustomView
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorAccent"
        />

</LinearLayout>
```

运行效果图。图中的方块就是我们自定义的 CustomView，它会随着我们手指的滑动改变自己的位置。



#### 3.2 offsetLeftAndRight()与 offsetTopAndBottom()

这两种方法和 layout()方法的效果差不多，其使用方式也差不多。我们将 ACTION_MOVE 中的代码替换成如下代码：

```java
    case MotionEvent.ACTION_MOVE:
        // 计算移动的距离
        int offsetX = x - lastX;
        int offsetY = y - lastY;
        // 对 left 和 right 进行偏移
        offsetLeftAndRight(offsetX);
        // 对 top 和 bottom 进行偏移
        offsetTopAndBottom(offsetY);
        break;
```

#### 3.3 LayoutParams（改变布局参数）

LayoutParams 主要保存了一个 View 的布局参数，因此我们可以通过 LayoutParams 来改变 View 的布局参数从而达到改变 View 位置的效果。同样，我们将 ACTION_MOVE 中的代码替换成如下代码：

```java
    LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) getLayoutParams();
    lp.leftMargin = getLeft() + offsetX;
    lp.topMargin = getTop() + offsetY;
    setLayoutParams(lp);
```

因为父控件是 LinearLayout，所以我们用了 LinearLayout.LayoutParams。如果父控件是RelativeLayout，则要使用 RelativeLayout.LayoutParams。除了使用布局的 LayoutParams 外，我们还可以用 ViewGroup.MarginLayoutParams 来实现：

```java
    ViewGroup.MarginLayoutParams mlp = (ViewGroup.MarginLayoutParams) getLayoutParams();
    mlp.leftMargin = getLeft() + offsetX;
    mlp.topMargin = getTop() + offsetY;
    setLayoutParams(mlp);
```

#### 3.4 动画

可以采用 View 动画来移动，在 res 目录新建 anim 文件夹并创建 translate.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="1000"
        android:fromXDelta="0"
        android:toXDelta="300" />
</set>
```

接下来在 Java 代码中调用就好了，代码如下所示：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    CustomView customView = findViewById(R.id.custom_view);
    customView.setAnimation(AnimationUtils.loadAnimation(this, R.anim.translate));
}
```

运行程序，我们设置的方块会向右平移 300 像素，然后又会回到原来的位置。为了解决这个问题，我们需要在 translate.xml 中加上 fillAfter="true"，代码如下所示。运行代码后会发现，方块向右平移 300 像素后就停留在当前位置了。

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true">
    <translate
        android:duration="2000"
        android:fromXDelta="0"
        android:toXDelta="300" />
</set>
```

需要注意的是，View 动画并不能改变 View 的位置参数。如果对一个 Button 进行如上的平移动画操作，当 Button 平移 300 像素停留在当前位置时，我们点击这个 Button 并不会触发点击事件，但在我们点击这个 Button 的原始位置时却触发了点击事件。对于系统来说这个 Button 并没有改变原有的位置，所以我们点击其他位置当然不会触发这个 Button 的点击事件。在 Android 3.0 时出现的属性动画解决了上述问题，因为它不仅可以执行动画，还能够改变 View 的位置参数。当然，这里使用属性动画移动那就更简单了，我们让 CustomView 在 2000ms 内沿着 *X* 轴向右平移 300 像素，代码如下所示。关于属性动画，见 View > 2-动画 > 属性动画.md

```java
    ObjectAnimator.ofFloat(customView, "translationX", 0, 300)
            .setDuration(2000L)
            .start();
	}
```

#### 3.5 scrollTo 与 scollBy

scrollTo(x,y)表示移动到一个具体的坐标点，而 scrollBy(dx,dy)则表示移动的增量为 dx、dy。其中，scollBy 最终也是要调用 scollTo 的。View.java 的 scollBy 和 scollTo 的源码如下所示：

```java
    public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
    }
    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }
```

scollTo、scollBy 移动的是 View 的内容，如果在 ViewGroup 中使用，则是移动其所有的子View。我们将 ACTION_MOVE 中的代码替换成如下代码：

```java
	((View) getParent()).scrollBy(-offsetX,-offsetY);
```

为什么要设置为负值呢？

想要让 View 中的内容下移，那么此 View 就需要向上滚动。

这个可以参考编辑器的滚动条，多多体验。

（向上滚动，内容下移；向下滚动，内容上移。）

![image-20210420144412733](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210420144412733.png)

#### 3.6 Scroller

我们在用 scollTo/scollBy 方法进行滑动时，这个过程是瞬间完成的，很多时候我们会希望过渡的更加平滑。这里可以使用 Scroller 来实现有过渡效果的滑动，这个过程不是瞬间完成的，而是在一定的时间间隔内完成的。Scroller 本身是不能实现 View 的滑动的，它需要与 View 的 computeScroll()方法配合才能实现弹性滑动的效果。在这里我们实现 CustomView 平滑地向右移动。首先我们要初始化 Scroller，代码如下所示：

```java
    public CustomView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        mScroller = new Scroller(context);
    }
```

接下来重写 computeScroll()方法，系统会在绘制 View 的时候在 draw()方法中调用该方法。 在这个方法中，我们调用父类的 scrollTo()方法并通过 Scroller 来不断获取当前的滚动值，每滑动一小段距离我们就调用 invalidate()方法不断地进行重绘，重绘就会调用 computeScroll()方法，这样我们通过不断地移动一个小的距离并连贯起来就实现了平滑移动的效果。

```java
    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            ((View) getParent()).scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            invalidate();
        }
    }
```

我们在 CustomView 中写一个 smoothScrollTo 方法，调用 Scroller 的 startScroll()方法，在2000ms 内沿 *X* 轴平移 delta 像素，代码如下所示：

```java
    public void smoothScrollTo(int destX, int destY) {
        // Android进阶之光中用的是：getScrollX()，应该是存在错误的（书中例子结果是对的），但
        // 每次滚动都会从 0 开始，应该是不符合需求的，我们还是需要滚动其父 View
        int scrollX = ((View) getParent()).getScrollX();
        int delta = destX - scrollX;
        mScroller.startScroll(scrollX, 0, delta, 0, 2000);
        invalidate();
    }
```

最后调用 CustomView 的 smoothScrollTo()方法。这里我们设定 CustomView 沿着 *X* 轴向右平移 400 像素。

```java
    customView.smoothScrollTo(-400, 0);
```

### 4、解析 Scroller

在 3.6 节中我们介绍了如何使用 Scroller 进行滑动，但是其使用流程和一般的类的使用方式稍有不同。为了更好地理解 Scroller 的使用流程，我们有必要学习一下 Scroller 的源码。要想使用 Scroller，必须先调用 new Scroller()。下面先来看看 Scroller 的构造方法，代码如下所示：

```java
    public Scroller(Context context) {
        this(context, null);
    }
    public Scroller(Context context, Interpolator interpolator) {
        this(context, interpolator,
                context.getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.HONEYCOMB);
    }
    public Scroller(Context context, Interpolator interpolator, boolean flywheel) {
        mFinished = true;
        if (interpolator == null) {
            mInterpolator = new ViscousFluidInterpolator();
        } else {
            mInterpolator = interpolator;
        }
        mPpi = context.getResources().getDisplayMetrics().density * 160.0f;
        mDeceleration = computeDeceleration(ViewConfiguration.getScrollFriction());
        mFlywheel = flywheel;

        mPhysicalCoeff = computeDeceleration(0.84f); // look and feel tuning
    }

```

Scroller 有三个构造方法，通常情况下我们都用第一个；第二个需要传进去一个插值器 Interpolator，如果不传则采用默认的插值器 ViscousFluidInterpolator。接下来看看 Scroller 的 startScroll()方法，代码如下所示：

```java
    public void startScroll(int startX, int startY, int dx, int dy, int duration) {
        mMode = SCROLL_MODE;
        mFinished = false;
        mDuration = duration;
        mStartTime = AnimationUtils.currentAnimationTimeMillis();
        mStartX = startX;
        mStartY = startY;
        mFinalX = startX + dx;
        mFinalY = startY + dy;
        mDeltaX = dx;
        mDeltaY = dy;
        mDurationReciprocal = 1.0f / (float) mDuration;
    }
```

在 startScroll()方法中并没有调用类似开启滑动的方法，而是保存了传进来的各种参数：startX 和 startY 表示滑动开始的起点，dx 和 dy 表示滑动的距离，duration 则表示滑动持续的时间。所以 startScroll()方法只是用来做前期准备的，并不能使 View 进行滑动。关键是我们在 startScroll()方法后调用了 invalidate()方法，这个方法会导致 View 的重绘，而 View 的重绘会调用 View 的 draw()方法，draw()方法又会调用 View 的 computeScroll()方法。我们重写 computeScroll()方法如下：

```java
    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            ((View) getParent()).scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            invalidate();
        }
    }
```

我们在 computeScroll()方法中通过 Scroller 来获取当前的 ScrollX 和 ScrollY，然后调用 scrollTo()方法进行 View 的滑动，接着调用 invalidate 方法来让 View 进行重绘，重绘又会调用 computeScroll()方法来实现 View 的滑动。这样通过不断地移动一个小的距离并连贯起来就实现了平滑移动的效果。但是在 Scroller 中如何获取当前位置的 ScrollX 和 ScrollY 呢？需要在调用 scrollTo()方法前会调用 Scroller 的 computeScrollOffset()方法。接下来看看computeScrollOffset()方法：

```java
    public boolean computeScrollOffset() {
        if (mFinished) {
            return false;
        }

        int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
    
        if (timePassed < mDuration) {
            switch (mMode) {
            case SCROLL_MODE:
                final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                mCurrX = mStartX + Math.round(x * mDeltaX);
                mCurrY = mStartY + Math.round(x * mDeltaY);
                break;
            case FLING_MODE:
                final float t = (float) timePassed / mDuration;
                final int index = (int) (NB_SAMPLES * t);
                float distanceCoef = 1.f;
                float velocityCoef = 0.f;
                if (index < NB_SAMPLES) {
                    final float t_inf = (float) index / NB_SAMPLES;
                    final float t_sup = (float) (index + 1) / NB_SAMPLES;
                    final float d_inf = SPLINE_POSITION[index];
                    final float d_sup = SPLINE_POSITION[index + 1];
                    velocityCoef = (d_sup - d_inf) / (t_sup - t_inf);
                    distanceCoef = d_inf + (t - t_inf) * velocityCoef;
                }

                mCurrVelocity = velocityCoef * mDistance / mDuration * 1000.0f;
                
                mCurrX = mStartX + Math.round(distanceCoef * (mFinalX - mStartX));
                // Pin to mMinX <= mCurrX <= mMaxX
                mCurrX = Math.min(mCurrX, mMaxX);
                mCurrX = Math.max(mCurrX, mMinX);
                
                mCurrY = mStartY + Math.round(distanceCoef * (mFinalY - mStartY));
                // Pin to mMinY <= mCurrY <= mMaxY
                mCurrY = Math.min(mCurrY, mMaxY);
                mCurrY = Math.max(mCurrY, mMinY);

                if (mCurrX == mFinalX && mCurrY == mFinalY) {
                    mFinished = true;
                }

                break;
            }
        }
        else {
            mCurrX = mFinalX;
            mCurrY = mFinalY;
            mFinished = true;
        }
        return true;
    }
```

首先会计算动画持续的时间 timePassed。如果动画持续时间小于我们设置的滑动持续时间mDuration，则执行 Switch 语句。因为在 startScroll()方法中的 mMode 值为 SCROLL_MODE，所以执行分支语句 SCROLL_MODE，然后根据插值器 Interpolator 来计算出在该时间段内移动的距离，赋值给 mCurrX 和 mCurrY，这样我们就能通过 Scroller 来获取当前的 ScrollX 和 ScrollY 了。另外，computeScrollOffset()的返回值如果为 true 则表示滑动未结束，为 false 则表示滑动结束。所以，如果滑动未结束，我们就得持续调用 scrollTo()方法和 invalidate()方法来进行 View 的滑动。讲到这里总结一下 Scroller 的原理：

Scroller 并不能直接实现 View 的滑动，它需要配合 View 的 computeScroll()方法。在 computeScroll()中不断让 View 进行重绘，每次重绘都会计算滑动持续的时间，根据这个持续时间就能算出这次 View 滑动的位置，我们根据每次滑动的位置调用 scrollTo()方法进行滑动，这样不断地重复上述过程就形成了弹性滑动。



# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 脑图



# 参考

1、《Android进阶之光》