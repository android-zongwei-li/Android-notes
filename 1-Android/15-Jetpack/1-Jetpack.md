# Jetpack

对于大多数 Android 开发工程师来说，**Jetpack** 一定是一个既熟悉又陌生的东西。

通常在面试的最后阶段，面试官会问一下候选人对 Jetpack 的理解，旨在考察他们对目前 Android 主流开发技术的掌握程度以及是否还保持着对新技术持续学习的能力。

有人回答 Jetpack 就是 LiveData、ViewModel 这些东西，有人回答 Jetpack 是一套 MVVM 框架，当然更多人的回答是，听过、但没用过，所以也说不出它到底是什么。

## Jetpack是什么？

在 Jetpack 的官方文档中是这样对它定义的：

> Jetpack 是一套组件库，可帮助开发人员遵循最佳实践，减少样板代码并编写可在 Android 版本和设备上一致工作的代码，以便开发人员可以专注于他们关心的代码。

根据定义其实可以提炼出两个核心点：

1. 它是一套组件库。*（说明它是由许多个不同的组件库构成，并不是一个单一的组件库）*
2. 使用 Jetpack 可以帮助我们在不同的 Android 版本和不同的设备上，实现行为一致的工作代码。（说明 Jetpack 可以轻松的处理由 Android 版本不一致和设备不同产生的差异性和兼容性问题）

我们先来看看 Jetpack 包含哪些组件库

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e694745f9ae6496d9b6e22dc32a28b59~tplv-k3u1fbpfcp-watermark.image)

根据官网上的介绍，目前 Jetpack 一共有 85 个组件库，有些看着很熟悉，比如：**viewPager、fragment、recyclerview 等等**，但有些好像根本就没有见过，也没有用过。

为了弄清这 85 个组件库分别是做什么的，我们把每一个的文档都详细读一遍，包含它们是干什么的、如何依赖以及该如何使用，然后又分别在 Android Studio 中单独集成，目的就是想看看它们包中到底有什么类，到底是干什么的

再结合官方 Youtube 频道视频内容的介绍，在经过两天的梳理，我整理出了下面的内容。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b710d212c54499db33ff11c32bc2bd2~tplv-k3u1fbpfcp-watermark.image) ![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a76e747e6f54c6081b223ca42f27355~tplv-k3u1fbpfcp-watermark.image)

在知道了这 85 个组件库分别是做什么之后，接下来我对每一个库进行了分类和打标签，这么做的目的是可以帮助我在之后实际写代码的时候，可以快速的使用他们。

经过第二轮的梳理，将 Jetpack 的 85 个组件库进行了下面的分类和标签整理。

**第一个是核心类（8个）**，你也可以把它理解为基础类，也就是说我们一个最基本的 Android 工程都会默认依赖这些组件库。

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f3dfd8a5db54734adbb63b3640d6d51~tplv-k3u1fbpfcp-watermark.image)

**第二个是架构组件（10个）**，Jetpack 推出之后很令人兴奋的一点，就是 Google 引入了现代 Android 应用开发的架构指南，结合 MVVM 的架构设计，帮助我们轻松的处理 UI 与业务逻辑之间的关系。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22b4ee48097d4ad2b83b3c7389d2f9ab~tplv-k3u1fbpfcp-watermark.image)

**第三个是 UI 组件（22个）**，这里需要说明一点，大多数的 UI 组件其实都包含着核心组件中的 appcompat * 中了，这里列出的是 Jetpack 中以独立组件库存在的 UI 组件。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d832649d5a94caaad8f2de83fcdb234~tplv-k3u1fbpfcp-watermark.image)

**第四个是特殊业务组件（16个）**，根据不同的业务场景，选择性使用。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c965b10c6387445ba5e2286b9983bfeb~tplv-k3u1fbpfcp-watermark.image)

**第五个是用不着的组件（15个）**，这个完全是出于个人出发，对于从事 Android 互联网项目的开发者来说，涉及游戏、车载、TV 等或平时极少使用的组件，都规整到这一类中了。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d78e8e53536d41c9ae08aed82730986d~tplv-k3u1fbpfcp-watermark.image)

**第六个是弃用的组件（11个）**，有一些是因为官方不再更新维护了，有一些是在 Jetpack 中有更好的替代解决方案，如果我们的项目中还在使用这些组件库的话，建议尽快替换到最新的替代组件上。

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5789d88f1654eaba5110408e59ba91d~tplv-k3u1fbpfcp-watermark.image)

**第七个是用于测试的组件（2个）。**

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/daaaab8718c746549c3fba7ba7e344f2~tplv-k3u1fbpfcp-watermark.image)

看到这里，相信大家应该都理解了最开始的定义中，我们提炼出的第一点内容：**Jetpack 是一套组件库。**没错 Jetpack 是由 85 个组件库构成的，每一个都可以根据自己的需求单独依赖使用，非常灵活和方面。

同时经过梳理，希望可以帮助大家更好的了解了这 85 个组件库分别是做什么的，也希望大家可以在通过标签分类之后，可以快速的在不同场景下，选择合适的组件，帮助自己完成对应功能的实现。