> version：2021/9/27
>
> review：

[原文地址](https://developer.android.google.cn/guide/fragments#videos)

目录

[TOC]



# Fragment

[`Fragment`](https://developer.android.google.cn/reference/androidx/fragment/app/Fragment) 表示应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。Fragment 不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。Fragment 的视图层次结构会成为宿主的视图层次结构的一部分，或附加到宿主的视图层次结构。

> **注意**：某些 [Android Jetpack](https://developer.android.google.cn/jetpack/androidx/versions) 库（如 [Navigation](https://developer.android.google.cn/guide/navigation)、[`BottomNavigationView`](https://developer.android.google.cn/reference/com/google/android/material/bottomnavigation/BottomNavigationView) 和 [`ViewPager2`](https://developer.android.google.cn/jetpack/androidx/releases/viewpager2)）经过精心设计，可与 Fragment 配合使用。

## 模块化

Fragment 允许您将界面划分为离散的区块，从而将**模块化**和**可重用性**引入 Activity 的界面。Activity 是围绕应用的界面放置全局元素（如抽屉式导航栏）的理想位置。相反，Fragment 更适合定义和管理单个屏幕或部分屏幕的界面。

假设有一个响应各种屏幕尺寸的应用。在较大的屏幕上，该应用应显示一个静态抽屉式导航栏和一个采用网格布局的列表。在较小的屏幕上，该应用应显示一个底部导航栏和一个采用线性布局的列表。在 Activity 中管理所有这些变化因素可能会很麻烦。将导航元素与内容分离可使此过程更易于管理。然后，Activity 负责显示正确的导航界面，而 Fragment 采用适当的布局显示列表。

![同一屏幕的采用不同屏幕尺寸的两个版本。](images/fragment-screen-sizes.png)

**图 1.** 同一屏幕的采用不同屏幕尺寸的两个版本。

在左侧，大屏幕包含一个由 Activity 控制的抽屉式导航栏和一个由 Fragment 控制的网格列表。在右侧，小屏幕包含一个由 Activity 控制的底部导航栏和一个由 Fragment 控制的线性列表。

将界面划分为 Fragment 可让您更轻松地在运行时修改 Activity 的外观。当 Activity 处于 `STARTED` [生命周期状态](https://developer.android.google.cn/guide/components/activities/activity-lifecycle)或更高的状态时，可以添加、替换或移除 Fragment。您可以将这些更改的记录保留在由 Activity 管理的返回堆栈中，从而允许撤消这些更改。

您可以在同一 Activity 或多个 Activity 中使用同一 Fragment 类的多个实例，甚至可以将其用作另一个 Fragment 的子级。考虑到这一点，<u>您应只为 Fragment 提供管理它自己的界面所需的逻辑。您应避免让一个 Fragment 依赖于另一个 Fragment 或从一个 Fragment 操控另一个 Fragment。</u>

## 后续步骤

如需查看与 Fragment 相关的更多文档和资源，请参阅以下内容。

### 开始使用

- [创建 Fragment](https://developer.android.google.cn/guide/fragments/create)

### 更深入的主题

- [Fragment 管理器](https://developer.android.google.cn/guide/fragments/fragmentmanager)
- [Fragment 事务](https://developer.android.google.cn/guide/fragments/transactions)
- [在 Fragment 之间添加动画过渡效果](https://developer.android.google.cn/guide/fragments/animate)
- [Fragment 生命周期](https://developer.android.google.cn/guide/fragments/lifecycle)
- [保存与 Fragment 相关的状态](https://developer.android.google.cn/guide/fragments/saving-state)
- [在 Fragment 与 Activity 之间通信](https://developer.android.google.cn/guide/fragments/communicate)
- [使用应用栏](https://developer.android.google.cn/guide/fragments/appbar)
- [使用 DialogFragment 显示对话框](https://developer.android.google.cn/guide/fragments/dialogs)
- [测试 Fragment](https://developer.android.google.cn/guide/fragments/test)

### 视频

- [一个 Activity：为何、何时以及如何（2018 年 Android 开发者峰会）](https://www.youtube.com/watch?v=2k8x8V77CrU)
- [Fragment：过去、现在和未来（2019 年 Android 开发者峰会）](https://www.youtube.com/watch?v=RS1IACnZLy4)





# 总结

1、可以认识到Fragment是什么。

2、有大量相关的知识还可以学习。



## 【精益求精】我还能做（补充）些什么？

1、文中的链接我还有很多没有查阅的。这些对知识体系的完善和扩充会很有帮助。