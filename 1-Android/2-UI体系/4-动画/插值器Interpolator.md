# 前言

- 动画的使用 是 `Android` 开发中常用的知识
- 可是动画的**种类繁多、使用复杂**，每当需要 **采用自定义动画 实现 复杂的动画效果**时，很多开发者就显得束手无策
- Android中 补间动画 & 属性动画实现动画的原理是：

![img](https:////upload-images.jianshu.io/upload_images/944365-62e601917ece2525.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

实现原理

其中，步骤2中的 插值器（`Interpolator`）和估值器（`TypeEvaluator`）是实现 复杂动画效果的关键；本文主要讲解 将详细讲解 插值器（`Interpolator`），通过阅读本文你将能轻松实现复杂的动画效果

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-a45587dafc18daa3.png?imageMogr2/auto-orient/strip|imageView2/2/w/868/format/webp)

# 1. 简介

- 定义：Android实现动画效果中的一个辅助接口
- 作用：设置 属性值 从初始值过渡到结束值 的变化规律

> 1. 如匀速、加速 & 减速 等等
> 2. 即确定了 动画效果变化的模式，如匀速变化、加速变化 等等

# 2. 应用场景

实现非线性运动的动画效果。非线性运动是指动画改变的速率不是一成不变的，如加速、减速运动的动画效果。

# 3. 具体使用

插值器在动画的使用有两种方式：在XML / Java代码中设置：

```php
/*
 * 使用方式1：xml
 * 主要是设置插值器属性 android:interpolator
 */
 <?xml version="1.0" encoding="utf-8"?>
    <scale xmlns:android="http://schemas.android.com/apk/res/android"

        // 通过资源ID设置插值器
        android:interpolator="@android:anim/overshoot_interpolator"
        android:duration="3000"
        android:fromXScale="0.0"
        android:fromYScale="0.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="2"
        android:toYScale="2" />
 >

/*
 * 使用方式2：java
 */
// 步骤1:创建 需要设置动画的 视图View
Button mButton = (Button) findViewById(R.id.Button);
// 步骤2：创建透明度动画的对象 & 设置动画效果
Animation alphaAnimation = new AlphaAnimation(1,0);
// 步骤3：创建对应的插值器类对象
alphaAnimation.setDuration(3000);
Interpolator overshootInterpolator = new OvershootInterpolator();
// 步骤4：给动画设置插值器
alphaAnimation.setInterpolator(overshootInterpolator);
// 步骤5：播放动画
mButton.startAnimation(alphaAnimation);
```

- 那么使用插值器时的资源ID是什么呢？即有哪些类型的插值器可供我们使用呢？
- 插值器分为两种：Android内置默认 & 自定义，下面我将详细介绍

# 4. 系统内置插值器

### 4.1 类型

`Android`内置了 9 种内置的插值器实现：

