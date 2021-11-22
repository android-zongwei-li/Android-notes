> version：2021/9/
>
> review：

[原文地址](https://developer.android.google.cn/guide/fragments/transactions)

目录

[TOC]

# Fragment 事务

在运行时， [`FragmentManager`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentManager)可以对fragments进行添加、删除、替换和执行其他操作，以响应用户交互。您提交的每一组fragments更改都称为事务（transactions），您可以使用 [`FragmentTransaction`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction) 类提供的API在事务中指定要做什么。可以将多个操作分组到单个事务中--例如，事务可以添加或替换多个fragment。当您在同一屏幕上显示多个同级片段时，例如使用拆分视图时，此分组可能非常有用。

您可以将每个事务保存到FragmentManager管理的`回退堆栈`中，允许用户通过回退完成fragment的更改--类似于回退activities。

您可以从FragmentManager获得一个FragmentTransaction实例，方法是调用`beginTransaction()`，如下所示：

```java
FragmentManager fragmentManager = ...
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
```

The final call on each `FragmentTransaction` must commit the transaction. The `commit()` call signals to the `FragmentManager` that all operations have been added to the transaction.

对每个FragmentTransaction的最后调用必须提交（commit ）事务。commit()通知FragmentManager所有操作都已添加到事务中。

```java
FragmentManager fragmentManager = ...
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

// Add operations here

fragmentTransaction.commit();
```



## 允许fragment状态更改的重排序

每个`FragmentTransaction`都应该使用`setReorderingAllowed(True)`：

```java
FragmentManager fragmentManager = ...
fragmentManager.beginTransaction()
    ...
    .setReorderingAllowed(true)
    .commit();
```

为了实现兼容性，默认情况下不启用重排序标志。但是它可以使`FragmentManager`正确地执行`FragmentTransaction`，特别是当它在回退栈（back stack）上操作并运行动画和转换时。启用标志可以确保，如果多个事务一起执行，任何中间fragment(即添加的、然后立即替换的)都不会进行生命周期更改，也不会执行它们的动画或转换。请注意，此标志既会影响事务的初始执行，也会影响使用`popBackStack()`逆转事务。

> Q：还不是特别理解。



## 添加和移除 fragments

要将片段添加到`FragmentManager`，请在事务上调用 [`add()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#add(int, java.lang.Class, android.os.Bundle))。此方法接收fragment 容器的ID以及希望添加的fragment 的类名。添加的fragment 被切换到`RESUMED`状态。强烈建议容器是一个[`FragmentContainerView`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentContainerView)，它是view层次结构的一部分。

要从宿主中删除fragment ，请调用 [`remove()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#remove(androidx.fragment.app.Fragment))，传入一个fragment 实例，该实例是通过`findFragmentById()`或`findFragmentByTag()`从fragment 管理器获取的。如果fragment 的视图先前添加到容器中，则此时视图会从容器中移除。被移除的fragment 将切换到 `DESTROYED`状态。

使用 [`replace()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#replace(int, java.lang.Class, android.os.Bundle))将容器中的现有fragment 替换为您提供的新fragment 类的实例。调用 `replace()` 相当于使用容器中的一个fragment 调用remove()，并向该容器中添加一个新的fragment。下面的代码片段展示了如何将一个fragment 替换为另一个fragment ：

```java
// Create new fragment and transaction
FragmentManager fragmentManager = ...
FragmentTransaction transaction = fragmentManager.beginTransaction();
transaction.setReorderingAllowed(true);

// Replace whatever is in the fragment_container view with this fragment
transaction.replace(R.id.fragment_container, ExampleFragment.class, null);

// Commit the transaction
transaction.commit();
```

在本例中， `ExampleFragment` 的一个新实例替换了当前由 `R.id.fragment_container`标识的布局容器中的fragment (如果有的话)。

> 注意：强烈建议始终使用采用Class而不是fragment 的实例进行操作，以确保创建和（从保存状态中）恢复fragment 的机制相同。有关详细信息，请参阅 [Fragment manager](https://developer.android.google.cn/guide/fragments/fragmentmanager) 。

默认情况下，`FragmentTransaction`中所做的更改不会添加到回退栈中。要保存这些更改，可以在`FragmentTransaction`上调用 [`addToBackStack()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#addToBackStack(java.lang.String))。有关更多信息，请参见 [Fragment manager](https://developer.android.google.cn/guide/fragments/fragmentmanager).。

### 异步提交

调用 [`commit()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#commit())不会立即执行事务。相反，事务会在UI线程排序并尽快执行。但是，如果有必要的话，您可以调用 [`commitNow()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#commit())来立即在UI线程上运行fragment 事务。

注意，`commitNow` 与`addToBackStack`不兼容。或者，可以通过调用[`executePendingTransactions()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentManager#executePendingTransactions())执行所有通过`FragmentTransaction` [`commit()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#commit())挂起（pending）并尚未运行的事务。此方法与`addToBackStack`兼容。

对于绝大多数用例，`commit()`能满足所有需要。

> Q：不兼容具体是什么含义呢？



### 操作顺序非常重要

The order in which you perform operations within a [`FragmentTransaction`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction) is significant, particularly when using `setCustomAnimations()`. This method applies the given animations to all fragment operations that follow it.

在[`FragmentTransaction`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction)中执行操作的顺序非常重要，特别是在使用`setCustomAnimations()`动画时。此方法将给定的动画应用于其后的所有fragment 操作。

```java
getSupportFragmentManager().beginTransaction()
        .setCustomAnimations(enter1, exit1, popEnter1, popExit1)
        .add(R.id.container, ExampleFragment.class, null) // gets the first animations
        .setCustomAnimations(enter2, exit2, popEnter2, popExit2)
        .add(R.id.container, ExampleFragment.class, null) // gets the second animations
        .commit()
```



## 限制fragment 的生命周期

`FragmentTransaction`可以影响事务范围内添加的单个fragment 的生命周期状态。在创建`FragmentTransactions`时，使用[`setMaxLifecycle()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#setMaxLifecycle(androidx.fragment.app.Fragment, androidx.lifecycle.Lifecycle.State))可以给fragment 设置一个最大的状态。例如， [`ViewPager2`](https://developer.android.google.cn/reference/androidx/viewpager2/widget/ViewPager2)使用 `setMaxLifecycle()`将屏幕外的fragments 限制在`STARTED`状态。



## 显示和隐藏 fragment 的视图

使用`FragmentTransaction`的 [`show()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#show(androidx.fragment.app.Fragment))和 [`hide()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#hide(androidx.fragment.app.Fragment)) 方法来显示和隐藏已经添加到容器中的fragments 的视图。这些方法设置fragments 视图的可见性，而不影响fragments 的生命周期。

虽然您不需要使用fragment事务来切换fragment中的视图的可见性，但是这些方法对于您想要可见状态的更改与回退栈上的事务相关联的情况很有用。



## Attaching and detaching fragments

The `FragmentTransaction` method [`detach()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#detach(androidx.fragment.app.Fragment)) detaches the fragment from the UI, destroying its view hierarchy. The fragment remains in the same state (`STOPPED`) as when it is put on the back stack. This means that the fragment was removed from the UI but is still managed by the fragment manager.

附加和分离fragment。`FragmentTransaction`的 [`detach()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#detach(androidx.fragment.app.Fragment))方法从UI中分离fragment ，销毁（destroying ）其view结构。当fragment放在回退栈（back stack）时，它仍然处于相同的状态 (`STOPPED`)。这意味着fragment 已从UI中删除，但仍由fragment 管理器管理。

The [`attach()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#attach(androidx.fragment.app.Fragment)) method reattaches a fragment from which it was previously detached. This causes its view hierarchy to be recreated, attached to the UI, and displayed.

 [`attach()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#attach(androidx.fragment.app.Fragment)) 方法重新附加以前分离（detached）过的fragment 。这将导致它的view结构被重新创建、附加到UI并显示出来。

As a `FragmentTransaction` is treated as a single atomic set of operations, calls to both `detach` and `attach` on the same fragment instance in the same transaction effectively cancel each other out, thus avoiding the destruction and immediate recreation of the fragment's UI. Use separate transactions, separated by `executePendingOperations()` if using `commit()`, if you want to detach and then immediately re-attach a fragment.

由于`FragmentTransaction`被视为单原子操作集，因此对同一事务中相同fragment 实例的`detach` 和`attach` 调用会有效地相互抵消，从而避免了fragment UI的破坏和立即重建。如果您想使用单独的事务来detach()，然后立即重新attach一个fragment，如果使用了 `commit()`，则用`executePendingOperations()`分隔。

> 我解释一下：因为会detach和attach会相互抵消，因此需要要让事务先执行一下。还没遇到过这种使用场景。



> 注意： `attach()`和`detach()`方法与fragment 的[`onAttach()`](https://developer.android.google.cn/reference/androidx/fragment/app/Fragment#onAttach(android.content.Context))和 [`onDetach()`](https://developer.android.google.cn/reference/androidx/fragment/app/Fragment#onDetach())方法无关。有关这些fragment方法的更多信息，请参见 [Fragment lifecycle](https://developer.android.google.cn/guide/fragments/lifecycle)。





# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、

