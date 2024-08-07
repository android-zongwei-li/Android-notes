第16讲：为什么 RecyclerView 可以完美替代 Listview？



本课时我们学习为什么 RecyclerView 可以完美替代 LI。

RecyclerView 简称 RV， 是作为 ListView 和 GridView 的加强版出现的，目的是在有限的屏幕之上展示大量的内容，因此 RecyclerView 的复用机制的实现是它的一个核心部分。

RV 常规使用方式如下：

![image.png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68ue6AI3vLAACGhkLqn8M080.png)

解释说明。

- setLayoutManager：必选项，设置 RV 的布局管理器，决定 RV 的显示风格。常用的有线性布局管理器（LinearLayoutManager）、网格布局管理器（GridLayoutManager）、瀑布流布局管理器（StaggeredGridLayoutManager）。
- setAdapter：必选项，设置 RV 的数据适配器。当数据发生改变时，以通知者的身份，通知 RV 数据改变进行列表刷新操作。
- addItemDecoration：非必选项，设置 RV 中 Item 的装饰器，经常用来设置 Item 的分割线。
- setItemAnimator：非必选项，设置 RV 中 Item 的动画。

本课时主要来看下 RV 是如何一步步将每一个 ItemView 显示到屏幕上，然后再分析在显示和滑动过程中，是如何通过缓存复用来提升整体性能的。RV 本质上也是一个自定义控件，所以也符合上节课所讲的自定义控件的规则。因此我们也可以沿着分析其 onMeasure -> onLayout -> onDraw 这 3 个方法的路线来深入研究。

# 绘制流程分析

## onMeasure

RV 的 onMeasure 方法如下：

![image (1).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68ufiAe8_uAAV5DJyDB7M094.png)

- 图中 1 处，表示在 XML 布局文件中，RV 的宽高被设置为 match_parent 或者具体值，那么直接将 skipMeasure 置为 true，并调用 mLayout（传入的 LayoutManager）的 onMeasure 方法测量自身的宽高即可。
- 图中 2 处，表示在 XML 布局文件中，RV 的宽高设置为 wrap_content，则会执行下面的 dispatchLayoutStep2()，其实就是测量 RecyclerView 的子 View 的大小，最终确定 RecyclerView 的实际宽高。