![img](https:////upload-images.jianshu.io/upload_images/944365-dccc87ed4ad77156.png?imageMogr2/auto-orient/strip|imageView2/2/w/1144/format/webp)

### 4.2 具体使用

- 当在XML文件设置插值器时，只需传入对应的插值器资源ID即可
- 当在Java代码设置插值器时，只需创建对应的插值器对象即可

> 系统默认的插值器是`AccelerateDecelerateInterpolator`，即先加速后减速

### 4.3 效果图

![img](https:////upload-images.jianshu.io/upload_images/944365-0a9435149480729e.gif?imageMogr2/auto-orient/strip|imageView2/2/w/397/format/webp)

- 使用`Android`内置的插值器能满足大多数的动画需求
- 如果上述9个插值器无法满足需求，还可以自定义插值器
- 下面将介绍如何自定义插值器（`Interpolator`）

# 5. 自定义插值器

### 5.1 本质

根据动画的进度（0%-100%）计算出当前属性值改变的百分比

### 5.2 实现方式

自定义插值器需要实现Interpolator或TimeInterpolator接口，并复写getInterpolation()方法

> 1. 补间动画 实现 `Interpolator`接口；属性动画实现`TimeInterpolator`接口
> 2. `TimeInterpolator`接口是属性动画中新增的，用于兼容`Interpolator`接口，这使得所有过去的`Interpolator`实现类都可以直接在属性动画使用

Interpolator接口和TimeInterpolator接口说明如下：

```csharp
// Interpolator接口
public interface Interpolator {  

    // 内部只有一个方法：getInterpolation()
     float getInterpolation(float input) {  
        // 参数说明
        // input值值变化范围是0-1，且随着动画进度（0% - 100% ）均匀变化
        // 即动画开始时，input值 = 0；动画结束时input = 1
        // 而中间的值则是随着动画的进度（0% - 100%）在0到1之间均匀增加
        
      ...// 插值器的计算逻辑

      return xxx；
      // 返回的值就是用于估值器继续计算的fraction值，下面会详细说明
    }  

// TimeInterpolator接口
// 同上
public interface TimeInterpolator {  
  
    float getInterpolation(float input){
         ...
    };  
}  
```

从上面可以看出，自定义插值器的关键在于：**对input值根据动画的进度(0%-100%)通过逻辑计算从而计算出当前属性值改变的百分比**。先来看两个已经实现好的系统内置差值器：

- 匀速插值器：`LinearInterpolator`
- 先加速再减速 插值器：`AccelerateDecelerateInterpolator`

```java
/*
 * 匀速差值器：LinearInterpolator
 */
@HasNativeInterpolator  
public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {  
   
    ... // 仅贴出关键代码

    public float getInterpolation(float input) {  
        return input;  
        // 没有对input值进行任何逻辑处理，直接返回
        // 即input值 = fraction值
        // 因为input值是匀速增加的，因此fraction值也是匀速增加的，所以动画的运动情况也是匀速的，所以是匀速插值器
    }  

/*
 * 先加速再减速 差值器:AccelerateDecelerateInterpolator
 */
@HasNativeInterpolator  
public class AccelerateDecelerateInterpolator implements Interpolator, NativeInterpolatorFactory {  
      
    ... // 仅贴出关键代码
    
    public float getInterpolation(float input) {  
        return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
        // input的运算逻辑如下：
        // 使用了余弦函数，因input的取值范围是0到1，那么cos函数中的取值范围就是π到2π。
        // 而cos(π)的结果是-1，cos(2π)的结果是1
        // 所以该值除以2加上0.5后，getInterpolation()方法最终返回的结果值还是在0到1之间。只不过经过了余弦运算之后，最终的结果不再是匀速增加的了，而是经历了一个先加速后减速的过程
        // 所以最终，fraction值 = 运算后的值 = 先加速后减速
        // 所以该差值器是先加速再减速的
    }  
}
```

### 5.3 实例说明

下面，我将写一个自定义`Interpolator`：先减速后加速。

```java
/*
 * 步骤1：根据需求实现Interpolator接口
 * DecelerateAccelerateInterpolator.java
 */
public class DecelerateAccelerateInterpolator implements TimeInterpolator {
    
    @Override
    public float getInterpolation(float input) {
        float result;
        if (input <= 0.5) {
            result = (float) (Math.sin(Math.PI * input)) / 2;
            // 使用正弦函数来实现先减速后加速的功能，逻辑如下：
            // 因为正弦函数初始弧度变化值非常大，刚好和余弦函数是相反的
            // 随着弧度的增加，正弦函数的变化值也会逐渐变小，这样也就实现了减速的效果。
            // 当弧度大于π/2之后，整个过程相反了过来，现在正弦函数的弧度变化值非常小，渐渐随着弧度继续增加，变化值越来越大，弧度到π时结束，这样从0过度到π，也就实现了先减速后加速的效果
        } else {
            result = (float) (2 - Math.sin(Math.PI * input)) / 2;
        }
        return result;
        // 返回的result值 = 随着动画进度呈先减速后加速的变化趋势
    }
}

/*
 * 步骤2：设置使用
 * MainActivity.java
 */
 // 1. 创建动画作用对象：此处以Button为例
 mButton = (Button) findViewById(R.id.Button);

 // 2. 获得当前按钮的位置
float curTranslationX = mButton.getTranslationX();

// 3. 创建动画对象 & 设置动画
ObjectAnimator animator = ObjectAnimator.ofFloat(mButton, "translationX", curTranslationX, 300,curTranslationX);
// 表示的是:
// 动画作用对象是mButton
// 动画作用的对象的属性是X轴平移
// 动画效果是:从当前位置平移到 x=1500 再平移到初始位置

// 4. 设置步骤1中设置好的插值器：先减速后加速
animator.setInterpolator(new DecelerateAccelerateInterpolator());

// 5. 启动动画
animator.start();
```

## 效果图

![img](https:////upload-images.jianshu.io/upload_images/944365-60cf56d176981da0.gif?imageMogr2/auto-orient/strip|imageView2/2/w/388/format/webp)

差值器.gif

# 6. 与估值器的区别

估值器和插值器很多人容易混淆，具体区别如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-49649401b1ae6ece.png?imageMogr2/auto-orient/strip|imageView2/2/w/639/format/webp)

# 7. 总结

- 本文对`Android` 动画中的插值器进行了详细分析，相信通过本文你已经能实现复杂的动画效果



# 参考

[Android动画：手把手带你深入了解神秘的插值器(Interpolator)](https://www.jianshu.com/p/2f19fe1e3ca1)