> version：2022/07/
>
> review：



目录

[TOC]



# 关键词



# 一、前置知识

View体系（View、ViewGroup）、自定义View



# 二、概述





# 三、核心类



| 类名           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| Adapter        | 负责将数据绑定到视图上。然后将绑定后的视图提供给RecyclerView使用。 |
| ViewHolder     | 是item views的一个包装类，RecyclerView中对于item vIews的获取都是通过VIewHolder。负责提供item views，并对item views做初始化等操作。 |
| LayoutManager  | 负责item views的布局。并在绘制布局流程中，提供回调。         |
| ItemDecoration | 负责绘制item views附近的分割线                               |
| ItemAnimator   | 为Item的一般操作添加动画效果，如增删条目等                   |
|                |                                                              |
|                |                                                              |
|                |                                                              |
|                |                                                              |



RecyclerVIew、Recycler、Adapter、ViewHolder之间的关系

RecyclerVIew在绘制（measure）的时候，会通过position去获取子view（item view），具体的获取工作交给Recycler来完成，Recycler内部会对VH进行缓存复用，当RecyclerVIew想要item view的时候，Recycler首先会查找是否有可以复用的VH，如果有的话，就返回给RecyclerView使用，没有的话，就通过adapter来创建，也就是去调用Adapter#onCreateViewHolder生成VH，VH是对item view的一个封装，其内部包含item ivew的相关信息，比如type、当前的position、当前item view的状态（通过flag来记录的）。

> 关于type的作用，这个是为了RecyclerVIew能够区分不同类型的VH，因为RecyclerVIew内部使用的都是抽象类ViewHolder，所以需要引入一个变量来记录ViewHolder的类型。





# 四、ViewHolder

1、ViewHolder的作用是什么？

1. 用来封装item view及其相关数据的，比如item view的类型（type）
2. 并通过Adapter提供item view给RecyclerView

VH的创建和更新是通过Adapter来完成的。

RV通过Recycler获取VH，在没有找到可复用的VH时，Recycler会通过adapter来创建。



2、如何理解ViewHolder的复用？

我们先看下面这两个列表，以对复用有个直观的感受：

| ![image-20220721010724240](images/RecyclerView%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90/image-20220721010724240.png) | ![image-20220721010857340](images/RecyclerView%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90/image-20220721010857340.png) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |

可以看到，上滑了几个item后，ViewHolder实现了复用（就是使用了之前创建的VH，没有重新new）。

复用的本质就是不新建VH对象，只去改变VH对象的内容，也就是通过`Adapter#onBindViewHolder(@NonNull VH holder, int position)`方法，去更新item view。

```java
// holder就是被复用的对象
public abstract void onBindViewHolder(@NonNull VH holder, int position);
```

复用有啥好处呢？

1. 避免频繁的销毁和创建VH对象，提高程序的性能。

   其一是，VH的创建需要传入一个item view的，而 item view 通常都是通过 View.inflate 去加载的，是需要去解析xml文件的，所以是开销比较大的。

   其二是，对象的频繁销毁与创建本身就会影响性能，创建对象多了，内存占用高，再新建对象就会变慢，GC也频繁了。

2. 可以避免多次执行`findViewById()` ，这个方法会通过id去匹配view，需要遍历view children，通常使用VH都会在构造方法中进行`findViewById()`并把找到的view通过变量来持有，这样复用时，这部分时间开销就节约了。属于用空间换时间。



# 五、Adapter

<font color='orange'>Q：adapter的作用是什么，几个方法是做什么用的？</font>

```java
public abstract static class Adapter<VH extends ViewHolder> {

@NonNull
public abstract VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType);
public abstract void onBindViewHolder(@NonNull VH holder, int position);

}
```

adapter的作用有：

1、通过onCreateViewHolder()方法为RV提供VH（也就是item view），这里有个type参数，我们可以根据这个type返回不同的VH。type是通过adapter重写 `getItemViewType`方法设置的。

2、当VH复用时，根据position更新VH中item view。也就是调用了onBindViewHolder()。

3、RV回调（有多个）。在一些特殊的节点，调用adapter的方法，我们重写这些方法就可以加上自己的代码。简单看几个

```java
public abstract static class Adapter<VH extends ViewHolder> {
    // view被回收了
 public void onViewRecycled(@NonNull VH holder) {
        }
    // view附加到window上了
    public void onViewAttachedToWindow(@NonNull VH holder) {
        }
}
```

上面3个（种）方法，都是RV主动调用的，来获取或更新VH的。

除此以外，RV 和 adapter 之间还有adapter主动通知的代码，也就是订阅者或观察者模式。