注意：
在上图代码中还有一个 dispatchLayoutStep1() 方法，这个方法并不是本节课重点介绍内容，但是它跟RV的动画息息相关，详细可以参考： [RecyclerView.ItemAnimator实现动画效果](https://mp.weixin.qq.com/s?__biz=MzU3Mjc5NjAzMw==&mid=2247484487&idx=1&sn=bb0b7de72d20011199dcc140d6925f8e&chksm=fcca39a9cbbdb0bf29a48db16f3e7a019aa3d98da88d60389f79c91cbcf8092bdb6862513191&token=1367838045&lang=zh_CN#rd)

## onLayout

RV 的 onLayout 方法如下：

![image (2).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68ugWAAd26AAC9tEb_ppo563.png)

很简单，只是调用了一层 dispatchLayout() 方法，此方法具体如下：

![image (3).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68ugyAIYT4AAUpCeyxNoY082.png)

如果在 onMeasure 阶段没有执行 dispatchLayoutStep2() 方法去测量子 View，则会在 onLayout 阶段重新执行。

dispatchLayoutStep2() 源码如下：

![image (4).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68uhOAau0zAADCKkMOvDk556.png)

可以看出，核心逻辑是调用了 mLayout 的 onLayoutChildren 方法。这个方法是 LayoutManager 中的一个空方法，主要作用是测量 RV 内的子 View 大小，并确定它们所在的位置。LinearLayoutManager、GridLayoutManager，以及 StaggeredLayoutManager 都分别复写了这个方法，并实现了不同方式的布局。

以 LinearLayoutManager 的实现为例，展开分析，实现如下 ：

![image (5).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68uhyAbDK-AAVKbLTTats711.png)

解释说明：

1. 在 onLayoutChildren 中调用 fill 方法，完成子 View 的测量布局工作；
2. 在 fill 方法中通过 while 循环判断是否还有剩余足够空间来绘制一个完整的子 View；
3. layoutChunk 方法中是子 View 测量布局的真正实现，每次执行完之后需要重新计算 remainingSpace。

layoutChunk 是一个非常核心的方法，这个方法执行一次就填充一个 ItemView 到 RV，部分代码如下：

![image (6).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68uiWAUf1UAAXuHvOgyg0072.png)

说明：

- 图中 1 处从缓存（Recycler）中取出子 ItemView，然后调用 addView 或者 addDisappearingView 将子 ItemView 添加到 RV 中。
- 图中 2 处测量被添加的 RV 中的子 ItemView 的宽高。
- 图中 3 处根据所设置的 Decoration、Margins 等所有选项确定子 ItemView 的显示位置。

## onDraw

测量和布局都完成之后，就剩下最后的绘制操作了。代码如下：

![image (7).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68ui6ATOvDAADD582WJrs779.png)

这个方法很简单，如果有添加 ItemDecoration，则循环调用所有的 Decoration 的 onDraw 方法，将其显示。至于所有的子 ItemView 则是通过 Android 渲染机制递归的调用子 ItemView 的 draw 方法显示到屏幕上。

**小结**：RV 会将测量 onMeasure 和布局 onLayout 的工作委托给 LayoutManager 来执行，不同的 LayoutManager 会有不同风格的布局显示，这是一种策略模式。用一张图来描述这段过程如下：

![image (8).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68ujuAJSpLAACb76c7LSA053.png)

## 缓存复用原理 Recycler

缓存复用是 RV 中另一个非常重要的机制，这套机制主要实现了 ViewHolder 的缓存以及复用。

核心代码是在 Recycler 中完成的，它是 RV 中的一个内部类，主要用来缓存屏幕内 ViewHolder 以及部分屏幕外 ViewHolder，部分代码如下：

![image (9).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68ukSAZT3uAAD9_pc55Io230.png)

Recycler 的缓存机制就是通过上图中的这些数据容器来实现的，实际上 Recycler 的缓存也是分级处理的，根据访问优先级从上到下可以分为 4 级，如下：

![image (10).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68ukuAM0LVAACHb_a34AY925.png)

### 各级缓存功能

RV 之所以要将缓存分成这么多块，是为了在功能上进行一些区分，并分别对应不同的使用场景。

#### a 第一级缓存 mAttachedScrap&mChangedScrap

是两个名为 Scrap 的 ArrayList，这两者主要用来缓存屏幕内的 ViewHolder。为什么屏幕内的 ViewHolder 需要缓存呢？做过 App 开发的应该都熟悉下面的布局场景：

![image (11).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68uleADj-bAAkZbbtqexQ425.png)

通过下拉刷新列表中的内容，当刷新被触发时，只需要在原有的 ViewHolder 基础上进行重新绑定新的数据 data 即可，而这些旧的 ViewHolder 就是被保存在 mAttachedScrap 和 mChangedScrap 中。实际上当我们调用 RV 的 notifyXXX 方法时，就会向这两个列表进行填充，将旧 ViewHolder 缓存起来。

#### b 第二级缓存 mCachedViews

它用来缓存移除屏幕之外的 ViewHolder，默认情况下缓存个数是 2，不过可以通过 setViewCacheSize 方法来改变缓存的容量大小。如果 mCachedViews 的容量已满，则会根据 FIFO 的规则将旧 ViewHolder 抛弃，然后添加新的 ViewHolder，如下所示：

![mCachedViews.gif](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68umKATRCWAEsI2pK1Mbo977.gif)

通常情况下刚被移出屏幕的 ViewHolder 有可能接下来马上就会使用到，所以 RV 不会立即将其设置为无效 ViewHolder，而是会将它们保存到 cache 中，但又不能将所有移除屏幕的 ViewHolder 都视为有效 ViewHolder，所以它的默认容量只有 2 个。

#### c 第三级缓存 ViewCacheExtension

这是 RV 预留给开发人员的一个抽象类，在这个类中只有一个抽象方法，如下：

![image (12).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68umuAdLe4AACjFw2gqeI470.png)

开发人员可以通过继承 ViewCacheExtension，并复写抽象方法 getViewForPositionAndType 来实现自己的缓存机制。只是一般情况下我们不会自己实现也不建议自己去添加缓存逻辑，因为这个类的使用门槛较高，需要开发人员对 RV 的源码非常熟悉。

#### d 第四级缓存 RecycledViewPool

RecycledViewPool 同样是用来缓存屏幕外的 ViewHolder，当 mCachedViews 中的个数已满（默认为 2），则从 mCachedViews 中淘汰出来的 ViewHolder 会先缓存到 RecycledViewPool 中。ViewHolder 在被缓存到 RecycledViewPool 时，会将内部的数据清理，因此从 RecycledViewPool 中取出来的 ViewHolder 需要重新调用 onBindViewHolder 绑定数据。这就同最早的 ListView 中的使用 ViewHolder 复用 convertView 的道理是一致的，因此 RV 也算是将 ListView 的优点完美的继承过来。

RecycledViewPool 还有一个重要功能，官方对其有如下解释：

> RecycledViewPool lets you share Views between multiple RecyclerViews.
>

可以看出，多个 RV 之间可以共享一个 RecycledViewPool，这对于多 tab 界面的优化效果会很显著。**需要注意的是，RecycledViewPool 是根据 type 来获取 ViewHolder，每个 type 默认最大缓存 5 个。**因此多个 RecyclerView 共享 RecycledViewPool 时，必须确保共享的 RecyclerView 使用的 Adapter 是同一个，或 view type 是不会冲突的。

### RV 是如何从缓存中获取 ViewHolder 的

在上文介绍 onLayout 阶段时，有介绍在 layoutChunk 方法中通过调用 layoutState.next 方法拿到某个子 ItemView，然后添加到 RV 中。

看一下 layoutState.next 的详细代码：

![image (13).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68unSAJ7fRAAEcvzM2UaA043.png)

代码继续往下跟：

![image (14).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68un6AKPaiAADjaBH3UEk983.png)

可以看出最终调用 tryGetViewHolderForPositionByDeadline 方法来查找相应位置上的ViewHolder，在这个方法中会从上面介绍的 4 级缓存中依次查找：

![image (15).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68uoaAefdEAASuYKk5V44193.png)

如图中红框处所示，如果在各级缓存中都没有找到相应的 ViewHolder，则会使用 Adapter 中的 createViewHolder 方法创建一个新的 ViewHolder。

### 何时将 ViewHolder 存入缓存

接下来看下 ViewHolder 被存入各级缓存的场景。

#### 第一次 layout

当调用 setLayoutManager 和 setAdapter 之后，RV 会经历第一次 layout 并被显示到屏幕上，如下所示：

![image (16).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68upCATZ2cAACZFf9DHQ0775.png)

此时并不会有任何 ViewHolder 的缓存，所有的 ViewHolder 都是通过 createViewHolder 创建的。

#### 刷新列表

如果通过手势下拉刷新，获取到新的数据 data 之后，我们会调用 notifyXXX 方法通知 RV 数据发生改变，这回 RV 会先将屏幕内的所有 ViewHolder 保存在 Scrap 中，如下所示：

![image (17).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68upeAaYMqAAChKtO2D8A471.png)

当缓存执行完之后，后续通过 Recycler 就可以从缓存中获取相应 position 的 ViewHolder（姑且称为旧 ViewHolder），然后将刷新后的数据设置到这些 ViewHolder 上，如下所示：

![image (18).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/Ciqc1F68up6AKO1yAAChI3xlrRw121.png)

最后再将新的 ViewHolder 绘制到 RV 上：

![image (19).png](images/第16讲：为什么 RecyclerView 可以完美替代 Listview？/CgqCHl68uqSAS-0JAACL6DDqSsw611.png)

# 总结

这节课我带着你深入分析了 Android RecyclerView 源码中的 2 块核心实现：

- RecyclerView 是如何经过测量、布局，最终绘制到屏幕上，其中大部分工作是通过委托给 LayoutManager 来实现的。
- RecyclerView 的缓存复用机制，主要是通过内部类 Recycler 来实现。

谷歌 Android 团队对 RecyclerView 做了很多优化，导致 RecyclerView 最终的代码极其庞大。这也是为什么当 RecyclerView 出现问题的时候，排查问题的复杂度相对较高。理解 RecyclerView 的源码实现，有助于我们快速定位问题原因、拓展 RecyclerView 功能、提高分析 RecyclerView 性能问题的能力。