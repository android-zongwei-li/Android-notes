# 前言

- `V- Layout` 是阿里出品的基础 UI 框架，**用于快速实现页面的复杂布局**，在手机天猫 `Android`版 内广泛使用

![img](https:////upload-images.jianshu.io/upload_images/944365-2d26a7d75d523003.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162/format/webp)

电商图

- 让人激动的是，在上个月`V- Layout`终于在Github上开源！

> [Github - alibaba - vlayout ](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fvlayout)

![img](https:////upload-images.jianshu.io/upload_images/944365-909cf48deb29dc75.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Github

- 今天，我对`V- Layout`进行了详细分析，我将献上一份 `V- Layout`的使用攻略 & 源码分析，希望你们会喜欢。

> 阅读本文前请先看：
>  [Android Tangram模型：连淘宝、天猫都在用的UI框架模型你一定要懂](https://www.jianshu.com/p/b339c2d2d500)
>  Carson带你学Android开源库系列文章：
>  [Carson带你学Android：主流开源图片加载库对比(UIL、Picasso、Glide、Fresco)](https://www.jianshu.com/p/97994c9693f9)
>  [Carson带你学Android：主流开源网络请求库对比(Volley、OkHttp、Retrofit)](https://www.jianshu.com/p/050c6db5af5a)
>  [Carson带你学Android：网络请求库Retrofit使用教程](https://www.jianshu.com/p/a3e162261ab6)
>  [Carson带你学Android：网络请求库Retrofit源码分析](https://www.jianshu.com/p/a3e162261ab6)
>  [Carson带你学Android：图片加载库Glide使用教程](https://www.jianshu.com/p/c3a5518b58b2)
>  [Carson带你学Android：图片加载库Glide源码分析](https://www.jianshu.com/p/c3a5518b58b2)
>  [Carson带你学Android：V-Layout，淘宝、天猫都在用的UI框架，赶紧用起来吧！](https://www.jianshu.com/p/6b658c8802d1)

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-d51e75257764dc88.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

目录

------

# 1. 为什么要使用 V - Layout

在讲解 `V - Layout` 前，我们先来搞懂一个问题：为什么要使用 `V - Layout`

### 1.1 背景

- `Android`中 `UI` 性能消耗主要来自于两个方面：

1. 布局层次嵌套导致多重 `measure/layout`
2. `View`控件的创建和销毁

- 为了解决上述问题，现有的解决方式是：
  1. 少用嵌套布局
  2. 使用 `ListView/GirdView/RecyclerView`等基础空间来处理`View`的回收与复用。

**但是，很多时候我们需要在一个长列表下做多种类型的布局来分配各种元素,，特别是电商平台首页等页面，布局元素结构更加复杂多样。如下图：**

![img](https:////upload-images.jianshu.io/upload_images/944365-2d26a7d75d523003.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162/format/webp)

电商图



此时的解决方案有所变化：不采用子`View`的复用，只采用一个主要的复用容器（如`ListView` 或 `RecyclerView`+`LinearLayoutManager`），然后在其中使用嵌套方式直接用各个组件进行拼接，减少了复用的能力

### 1.2 问题

这种做法还是会损失Android应用的性能。

### 1.3 解决方案

- 通过自定义 `LayoutManager` 管理所有的布局类型
- 即阿里出品的基础 UI 框架项目 `VirtualLayout`就是采用该方式来解决上述问题

------

# 2. 简介

- 定义：`VirtualLayout`是阿里出品的基础 UI 框架项目
- 作用：快速实现复杂的布局格式的混排 & 通过组件回收提高性能

> - 基于 `RecyclerView` & `LayoutManager`扩展
> - 目前已在Github开源：[Github - alibaba - vlayout ](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fvlayout)

![img](https:////upload-images.jianshu.io/upload_images/944365-8e7b2d8aac6c49f4.gif?imageMogr2/auto-orient/strip|imageView2/2/w/200/format/webp)

效果图

------

# 3. 应用场景

- 复杂的布局格式混排，如：浮动布局、栏格布局、通栏布局、一拖N布局、瀑布流布局，还可以组合使用这些布局
- 具体场景是：如电商平台首页、活动页等等

> V - Layout 目前已在手机天猫 & 淘宝 Android 版内广泛使用

![img](https:////upload-images.jianshu.io/upload_images/944365-2d26a7d75d523003.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162/format/webp)

实际应用效果图

------

# 4. 原理解析

`V - Layout`的本质原理是：通过自定义一个`VirtualLayoutManager`（继承自 LayoutManager），用于管理一系列`LayoutHelper`，将具体的布局能力交给`LayoutHelper`来完成，从而 **快速实现组合布局** 的需求。

> 1. 每个 `LayoutHelper`负责 页面某一个范围内的布局
>
> 2. ```
>    V - Layout
>    ```
>
>    默认实现了10种默认布局：（对应同名的LayoutHelper）
>
>    ![img](https:////upload-images.jianshu.io/upload_images/944365-83f42eef8ff6f3e3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
>
>    布局类型

### 4.1 源码类说明

- `V - Layout`的源码类图如下：

  ![img](https:////upload-images.jianshu.io/upload_images/944365-0e2b7013328fabd4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  UML类图

  

- 具体类说明

![img](https:////upload-images.jianshu.io/upload_images/944365-17695dd3cd7f6a2d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- `V - Layout`默认实现了10种默认布局：（对应同名的LayoutHelper）

![img](https:////upload-images.jianshu.io/upload_images/944365-83f42eef8ff6f3e3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

布局类型

下面会进行详细介绍。

- 特别注意：

1. 每一种`LayoutHelper`负责布局一批组件范围内的组件，不同组件范围内的组件之间，如果类型相同，可以在滑动过程中回收复用。因此回收粒度比较细，且可以跨布局类型复用。
2. 支持扩展外部：即注册新的`LayoutHelper`，实现特殊的布局方式。下面会详细说明

介绍完类之后，我将详细分析  `V - Layout`的工作流程。

------

### 4.2 工作流程

- `V - Layout`的工作流程分为 初始化 & 布局流程。如下图：

![img](https:////upload-images.jianshu.io/upload_images/944365-b935b113df2736dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

工作流程

- 下面我将对初始化 & 布局流程作详细分析。

#### 4.2.1 初始化

- 在使用 `V - layout` 快速实现复杂布局前，需要先做一系列的初始化工作。

> 初始化流程与使用普通的 RecyclerView + LayoutManager 初始化流程基本一致：Vlayout的使用者

![img](https:////upload-images.jianshu.io/upload_images/944365-e507ce4fe30a7496.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

初始化流程

- 此处的初始化 实际上 就是 使用者在使用 `V - layout` 时需要做的初始化工作。
- 此处主要先讲解下数据列表的获取：本质上，是对页面实体 进行 卡片 & 组件的拆解，形成一个位置列表

![img](https:////upload-images.jianshu.io/upload_images/944365-ad486ce2686fdfde.png?imageMogr2/auto-orient/strip|imageView2/2/w/830/format/webp)

示意图

- 其他初始化步骤将在下面实例讲解进行详细说明

------

#### 4.2.2 具体布局流程

- 当完成初始化工作后，每当用户刚打开页面第一次渲染布局 或 用户滑动页面时，都会进行一次布局流程
- 布局流程的本质是：自定义 `VirtualLayoutManager`持续获取页面状态，并通过`LayoutHelperFinder`找到对应的`LayoutHelper`从而实现对应的布局逻辑，从而快速实现组合布局 的需求
- 具体流程如下

![img](https:////upload-images.jianshu.io/upload_images/944365-ff80669ee8cb51b0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

布局流程

------

### 总结

下面用一张图总结 `V - Layout` 的原理 & 工作流程

![img](https:////upload-images.jianshu.io/upload_images/944365-e668b1f8dc31ac9b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

原理 & 工作流程



在讲完原理后，接下来我将如何使用 `V - Layout`。

------

# 5. 使用步骤

- `V - Layout`的使用其实就是上面说的初始化工作
- 使用步骤如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-2d549add7ce7d08a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

使用步骤

下面我将对每个步骤进行详细说明。

### 步骤1：创建RecyclerView & VirtualLayoutManager 对象并进行绑定



```cpp
recyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);
        // 创建RecyclerView对象
        
        VirtualLayoutManager layoutManager = new VirtualLayoutManager(this);
        // 创建VirtualLayoutManager对象
        // 同时内部会创建一个LayoutHelperFinder对象，用来后续的LayoutHelper查找

        recyclerView.setLayoutManager(layoutManager);
        // 将VirtualLayoutManager绑定到recyclerView
```

### 步骤2：设置回收复用池大小

如果一屏内相同类型的 View 个数比较多，需要设置一个合适的大小，防止来回滚动时重新创建 View）



```cpp
// 设置组件复用回收池
        RecyclerView.RecycledViewPool viewPool = new RecyclerView.RecycledViewPool();
        recyclerView.setRecycledViewPool(viewPool);
        viewPool.setMaxRecycledViews(0, 10);
```

### 步骤3：设置Adapter

设置 V - Layout的Adapter有两种方式:

- 方式1：继承 自 `DelegateAdapter`
- 方式2：继承 自 `VirtualLayoutAdapter`
   下面会进行详细说明：

**方式1：继承 自 DelegateAdapter**

- 定义：`DelegateAdapter`是V - Layout专门为管理 `LayoutHelper`定制的 `Adapter`

> 继承自VirtualLayoutAdapter

- 作用：通过管理不同布局的Adapter，继而管理不同的 `LayoutHelper`，从而实现使用不同组合布局

> 1. 特别注意：虽不可直接绑定LayoutHelper，但是它内部有一个继承自RecyclerView.Adapter的内部类Adapter可以绑定LayoutHelper；
> 2. 即通过一个List把绑定好的Adapter打包起来，再放去DelegateAdapter，这样就可以实现组合使用不同的布局

- 具体做法：

> 1. 写法与复写系统自带的Adapter非常类似：只比系统自带的RecyclerAdapter需要多重载onCreateLayoutHelper方法，其余类似
> 2. 关于Android系统自带的RecyclerAdapter的使用具体请看我写的文章[Android开发：ListView、AdapterView、RecyclerView全面解析
>     ](https://www.jianshu.com/p/4e8e4fd13cf7)



```java
public class MyAdapter extends DelegateAdapter.Adapter<MyAdapter.MainViewHolder> {

// 比系统自带的RecyclerAdapter需要多重载onCreateLayoutHelper（）

    @Override
    public LayoutHelper onCreateLayoutHelper() {
        return layoutHelper;
    }

... // 其余写法与复写系统自带的Adapter相同
}
```

**方式2：继承 自 `VirtualLayoutAdapter`**

- 定义：当需要实现复杂需求时， 可以通过继承`VirtualLayoutAdapter`从而实现自定义Adapter
- 具体使用



```java
public class MyAdapter extends VirtualLayoutAdapter {
   ...// 自定义Adapter逻辑
}
```

### 步骤4：根据数据列表，创建对应的LayoutHelper

- 系统以封装好以下布局类型（对应同名的LayoutHelper）

![img](https:////upload-images.jianshu.io/upload_images/944365-83f42eef8ff6f3e3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

布局类型

- 具体使用如下：

#### 1. 线性布局（LinearLayoutHelper）

- 布局说明：布局子元素（`Item`）以线性排布的布局

![img](https:////upload-images.jianshu.io/upload_images/944365-b148e1da8fcf0943.png?imageMogr2/auto-orient/strip|imageView2/2/w/760/format/webp)

示意图

- 具体使用



```cpp
/**
    设置线性布局
       */
        LinearLayoutHelper linearLayoutHelper = new LinearLayoutHelper();
        // 创建对应的LayoutHelper对象

 // 所有布局的公共属性（属性会在下面详细说明）
        linearLayoutHelper.setItemCount(4);// 设置布局里Item个数
        linearLayoutHelper.setPadding(10,10,10,10);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        linearLayoutHelper.setMargin(10,10,10,10);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        linearLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        linearLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

 // linearLayoutHelper特有属性
        linearLayoutHelper.setDividerHeight(1); // 设置每行Item的距离
     
```

### 1. 所有布局的共有属性说明：

**a. setItemCount属性**

- 作用：设置整个布局里的Item数量

> 如设置的Item总数如与Adapter的getItemCount()方法返回的数量不同，会取决于后者

![img](https:////upload-images.jianshu.io/upload_images/944365-deed026d0b6de8b7.png?imageMogr2/auto-orient/strip|imageView2/2/w/434/format/webp)

示意图

- 具体使用



```csharp
// 接口示意
    public void setItemCount(int Count)

// 具体使用
        linearLayoutHelper.setItemCount(4);
```

**b. Adding & Margin属性**

- 定义：都是边距的含义，但二者边距的定义不同：

  1. `Padding`：是 `LayoutHelper` 的子元素相对 `LayoutHelper` 边缘的距离；

  2. ```
     Margin
     ```

     ：是 

     ```
     LayoutHelper
     ```

      边缘相对父控件（即

     ```
     RecyclerView
     ```

     ）的距离。具体如下图：

     ![img](https:////upload-images.jianshu.io/upload_images/944365-79456f4260176608.png?imageMogr2/auto-orient/strip|imageView2/2/w/310/format/webp)

     示意图

- 具体使用



```csharp
// 接口示意
    public void setPadding(int leftPadding, int topPadding, int rightPadding, int bottomPadding)
    public void setMargin(int leftMargin, int topMargin, int rightMargin, int bottomMargin)

// 具体使用
        linearLayoutHelper.setPadding(10,10,10,10);
        // 设置LayoutHelper的子元素相对LayoutHelper边缘的距离

        linearLayoutHelper.setMargin(10,10,10,10);
        // 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
```

------

**c. bgColor属性**

- 作用：设置布局背景颜色
- 具体使用：



```csharp
// 接口示意
public void setBgColor(int bgColor)

// 具体使用
linearLayoutHelper.setBgColor(Color.YELLOW);
```

**d. aspectRatio属性**

- 作用：设置布局内每行布局的宽与高的比。如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-bff7b3671136448a.png?imageMogr2/auto-orient/strip|imageView2/2/w/466/format/webp)

示意图

- 具体使用



```cpp
// 接口示意
public void setAspectRatio(float aspectRatio);
// LayoutHelper定义的aspectRatio

((VirutalLayoutManager.LayoutParams) layoutParams).mAspectRatio
// 视图的LayoutParams定义的aspectRatio
// 在LayoutHelper计算出视图宽度之后，用来确定视图高度时使用的，它会覆盖通过LayoutHelper的aspectRatio计算出来的视图高度，因此具备更高优先级。

// 具体使用
        linearLayoutHelper.setAspectRatio(6);
       
```

### 2. LinearLayoutHelper布局的特有属性说明

**a. dividerHeight属性**

- 作用：设置每行Item之间的距离

> 设置的间隔会与RecyclerView的addItemDecoration（）添加的间隔叠加

![img](https:////upload-images.jianshu.io/upload_images/944365-34838810a89b7c5b.png?imageMogr2/auto-orient/strip|imageView2/2/w/418/format/webp)

示意图

- 具体使用



```csharp
// 接口示意
public void setDividerHeight(int dividerHeight)

// 具体使用
 linearLayoutHelper.setDividerHeight(1);
```

------

### 2. 网格布局（GridLayout）

- 布局说明：布局里的Item以网格的形式进行排列

![img](https:////upload-images.jianshu.io/upload_images/944365-c394c6dda149c1a3.png?imageMogr2/auto-orient/strip|imageView2/2/w/744/format/webp)

示意图

- 具体使用



```csharp
        /**
        设置Grid布局
        */
        GridLayoutHelper gridLayoutHelper = new GridLayoutHelper(3);
        // 在构造函数设置每行的网格个数

        // 公共属性
        gridLayoutHelper.setItemCount(6);// 设置布局里Item个数
        gridLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        gridLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        gridLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        gridLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

        // gridLayoutHelper特有属性（下面会详细说明）
        gridLayoutHelper.setWeights(new float[]{40, 30, 30});//设置每行中 每个网格宽度 占 每行总宽度 的比例
        gridLayoutHelper.setVGap(20);// 控制子元素之间的垂直间距
        gridLayoutHelper.setHGap(20);// 控制子元素之间的水平间距
        gridLayoutHelper.setAutoExpand(false);//是否自动填充空白区域
        gridLayoutHelper.setSpanCount(3);// 设置每行多少个网格
        // 通过自定义SpanSizeLookup来控制某个Item的占网格个数
        gridLayoutHelper.setSpanSizeLookup(new GridLayoutHelper.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                if (position > 7 ) {
                    return 3;
                    // 第7个位置后,每个Item占3个网格
                }else {
                    return 2;
                    // 第7个位置前,每个Item占2个网格
                }
            }
        });
```

##### GridLayoutHelper布局的特有属性说明

**a. weights属性**

- 作用：设置每行中每个网格宽度占每行总宽度的比例

> 1. 默认情况下，每行中每个网格中的宽度相等
> 2. `weights`属性是一个float数组，每一项代表当个网格占每行总宽度的百分比；总和是100，否则布局会超出容器宽度；
> 3. 如果布局中有4列，那么weights的长度也应该是4；长度大于4，多出的部分不参与宽度计算；如果小于4，不足的部分默认平分剩余的空间。

![img](https:////upload-images.jianshu.io/upload_images/944365-f0866c8c86820731.png?imageMogr2/auto-orient/strip|imageView2/2/w/666/format/webp)

示意图

- 具体使用



```csharp
// 接口示意
public void setWeights(float[] weights)

// 具体使用
gridLayoutHelper.setWeights(new float[]{40, 30, 30});
```

**b. vGap、hGap属性**

- 作用：分别控制子元素之间的垂直间距 和 水平间距。

![img](https:////upload-images.jianshu.io/upload_images/944365-ba07f6be1226a639.png?imageMogr2/auto-orient/strip|imageView2/2/w/342/format/webp)

示意图

- 具体使用



```csharp
// 接口示意
    public void setHGap(int hGap)
    public void setVGap(int vGap)

// 具体使用
    gridLayoutHelper.setVGap(20);// 控制子元素之间的垂直间距
    gridLayoutHelper.setHGap(20);// 控制子元素之间的水平间距
```

**c. spanCount、spanSizeLookup属性**

- 作用：
  1. `spanCount`：设置每行多少个网格
  2. `spanSizeLookup`：设置每个 `Item`占用多少个网格（默认= 1）

![img](https:////upload-images.jianshu.io/upload_images/944365-939381edcc501975.png?imageMogr2/auto-orient/strip|imageView2/2/w/909/format/webp)

示意图

- 具体使用



```java
// 接口示意
public void setSpanCount(int spanCount)
public void setSpanSizeLookup(SpanSizeLookup spanSizeLookup)

// 具体使用
        gridLayoutHelper.setSpanCount(5);// 设置每行多少个网格

        // 通过自定义SpanSizeLookup来控制某个Item的占网格个数
        gridLayoutHelper.setSpanSizeLookup(new GridLayoutHelper.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                if (position > 7 ) {
                    return 3;
                    // 第7个位置后,每个Item占3个网格
                }else {
                    return 2;
                    // 第7个位置前,每个Item占2个网格
                }
            }
        });
```

**d. autoExpand属性**

- 作用：当一行里item的个数 < （每行网格列数 - spanCount值/  每个Item占有2个网格-setSpanSizeLookup ）时，是否自动填满空白区域

> 1. 若autoExpand=true，那么视图的总宽度会填满可用区域；
> 2. 否则会在屏幕上留空白区域。

![img](https:////upload-images.jianshu.io/upload_images/944365-73565464f158d950.png?imageMogr2/auto-orient/strip|imageView2/2/w/752/format/webp)

示意图

- 具体使用



```java
// 接口示意
public void setAutoExpand(boolean isAutoExpand)

// 具体使用
gridLayoutHelper.setAutoExpand(false);
```

------

# 3. 固定布局（FixLayoutHelper）

- 布局说明：布局里的Item 固定位置

> 固定在屏幕某个位置，且不可拖拽 & 不随页面滚动而滚动。如下图：（左上角）

![img](https:////upload-images.jianshu.io/upload_images/944365-b0f5cbd50e91b16a.gif?imageMogr2/auto-orient/strip|imageView2/2/w/388/format/webp)

固定布局

- 具体使用



```cpp
/**
         设置固定布局
         */

        FixLayoutHelper fixLayoutHelper = new FixLayoutHelper(FixLayoutHelper.TOP_LEFT,40,100);
        // 参数说明:
        // 参数1:设置吸边时的基准位置(alignType) - 有四个取值:TOP_LEFT(默认), TOP_RIGHT, BOTTOM_LEFT, BOTTOM_RIGHT
        // 参数2:基准位置的偏移量x
        // 参数3:基准位置的偏移量y


        // 公共属性
        fixLayoutHelper.setItemCount(1);// 设置布局里Item个数
        // 从设置Item数目的源码可以看出，一个FixLayoutHelper只能设置一个
//        @Override
//        public void setItemCount(int itemCount) {
//            if (itemCount > 0) {
//                super.setItemCount(1);
//            } else {
//                super.setItemCount(0);
//            }
//        }
        fixLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        fixLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        fixLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        fixLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

        // fixLayoutHelper特有属性
        fixLayoutHelper.setAlignType(FixLayoutHelper.TOP_LEFT);// 设置吸边时的基准位置(alignType)
        fixLayoutHelper.setX(30);// 设置基准位置的横向偏移量X
        fixLayoutHelper.setY(50);// 设置基准位置的纵向偏移量Y
```

##### FixLayoutHelper特有属性说明

**a. AlignType、x、y属性**

- 作用：

  1. alignType：吸边基准类型

  > 共有4个取值：TOP_LEFT(默认), TOP_RIGHT, BOTTOM_LEFT, BOTTOM_RIGHT，具体请看下面示意图

  1. x：基准位置的横向偏移量
  2. y：基准位置的纵向偏移量

![img](https:////upload-images.jianshu.io/upload_images/944365-4eb84390fc9c7ede.png?imageMogr2/auto-orient/strip|imageView2/2/w/974/format/webp)

示意图

- 作用对象：FixLayoutHelper, ScrollFixLayoutHelper, FloatLayoutHelper的属性



```csharp
// 接口示意
public void setAlignType(int alignType)
public void setX(int x)
public void setY(int y)

// 具体使用
fixLayoutHelper.setAlignType(FixLayoutHelper.TOP_LEFT);
fixLayoutHelper.setX(30);
fixLayoutHelper.setY(50);
```

------

# 4. 可选显示的固定布局（ScrollFixLayoutHelper）

- 布局说明：布局里的Item 固定位置

> 1. 固定在屏幕某个位置，且不可拖拽 & 不随页面滚动而滚动（继承自固定布局（FixLayoutHelper））
> 2. 唯一不同的是，可以自由设置该Item什么时候显示（到顶部显示 / 到底部显示），可如下图：（左上角）
> 3. 需求场景：到页面底部显示”一键到顶部“的按钮功能

以下示意图为：滑动到底部，布局才在左上角显示

![img](https:////upload-images.jianshu.io/upload_images/944365-76aba04ab12b276c.gif?imageMogr2/auto-orient/strip|imageView2/2/w/386/format/webp)

滑动到底部才在左上角显示

- 具体使用



```cpp
/**
         设置可选固定布局
         */

        ScrollFixLayoutHelper scrollFixLayoutHelper = new ScrollFixLayoutHelper(ScrollFixLayoutHelper.TOP_RIGHT,0,0);
        // 参数说明:
        // 参数1:设置吸边时的基准位置(alignType) - 有四个取值:TOP_LEFT(默认), TOP_RIGHT, BOTTOM_LEFT, BOTTOM_RIGHT
        // 参数2:基准位置的偏移量x
        // 参数3:基准位置的偏移量y

        // 公共属性
        scrollFixLayoutHelper.setItemCount(1);// 设置布局里Item个数
        // 从设置Item数目的源码可以看出，一个FixLayoutHelper只能设置一个
//        @Override
//        public void setItemCount(int itemCount) {
//            if (itemCount > 0) {
//                super.setItemCount(1);
//            } else {
//                super.setItemCount(0);
//            }
//        }
        scrollFixLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        scrollFixLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        scrollFixLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        scrollFixLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

        // fixLayoutHelper特有属性
     scrollFixLayoutHelper.setAlignType(FixLayoutHelper.TOP_LEFT);// 设置吸边时的基准位置(alignType)
        scrollFixLayoutHelper.setX(30);// 设置基准位置的横向偏移量X
        scrollFixLayoutHelper.setY(50);// 设置基准位置的纵向偏移量Y
        scrollFixLayoutHelper.setShowType(ScrollFixLayoutHelper.SHOW_ON_ENTER);// 设置Item的显示模式
```

##### ScrollFixLayoutHelper特有属性说明

**a. AlignType、x、y属性**

- 作用：

  1. alignType：吸边基准类型

  > 共有4个取值：TOP_LEFT(默认), TOP_RIGHT, BOTTOM_LEFT, BOTTOM_RIGHT，具体请看下面示意图

  1. x：基准位置的横向偏移量
  2. y：基准位置的纵向偏移量

![img](https:////upload-images.jianshu.io/upload_images/944365-4eb84390fc9c7ede.png?imageMogr2/auto-orient/strip|imageView2/2/w/974/format/webp)

示意图

- 具体使用



```csharp
// 接口示意
public void setAlignType(int alignType)
public void setX(int x)
public void setY(int y)

// 具体使用
ScrollFixLayoutHelper.setAlignType(FixLayoutHelper.TOP_LEFT);
ScrollFixLayoutHelper.setX(30);
ScrollFixLayoutHelper.setY(50);
```

**b. ShowType属性**

- 作用：设置Item的显示模式

> 共有三种显示模式
>
> 1. SHOW_ALWAYS：永远显示(即效果同固定布局)
> 2. SHOW_ON_ENTER：默认不显示视图，当页面滚动到该视图位置时才显示；
> 3. SHOW_ON_LEAVE：默认不显示视图，当页面滚出该视图位置时才显示

- 具体使用



```csharp
// 接口示意
public void setShowType(int showType)

// 具体使用
scrollFixLayoutHelper.setShowType(ScrollFixLayoutHelper.SHOW_ON_ENTER);
```

------

# 5. 浮动布局（FloatLayoutHelper）

- 布局说明：布局里的Item只有一个

> 1. 可随意拖动，但最终会被吸边到两侧
> 2. 不随页面滚动而移动

![img](https:////upload-images.jianshu.io/upload_images/944365-02983b34b5fc4948.gif?imageMogr2/auto-orient/strip|imageView2/2/w/388/format/webp)

示意图

- 具体使用



```cpp
/**
         设置浮动布局
         */
        FloatLayoutHelper floatLayoutHelper = new FloatLayoutHelper();
        // 创建FloatLayoutHelper对象

        // 公共属性
        floatLayoutHelper.setItemCount(1);// 设置布局里Item个数
        // 从设置Item数目的源码可以看出，一个FixLayoutHelper只能设置一个
//        @Override
//        public void setItemCount(int itemCount) {
//            if (itemCount > 0) {
//                super.setItemCount(1);
//            } else {
//                super.setItemCount(0);
//            }
//        }
        floatLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        floatLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        floatLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        floatLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

        // floatLayoutHelper特有属性
        floatLayoutHelper.setDefaultLocation(300,300);// 设置布局里Item的初始位置

        
```

# 6. 栏格布局（ColumnLayoutHelper）

- 布局说明：该布局只设有一栏（该栏设置多个Item）

> 可理解为只有一行的线性布局

![img](https:////upload-images.jianshu.io/upload_images/944365-acb01aebf85f2a70.png?imageMogr2/auto-orient/strip|imageView2/2/w/772/format/webp)

示意图



```cpp
/**
         设置栏格布局
         */
        ColumnLayoutHelper columnLayoutHelper = new ColumnLayoutHelper();
        // 创建对象

        // 公共属性
        columnLayoutHelper.setItemCount(3);// 设置布局里Item个数
        columnLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        columnLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        columnLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        columnLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

        // columnLayoutHelper特有属性
        columnLayoutHelper.setWeights(new float[]{30, 40, 30});// 设置该行每个Item占该行总宽度的比例
        // 同上面Weigths属性讲解
```

------

# 7. 通栏布局（SingleLayoutHelper）

- 布局说明：布局只有一栏，该栏只有一个Item

![img](https:////upload-images.jianshu.io/upload_images/944365-c1988974afe7474c.png?imageMogr2/auto-orient/strip|imageView2/2/w/742/format/webp)

示意图

- 具体使用



```cpp
/**
         设置通栏布局
         */

        SingleLayoutHelper singleLayoutHelper = new SingleLayoutHelper();

        // 公共属性
        singleLayoutHelper.setItemCount(3);// 设置布局里Item个数
        singleLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        singleLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        singleLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        singleLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比
        
```

------

# 8. 一拖N布局 （OnePlusNLayoutHelper）

- 布局说明：将布局分为不同比例，最多是1拖4。具体如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-a80552e502635986.png?imageMogr2/auto-orient/strip|imageView2/2/w/620/format/webp)

示意图1

![img](https:////upload-images.jianshu.io/upload_images/944365-dfaa0668d1e5ac16.png?imageMogr2/auto-orient/strip|imageView2/2/w/760/format/webp)

示意图2

- 具体使用



```cpp
/**
         设置1拖N布局
         */
        OnePlusNLayoutHelper onePlusNLayoutHelper = new OnePlusNLayoutHelper(5);
        // 在构造函数里传入显示的Item数
        // 最多是1拖4,即5个

        // 公共属性
        onePlusNLayoutHelper.setItemCount(3);// 设置布局里Item个数
        onePlusNLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        onePlusNLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        onePlusNLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        onePlusNLayoutHelper.setAspectRatio(3);// 设置设置布局内每行布局的宽与高的比
```

------

# 9. 吸边布局（StickyLayoutHelper）

- 布局说明：布局只有一个Item，显示逻辑如下：
  1. 当它包含的组件处于屏幕可见范围内时，像正常的组件一样随页面滚动而滚动
  2. 当组件将要被滑出屏幕返回的时候，可以吸到屏幕的顶部或者底部，实现一种吸住的效果
- 示意图（吸在顶部）

![img](https:////upload-images.jianshu.io/upload_images/944365-9ccde2d4177f1276.gif?imageMogr2/auto-orient/strip|imageView2/2/w/390/format/webp)

示意图

- 具体使用



```csharp
        /**
         设置吸边布局
         */
        StickyLayoutHelper stickyLayoutHelper = new StickyLayoutHelper();

        // 公共属性
        stickyLayoutHelper.setItemCount(3);// 设置布局里Item个数
        stickyLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        stickyLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        stickyLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        stickyLayoutHelper.setAspectRatio(3);// 设置设置布局内每行布局的宽与高的比

        // 特有属性
        stickyLayoutHelper.setStickyStart(true);
        // true = 组件吸在顶部
        // false = 组件吸在底部

        stickyLayoutHelper.setOffset(100);// 设置吸边位置的偏移量

        Adapter_StickyLayout = new MyAdapter(this, stickyLayoutHelper,1, listItem) {
            // 设置需要展示的数据总数,此处设置是1
            // 为了展示效果,通过重写onBindViewHolder()将布局的第一个数据设置为Stick
            @Override
            public void onBindViewHolder(MainViewHolder holder, int position) {
                super.onBindViewHolder(holder, position);
                if (position == 0) {
                    holder.Text.setText("Stick");
                }
            }
        };

        adapters.add(Adapter_StickyLayout) ;
        // 将当前的Adapter加入到Adapter列表里
```

### stickyStart、 offset属性说明

- 作用：

  1. `stickyStart`：设置吸边位置

  > 当视图的位置在屏幕范围内时，视图会随页面滚动而滚动；当视图的位置滑出屏幕时，StickyLayoutHelper会将视图固定在顶部（stickyStart = true）或 底部（stickyStart = false）

  1. `offset`：设置吸边的偏移量

- 具体使用



```java
// 接口示意
        public void setStickyStart(boolean stickyStart)
        public void setOffset(int offset)

// 具体使用
        stickyLayoutHelper.setStickyStart(true);
        // true = 组件吸在顶部
        // false = 组件吸在底部
        stickyLayoutHelper.setOffset(100);// 设置吸边位置的偏移量
```

------

# 10. 瀑布流布局（StaggeredGridLayoutHelper）

- 布局说明：以网格的形式进行布局。与网格布局类似，区别在于：
  - 网格布局每栏的Item高度是相等的；
  - 瀑布流布局每栏的Item高度是可以不相等的。

![img](https:////upload-images.jianshu.io/upload_images/944365-d338e215a4f5c538.png?imageMogr2/auto-orient/strip|imageView2/2/w/756/format/webp)

示意图

- 具体使用



```cpp
        /**
         设置瀑布流布局
         */

        StaggeredGridLayoutHelper staggeredGridLayoutHelper = new StaggeredGridLayoutHelper();
        // 创建对象

        // 公有属性
        staggeredGridLayoutHelper.setItemCount(20);// 设置布局里Item个数
        staggeredGridLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        staggeredGridLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        staggeredGridLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        staggeredGridLayoutHelper.setAspectRatio(3);// 设置设置布局内每行布局的宽与高的比

        // 特有属性
        staggeredGridLayoutHelper.setLane(3);// 设置控制瀑布流每行的Item数
        staggeredGridLayoutHelper.setHGap(20);// 设置子元素之间的水平间距
        staggeredGridLayoutHelper.setVGap(15);// 设置子元素之间的垂直间距

       
```

### 自定义布局（即自定义LayoutHelper）

除了使用系统提供的默认布局 `LayoutHelper`，开发者还可以通过自定义LayoutHelper从而实现自定义布局样式。有三种方式：

1. 继承`BaseLayoutHelper`：从上而下排列的顺序 & 内部 `View`可以按行回收的布局；主要实现`layoutViews()`、`computeAlignOffset()`等方法

> `LinearLayoutHelper`、`GridLayoutHelper`都是采用该方法实现

1. 继承`AbstractFullFillLayoutHelper`：有些布局内部的 `View` 并不是从上至下排列的顺序（即 `Adatper` 里的数据顺序和物理视图顺序不一致，那么可能就不能按数据顺序布局和回收），需要一次性布局
    & 回收。主要实现`layoutViews()`等方法

> `OnePlusNLayoutHelper`采用该方法实现

1. 继承`FixAreaLayoutHelper`：`fix` 类型布局，子节点不随页面滚动而滚动。主要实现`layoutViews()`、`beforeLayout()`、`afterLayout()`等方法

> `FixLayoutHelper`采用该方法实现

------

### 步骤5：将生成的LayoutHelper 交给Adapter，并绑定到RecyclerView 对象

此处的做法会因步骤3中Adapter的设置而有所不同



```csharp
<-- Adapter继承 自 DelegateAdapter -->

// 步骤1：设置Adapter列表（同时也是设置LayoutHelper列表）
        List<DelegateAdapter.Adapter> adapters = new LinkedList<>();

// 步骤2：创建自定义的Adapter对象 & 绑定数据 & 绑定上述对应的LayoutHelper
// 绑定你需要展示的布局LayoutHelper即可，此处仅展示两个。
        MyAdapter Adapter_linearLayout    = new MyAdapter(this, linearLayoutHelper，ListItem)；
// ListItem是需要绑定的数据（其实取决于你的Adapter如何定义）

        MyAdapter Adapter_gridLayoutHelper    = new MyAdapter(this, gridLayoutHelper，ListItem)；

// 步骤3：将创建的Adapter对象放入到DelegateAdapter.Adapter列表里
        adapters.add（Adapter_linearLayout ) ;
        adapters.add（Adapter_gridLayoutHelper ) ;

// 步骤4：创建DelegateAdapter对象 & 将layoutManager绑定到DelegateAdapter
DelegateAdapter delegateAdapter = new DelegateAdapter(layoutManager);

// 步骤5：将DelegateAdapter.Adapter列表绑定到DelegateAdapter
 delegateAdapter.setAdapters(adapters);

// 步骤6：将delegateAdapter绑定到recyclerView
recyclerView.setAdapter(delegateAdapter);



<-- Adapter继承 自 VirtualLayoutAdapter -->

// 步骤1：设置LayoutHelper列表
        List<LayoutHelper> helpers = new LinkedList<>();

// 步骤2：绑定上述对应的LayoutHelper
helpers.add（Adapter_linearLayout ）；
helpers.add（Adapter_gridLayoutHelper ) ;

// 步骤3：创建自定义的MyAdapter对象 & 绑定layoutManager
MyAdapter myAdapter = new MyAdapter(layoutManager);

// 步骤4：将 layoutHelper 列表传递给 adapter
myAdapter.setLayoutHelpers(helpers);

// 步骤5：将adapter绑定到recyclerView
recycler.setAdapter(myAdapter);
```

至此，使用过程讲解完毕。

------

# 6. 实例说明

- V-Layout的优点在于快速的组合不同布局
- 下面，我将根据上面的步骤说明，用一个实例来使用 `V - Layout`快速组合布局

### 步骤1：在`Android - Gradle`加入依赖



```bash
compile ('com.alibaba.android:vlayout:1.0.3@aar') {
        transitive = true
    }
```

### 步骤2：定义主 `xml`布局

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
    tools:context="scut.carson_ho.v_layoutusage.MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/my_recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="horizontal" />
</RelativeLayout>
```

### 步骤3：定义 `RecyclerView`每个子元素（ `Item` ）的xml布局

*item.xml*

> 此处定义的 `Item` 布局是常用的 **上字下图**



```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="New Text"
        android:id="@+id/Item" />

    <ImageView
        android:layout_alignParentRight="true"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/Image"/>

</LinearLayout>
```

### 步骤4：设置Adapter

- 设置 V - Layout的Adapter有两种方式:
  1. 继承 自 `DelegateAdapter`

> 此处主要以该方式进行演示

1. 继承 自 `VirtualLayoutAdapter`

- 具体使用

*MyAdapter.java*



```java
public class MyAdapter extends DelegateAdapter.Adapter<MyAdapter.MainViewHolder> {
    // 使用DelegateAdapter首先就是要自定义一个它的内部类Adapter，让LayoutHelper和需要绑定的数据传进去
    // 此处的Adapter和普通RecyclerView定义的Adapter只相差了一个onCreateLayoutHelper()方法，其他的都是一样的做法.

    private ArrayList<HashMap<String, Object>> listItem;
    // 用于存放数据列表

    private Context context;
    private LayoutHelper layoutHelper;
    private RecyclerView.LayoutParams layoutParams;
    private int count = 0;

    private MyItemClickListener myItemClickListener;
    // 用于设置Item点击事件

    //构造函数(传入每个的数据列表 & 展示的Item数量)
    public MyAdapter(Context context, LayoutHelper layoutHelper, int count, ArrayList<HashMap<String, Object>> listItem) {
        this(context, layoutHelper, count, new RecyclerView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, 300), listItem);
    }

    public MyAdapter(Context context, LayoutHelper layoutHelper, int count, @NonNull RecyclerView.LayoutParams layoutParams, ArrayList<HashMap<String, Object>> listItem) {
        this.context = context;
        this.layoutHelper = layoutHelper;
        this.count = count;
        this.layoutParams = layoutParams;
        this.listItem = listItem;
    }

    // 把ViewHolder绑定Item的布局
    @Override
    public MainViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return new MainViewHolder(LayoutInflater.from(context).inflate(R.layout.item, parent, false));
    }

    // 此处的Adapter和普通RecyclerView定义的Adapter只相差了一个onCreateLayoutHelper()方法
    @Override
    public LayoutHelper onCreateLayoutHelper() {
        return layoutHelper;
    }

    // 绑定Item的数据
    @Override
    public void onBindViewHolder(MainViewHolder holder, int position) {
        holder.Text.setText((String) listItem.get(position).get("ItemTitle"));
        holder.image.setImageResource((Integer) listItem.get(position).get("ItemImage"));

    }

    // 返回Item数目
    @Override
    public int getItemCount() {
        return count;
    }

    // 设置Item的点击事件
    // 绑定MainActivity传进来的点击监听器
    public void setOnItemClickListener(MyItemClickListener listener) {
        myItemClickListener = listener;
    }

    
    //定义Viewholder
    class MainViewHolder extends RecyclerView.ViewHolder {
        public TextView Text;
        public ImageView image;

        public MainViewHolder(View root) {
            super(root);
            
            // 绑定视图
            Text = (TextView) root.findViewById(R.id.Item);
            image = (ImageView) root.findViewById(R.id.Image);

            root.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    if (myItemClickListener != null)
                        myItemClickListener.onItemClick(v, getPosition());
                }

            }
            //监听到点击就回调MainActivity的onItemClick函数
            );

        }

        public TextView getText() {
            return Text;
        }
    }
}
```

### 以下步骤都将写在同一个`.Java`文件里

步骤5：创建RecyclerView & VirtualLayoutManager 对象并进行绑定
 步骤6：设置回收复用池大小
 步骤7：设置需要存放的数据
 步骤8：根据数据列表，创建对应的LayoutHelper
 步骤9：将生成的LayoutHelper 交给Adapter，并绑定到RecyclerView 对象

> 详细请看注释

*MainActivity.java*



```csharp
public class MainActivity extends AppCompatActivity implements MyItemClickListener {
    RecyclerView recyclerView;
    MyAdapter Adapter_linearLayout,Adapter_GridLayout,Adapter_FixLayout,Adapter_ScrollFixLayout
            ,Adapter_FloatLayout,Adapter_ColumnLayout,Adapter_SingleLayout,Adapter_onePlusNLayout,
            Adapter_StickyLayout,Adapter_StaggeredGridLayout;
    private ArrayList<HashMap<String, Object>> listItem;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /**
         * 步骤1：创建RecyclerView & VirtualLayoutManager 对象并进行绑定
         * */
        recyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);
        // 创建RecyclerView对象

        VirtualLayoutManager layoutManager = new VirtualLayoutManager(this);
        // 创建VirtualLayoutManager对象
        // 同时内部会创建一个LayoutHelperFinder对象，用来后续的LayoutHelper查找

        recyclerView.setLayoutManager(layoutManager);
        // 将VirtualLayoutManager绑定到recyclerView

        /**
         * 步骤2：设置组件复用回收池
         * */
        RecyclerView.RecycledViewPool viewPool = new RecyclerView.RecycledViewPool();
        recyclerView.setRecycledViewPool(viewPool);
        viewPool.setMaxRecycledViews(0, 10);

        /**
         * 步骤3:设置需要存放的数据
         * */
        listItem = new ArrayList<HashMap<String, Object>>();
        for (int i = 0; i < 100; i++) {
            HashMap<String, Object> map = new HashMap<String, Object>();
            map.put("ItemTitle", "第" + i + "行");
            map.put("ItemImage", R.mipmap.ic_launcher);
            listItem.add(map);

        }

        /**
         * 步骤4:根据数据列表,创建对应的LayoutHelper
         * */

        // 为了展示效果,此处将上面介绍的所有布局都显示出来

        /**
         设置线性布局
         */
        LinearLayoutHelper linearLayoutHelper = new LinearLayoutHelper();
        // 创建对应的LayoutHelper对象

        // 公共属性
        linearLayoutHelper.setItemCount(4);// 设置布局里Item个数
        linearLayoutHelper.setPadding(20, 20, 20, 20);// 设置LayoutHelper的子元素相对LayoutHelper边缘的距离
        linearLayoutHelper.setMargin(20, 20, 20, 20);// 设置LayoutHelper边缘相对父控件（即RecyclerView）的距离
        // linearLayoutHelper.setBgColor(Color.GRAY);// 设置背景颜色
        linearLayoutHelper.setAspectRatio(6);// 设置设置布局内每行布局的宽与高的比

        // linearLayoutHelper特有属性
        linearLayoutHelper.setDividerHeight(10);
        // 设置间隔高度
        // 设置的间隔会与RecyclerView的addItemDecoration（）添加的间隔叠加.

        linearLayoutHelper.setMarginBottom(100);
        // 设置布局底部与下个布局的间隔


        // 创建自定义的Adapter对象 & 绑定数据 & 绑定对应的LayoutHelper进行布局绘制
         Adapter_linearLayout  = new MyAdapter(this, linearLayoutHelper, 4, listItem) {
             // 参数2:绑定绑定对应的LayoutHelper
             // 参数3:传入该布局需要显示的数据个数
             // 参数4:传入需要绑定的数据

             // 通过重写onBindViewHolder()设置更丰富的布局效果
             @Override
             public void onBindViewHolder(MainViewHolder holder, int position) {
                 super.onBindViewHolder(holder, position);
                 // 为了展示效果,将布局的第一个数据设置为linearLayout
                 if (position == 0) {
                     holder.Text.setText("Linear");
                 }

                  //为了展示效果,将布局里不同位置的Item进行背景颜色设置
                 if (position < 2) {
                     holder.itemView.setBackgroundColor(0x66cc0000 + (position - 6) * 128);
                 } else if (position % 2 == 0) {
                     holder.itemView.setBackgroundColor(0xaa22ff22);
                 } else {
                     holder.itemView.setBackgroundColor(0xccff22ff);
                 }

             }
         };

        Adapter_linearLayout.setOnItemClickListener(this);
        // 设置每个Item的点击事件

        ....// 还有其他布局，由于代码量就较多就不贴出来了。

        /**
         *步骤5:将生成的LayoutHelper 交给Adapter，并绑定到RecyclerView 对象
         **/

        // 1. 设置Adapter列表（同时也是设置LayoutHelper列表）
        List<DelegateAdapter.Adapter> adapters = new LinkedList<>();

        // 2. 将上述创建的Adapter对象放入到DelegateAdapter.Adapter列表里
        adapters.add(Adapter_linearLayout) ;
        adapters.add(Adapter_StickyLayout) ;
        adapters.add(Adapter_ScrollFixLayout) ;
        adapters.add(Adapter_GridLayout) ;
        adapters.add(Adapter_FixLayout) ;
        adapters.add(Adapter_FloatLayout) ;
        adapters.add(Adapter_ColumnLayout) ;
        adapters.add(Adapter_SingleLayout) ;
        adapters.add(Adapter_onePlusNLayout) ;
        adapters.add(Adapter_StaggeredGridLayout) ;

        // 3. 创建DelegateAdapter对象 & 将layoutManager绑定到DelegateAdapter
        DelegateAdapter delegateAdapter = new DelegateAdapter(layoutManager);

        // 4. 将DelegateAdapter.Adapter列表绑定到DelegateAdapter
        delegateAdapter.setAdapters(adapters);

        // 5. 将delegateAdapter绑定到recyclerView
        recyclerView.setAdapter(delegateAdapter);


        /**
         *步骤6:Item之间的间隔
         **/

        recyclerView.addItemDecoration(new RecyclerView.ItemDecoration() {
            public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
                outRect.set(5, 5, 5, 5);
            }
        });


    }

    /**
     *步骤7:实现Item点击事件
     **/
    // 点击事件的回调函数
    @Override
    public void onItemClick(View view, int postion) {
        System.out.println("点击了第"+postion+"行");
        Toast.makeText(this, (String) listItem.get(postion).get("ItemTitle"), Toast.LENGTH_SHORT).show();
    }
}
```

### 效果图

![img](https:////upload-images.jianshu.io/upload_images/944365-73ba1d3af5747b17.gif?imageMogr2/auto-orient/strip|imageView2/2/w/390/format/webp)

总效果图

### 源码地址

Carson_Ho的Github地址：[V - Layout](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FCarson-Ho%2FVLayout-Guide)

> 参考文档：
>  [https://github.com/alibaba/vlayout](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fvlayout)
>  [http://pingguohe.net/2017/02/28/vlayout-design.html](https://links.jianshu.com/go?to=http%3A%2F%2Fpingguohe.net%2F2017%2F02%2F28%2Fvlayout-design.html)

------

# 7. 总结

- 看完本文，你应该非常了解阿里出品的`V - Layout` 的使用 & 原理



[Carson带你学Android：淘宝、天猫都在用的UI框架V-Layout，赶紧用起来吧！](https://www.jianshu.com/p/6b658c8802d1)