```java
public abstract static class Adapter<VH extends ViewHolder> {
private final AdapterDataObservable mObservable = new AdapterDataObservable();

// 添加观察者
public void registerAdapterDataObserver(@NonNull AdapterDataObserver observer) {
    mObservable.registerObserver(observer);
}
// 数据变化，通知观察者
public final void notifyItemChanged(int position) {
    mObservable.notifyItemRangeChanged(position, 1);
}
}

public class RecyclerView extends ViewGroup implements ScrollingView,
        NestedScrollingChild2, NestedScrollingChild3 {
private final RecyclerViewDataObserver mObserver = new RecyclerViewDataObserver();
    
private void setAdapterInternal(@Nullable Adapter adapter, boolean compatibleWithPrevious,
            boolean removeAndRecycleViews) {
        if (adapter != null) {
            // 往adapter注册观察者
            adapter.registerAdapterDataObserver(mObserver);
            adapter.onAttachedToRecyclerView(this);
		}    
}

// RV中的观察者，adapter通知有变化时，观察者代码执行
private class RecyclerViewDataObserver extends AdapterDataObserver {
        @Override
        public void onItemRangeChanged(int positionStart, int itemCount, Object payload) {
            assertNotInLayoutOrScroll(null);
            if (mAdapterHelper.onItemRangeChanged(positionStart, itemCount, payload)) {
                triggerUpdateProcessor();
            }
        }

        // 核心就是告诉RV，要去更新UI了。
        void triggerUpdateProcessor() {
            if (POST_UPDATES_ON_ANIMATION && mHasFixedSize && mIsAttached) {
                ViewCompat.postOnAnimation(RecyclerView.this, mUpdateChildViewsRunnable);
            } else {
                mAdapterUpdateDuringMeasure = true;
                requestLayout();
            }
        }
    }
}
```

上面的代码是一个标准的观察者模式，RV提供观察者，注册到adapter中，adapter数据变化时，通知给观察者，观察者再触发RV的更新。

观察者的核心作用就是告诉RV，数据有变化了，要去更新下UI。

小结一下，细分下来总共四个作用，分别是：

RV主动调用的：提供VH，更新VH，回调

我们主动调用的：通知RV（观察者）数据变化了











# FAQ：常见问题

## 1、ViewHolder

<font color='orange'>Q：ViewHolder的作用是什么？</font>

<font color='orange'>Q：如何理解ViewHolder的复用？</font>

关键是如何复用的？



<font color='orange'>Q：ViewHolder封装如何对findViewById优化？</font>

关键是因为复用。

因为VH会被Recycler复用（也就是VH不会被销毁掉），所以只要在VH的构造方法中，进行findViewById，并把找到的view通过VH来持有，这样复用的时候，就不需要通过findViewById()来遍历了。

实际上，不局限于findViewById，所有在VH构造方法或者static中执行的代码都会起到复用的效果。比如在VH中，对item view做一些初始化，这些代码是会复用生效的。比如item view中有一个button#setOnClickListener()，这个代码也是会复用的。



<font color='orange'>Q：ViewHolder为什么要被声明成静态内部类</font>

因为VH不需要使用RV的数据（不需要使用外部类的变量、调用外部类的方法），所以不需要持有RV。所以不需要设置成内部类，因为内部类会默认持有外部类的引用，可以访问外部类的数据。如果设置成内部类，一个是没有任何作用，反而因为持有着RV，可能会引起内存泄漏等问题。本质上VH就是也个封装好的Bean类，只是用来提供和保存数据的，来给RV、Adapter使用。

> 给我的启发就是，如果创建内部类的话，默认都应该是static的，除非必须使用到外部类的数据，才应该考虑改成内部类。也不一定就要用内部类，因为还可以通过方法传入外部类。还是要结合具体问题来分析。

ViewHolder为啥要放到RecyclerView内部呢？

主要就是强调一下，从设计上来看，RecyclerView是依赖VH的。其实，将VH独立到一个Java文件中，也是没有任何影响的。实际上，VH只负责保存和提供item view相关的数据，至于这个数据是RV使用还是Adapter使用，它是不关心的。所以，我来写的话，应该会把VH独立到一个文件中去。



## 2、Adapter

<font color='orange'>Q：adapter的作用是什么，几个方法是做什么用的？</font>



<font color='orange'>Q：什么时候停止调用onCreateViewHolder？</font>



<font color='orange'>Q：如何理解adapter订阅者模式？</font>

上面讲adapter作用的时候，已经介绍了观察者模式。之所以需要是因为，在RV中item的更新，不仅是滑动时才有的（滑动时，RV会主动调用adapter的onCreateViewHolder或onBindViewHolder来创建或更新VH），而且RV不动时，数据也可能会变化，这个时候，RV是不知道的，所以就需要一种方式主动告诉RV，这个事情就是通过adapter的观察者模式来做的。



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>

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

1、[RecyclerView问题汇总](https://juejin.cn/post/6844903837724213256)
