# 前言

- 动画的使用 是 `Android` 开发中常用的知识
- 可是动画的**种类繁多、使用复杂**，每当需要 **采用自定义动画 实现 复杂的动画效果**时，很多开发者就显得束手无策
- Android中 补间动画 & 属性动画实现动画的原理是：

![img](https:////upload-images.jianshu.io/upload_images/944365-62e601917ece2525.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

实现原理

- 其中，步骤2中的 插值器（`Interpolator`）和估值器（`TypeEvaluator`）是实现 复杂动画效果的关键
- 本文将详细讲解 估值器（`TypeEvaluator`），通过阅读本文你将能轻松实现复杂的动画效果

> Carson带你学`Android`动画系列文章：
>  [Carson带你学Android：一份全面&详细的动画知识学习攻略](https://www.jianshu.com/p/53759778284a)
>  [Carson带你学Android：常见的三种动画类型](https://www.jianshu.com/p/53759778284a)
>  [Carson带你学Android：补间动画学习教程](https://www.jianshu.com/p/733532041f46)
>  [Carson带你学Android：属性动画学习教程](https://www.jianshu.com/p/2412d00a0ce4)
>  [Carson带你学Android：逐帧动画学习教程](https://www.jianshu.com/p/225fe1feba60)
>  [Carson带你学Android：自定义动画神器-估值器(含实例教学)](https://www.jianshu.com/p/ab5785f017b2)
>  [Carson带你学Android：自定义动画神器-插值器(含实例教学)](https://www.jianshu.com/p/2f19fe1e3ca1)

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-dd640b91d4a5e822.png?imageMogr2/auto-orient/strip|imageView2/2/w/840/format/webp)

示意图

------

# 1. 简介

- 定义：一个接口
- 作用：设置 属性值 从初始值过渡到结束值 的变化具体数值

> 1. 插值器（`Interpolator`）决定 值 的变化规律（匀速、加速blabla），即决定的是变化趋势；而接下来的具体变化数值则交给估值器
> 2. 属性动画特有的属性

------

# 2. 应用场景

协助插值器 实现非线性运动的动画效果

> 非线性运动：动画改变的速率不是一成不变的，如加速 & 减速运动都属于非线性运动

------

# 3. 具体使用

设置方式如下：



```cpp
ObjectAnimator anim = ObjectAnimator.ofObject(myView2, "height", new Evaluator()，1，3);
// 在第4个参数中传入对应估值器类的对象
// 系统内置的估值器有3个：
// IntEvaluator：以整型的形式从初始值 - 结束值 进行过渡
// FloatEvaluator：以浮点型的形式从初始值 - 结束值 进行过渡
// ArgbEvaluator：以Argb类型的形式从初始值 - 结束值 进行过渡
```

效果图：

![img](https:////upload-images.jianshu.io/upload_images/944365-21ebbc2efbb1a810.png?imageMogr2/auto-orient/strip|imageView2/2/w/954/format/webp)

FloatEvaluator

![img](https:////upload-images.jianshu.io/upload_images/944365-3b0aa80a2ec26e88.png?imageMogr2/auto-orient/strip|imageView2/2/w/852/format/webp)

IntEvaluator

- 如果上述内置的估值器无法满足需求，还可以自定义估值器
   下面将介绍如何自定义估值器（Interpolator）

------

# 4. 自定义估值器

### 4.1 本质

根据 插值器计算出当前属性值改变的百分比  & 初始值 & 结束值 来计算 当前属性具体的数值

> 如：动画进行了50%（初始值=100，结束值=200 ），那么匀速插值器计算出了当前属性值改变的百分比是50%，那么估值器则负责计算当前属性值 = 100 + （200-100）x50% = 150.

### 4.2 具体实现方式

自定义估值器需要实现 TypeEvaluator接口 & 复写`evaluate()`



```java
// 步骤1：实现TypeEvaluator接口
    public class ObjectEvaluator implements TypeEvaluator{  

// 步骤2：复写evaluate()
// 作用：估值器的计算逻辑，即写入对象动画过渡的逻辑
    @Override  
    public Object evaluate(float fraction, Object startValue, Object endValue) {  
        // 参数说明
        // fraction：表示动画完成度（根据它来计算当前动画的值），也是插值器getInterpolation()的返回值
        // startValue：动画的初始值
        // endValue：动画的结束值

        // 估值器的计算逻辑
        ... 
        
        // 返回对象动画过渡逻辑计算后的值
        // 即赋给动画属性的具体数值
        return value;  
    } 

// 特别注意
// 那么插值器的input值 和 估值器fraction有什么关系呢？
// 答：input的值决定了fraction的值：input值经过计算后传入到插值器的getInterpolation（），然后通过实现getInterpolation（）中的逻辑算法，根据input值来计算出一个返回值，而这个返回值就是fraction了
```

在学习自定义估值器前，我们先来看一个已经实现好的系统内置差值器：浮点型估值器器：`FloatEvaluator`



```java
// 步骤1：FloatEvaluator实现了TypeEvaluator接口
public class FloatEvaluator implements TypeEvaluator {  

// 步骤2：重写evaluate()
    public Object evaluate(float fraction, Object startValue, Object endValue) { 

        // 初始值过渡到结束值 的算法是：
        // 1. 用结束值减去初始值，算出它们之间的差值
        // 2. 用上述差值乘以fraction系数
        // 3. 再加上初始值，就得到当前动画的值
        float startFloat = ((Number) startValue).floatValue();  
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);  
    }  
} 
```

属性动画中的`ValueAnimator.ofInt（）` & `ValueAnimator.ofFloat（）`都具备系统内置的估值器，即`FloatEvaluator` & `IntEvaluator`，即系统已经默认实现了 **如何从初始值 过渡到 结束值 的逻辑**

但对于`ValueAnimator.ofObject（）`，从上面的工作原理可以看出并没有系统默认实现，因为对对象的动画操作复杂 & 多样，系统无法知道如何从初始对象过度到结束对象

因此，对于`ValueAnimator.ofObject（）`，我们需自定义估值器（`TypeEvaluator`）来告知系统如何进行从 初始对象 过渡到 结束对象的逻辑。

------

# 5. 实例说明

- 下面我将用实例说明 该如何自定义`TypeEvaluator`接口并通过`ValueAnimator.ofObject（）`实现动画效果

- 实现的动画效果：一个圆从一个点 移动到 另外一个点

  ![img](https:////upload-images.jianshu.io/upload_images/944365-45b817bd4ca8c119.gif?imageMogr2/auto-orient/strip|imageView2/2/w/386/format/webp)

  效果图

- 工程目录文件如下：

  ![img](https:////upload-images.jianshu.io/upload_images/944365-07fe96c57b90f28f.png?imageMogr2/auto-orient/strip|imageView2/2/w/313/format/webp)

  工程目录

### 步骤1：定义对象类

- 因为`ValueAnimator.ofObject（）`是面向对象操作的，所以需要自定义对象类。
- 本例需要操作的对象是 **圆的点坐标**
   *Point.java*



```cpp
public class Point {

    // 设置两个变量用于记录坐标的位置
    private float x;
    private float y;

    // 构造方法用于设置坐标
    public Point(float x, float y) {
        this.x = x;
        this.y = y;
    }

    // get方法用于获取坐标
    public float getX() {
        return x;
    }

    public float getY() {
        return y;
    }
}
```

## 步骤2：根据需求实现TypeEvaluator接口

- 实现`TypeEvaluator`接口的目的是自定义如何 从初始点坐标 过渡 到结束点坐标；

- 本例实现的是一个从左上角到右下角的坐标过渡逻辑。

  ![img](https:////upload-images.jianshu.io/upload_images/944365-45b817bd4ca8c119.gif?imageMogr2/auto-orient/strip|imageView2/2/w/386/format/webp)

  效果图

*PointEvaluator.java*



```java
// 实现TypeEvaluator接口
public class PointEvaluator implements TypeEvaluator {

    // 复写evaluate（）
    // 在evaluate（）里写入对象动画过渡的逻辑
    @Override
    public Object evaluate(float fraction, Object startValue, Object endValue) {

        // 将动画初始值startValue 和 动画结束值endValue 强制类型转换成Point对象
        Point startPoint = (Point) startValue;
        Point endPoint = (Point) endValue;

        // 根据fraction来计算当前动画的x和y的值
        float x = startPoint.getX() + fraction * (endPoint.getX() - startPoint.getX());
        float y = startPoint.getY() + fraction * (endPoint.getY() - startPoint.getY());
        
        // 将计算后的坐标封装到一个新的Point对象中并返回
        Point point = new Point(x, y);
        return point;
    }

}
```

- 上面步骤是根据需求自定义`TypeEvaluator`的实现
- 下面将讲解如何通过对 `Point` 对象进行动画操作，从而实现整个自定义View的动画效果。

### 步骤3：将属性动画作用到自定义View当中

*MyView.java*



```java
/**
 * Created by Carson_Ho on 17/4/18.
 */
public class MyView extends View {
    // 设置需要用到的变量
    public static final float RADIUS = 70f;// 圆的半径 = 70
    private Point currentPoint;// 当前点坐标
    private Paint mPaint;// 绘图画笔
    

    // 构造方法(初始化画笔)
    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
        // 初始化画笔
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.BLUE);
    }

    // 复写onDraw()从而实现绘制逻辑
    // 绘制逻辑:先在初始点画圆,通过监听当前坐标值(currentPoint)的变化,每次变化都调用onDraw()重新绘制圆,从而实现圆的平移动画效果
    @Override
    protected void onDraw(Canvas canvas) {
        // 如果当前点坐标为空(即第一次)
        if (currentPoint == null) {
            currentPoint = new Point(RADIUS, RADIUS);
            // 创建一个点对象(坐标是(70,70))

            // 在该点画一个圆:圆心 = (70,70),半径 = 70
            float x = currentPoint.getX();
            float y = currentPoint.getY();
            canvas.drawCircle(x, y, RADIUS, mPaint);


 // (重点关注)将属性动画作用到View中
            // 步骤1:创建初始动画时的对象点  & 结束动画时的对象点
            Point startPoint = new Point(RADIUS, RADIUS);// 初始点为圆心(70,70)
            Point endPoint = new Point(700, 1000);// 结束点为(700,1000)

            // 步骤2:创建动画对象 & 设置初始值 和 结束值
            ValueAnimator anim = ValueAnimator.ofObject(new PointEvaluator(), startPoint, endPoint);
            // 参数说明
            // 参数1：TypeEvaluator 类型参数 - 使用自定义的PointEvaluator(实现了TypeEvaluator接口)
            // 参数2：初始动画的对象点
            // 参数3：结束动画的对象点

            // 步骤3：设置动画参数
            anim.setDuration(5000);
            // 设置动画时长

// 步骤3：通过 值 的更新监听器，将改变的对象手动赋值给当前对象
// 此处是将 改变后的坐标值对象 赋给 当前的坐标值对象
            // 设置 值的更新监听器
            // 即每当坐标值（Point对象）更新一次,该方法就会被调用一次
            anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    currentPoint = (Point) animation.getAnimatedValue();
                    // 将每次变化后的坐标值（估值器PointEvaluator中evaluate（）返回的Piont对象值）到当前坐标值对象（currentPoint）
                    // 从而更新当前坐标值（currentPoint）

// 步骤4：每次赋值后就重新绘制，从而实现动画效果
                    invalidate();
                    // 调用invalidate()后,就会刷新View,即才能看到重新绘制的界面,即onDraw()会被重新调用一次
                    // 所以坐标值每改变一次,就会调用onDraw()一次
                }
            });

            anim.start();
            // 启动动画


        } else {
            // 如果坐标值不为0,则画圆
            // 所以坐标值每改变一次,就会调用onDraw()一次,就会画一次圆,从而实现动画效果

            // 在该点画一个圆:圆心 = (30,30),半径 = 30
            float x = currentPoint.getX();
            float y = currentPoint.getY();
            canvas.drawCircle(x, y, RADIUS, mPaint);
        }
    }


}
```

### 步骤4：在布局文件加入自定义View空间

*activity_main.xml*



```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="scut.carson_ho.valueanimator_ofobject.MainActivity">

    <scut.carson_ho.valueanimator_ofobject.MyView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
         />
</RelativeLayout>
```

### 步骤5：在主代码文件设置显示视图

*MainActivity.java*



```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

## 效果图

![img](https:////upload-images.jianshu.io/upload_images/944365-45b817bd4ca8c119.gif?imageMogr2/auto-orient/strip|imageView2/2/w/386/format/webp)

效果图

### 源码地址

[Carson_Ho的Github地址](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FCarson-Ho%2FPropertyAnimator_ofObject)

------

# 6. 与插值器的区别

估值器和插值器很多人容易混淆，具体区别如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-b753525db783e039.png?imageMogr2/auto-orient/strip|imageView2/2/w/1140/format/webp)

# 7. 总结

- 本文对`Android` 动画中的估值器使用进行了详细分析，相信通过本文你已经能实现复杂的动画效果



# 参考

[Android动画：深入解析插值器(TypeEvaluator) (含实例教学)](https://www.jianshu.com/p/ab5785f017b2)