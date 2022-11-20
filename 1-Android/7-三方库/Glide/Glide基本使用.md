> 前言：Glide作为一个优秀的图片加载框架，它涉及到的知识点很丰富，学习其源码对于理解Android大有脾益。此篇先总结Glide的基本使用，并归纳它的优点。后续篇章将从它的优点出发，去学习其实现以及背后涉及到的原理。



#一、简介与优势
**1、简介**
Glide作为一个图片加载框架，功能全、性能高，使用简单。

**2、Glide优点**
1、支持多种数据源，本地、网络、assets、gif等都支持。
2、生命周期集成到Glide
3、高效处理Bitmap；使用Bitmap pool复用Bitmap
4、高效缓存，支持memory和disk图片缓存，默认使用二级缓存
5、图片加载过程可以监听
6、可配置度高，自适应高

#二、基本使用
######1、加载图片基本流程
加载图片的核心代码就一行，通过这行代码，可以完成图片的加载与展示。
```
        Glide.with(this).load(url2).into(ivBg);
```
分析：
（1）Glide.with()
用于创建一个加载图片的实例。with()方法可以接收Context、Activity、Fragment、View等类型的参数。with()方法传入的实例会决定Glide加载图片的生命周期，如果传入的是Activity或者Fragment实例，那么当这个Activity或Fragment销毁时，图片加载也会停止。如果传入的是ApplicationContext，那么只有当应用程序被杀掉的时候，图片加载才会停止。
（2）load()
用于指定要加载的图片资源，可以来自网络、本地、应用、二进制流、Uri对象等。
（3）into()
让图片显示到指定控件。

######2、占位图（加载占位图/异常占位图）
加载占位图用来解决网络加载时有段时间图片空白的情况。异常占位符解决加载失败的情况。
```
Glide.with(MainActivity.this).load(url)
        .placeholder(R.mipmap.ic_loading)
        .error(R.drawable.error)
        .into(ivBg);
```

######3、缓存策略设置
目前有一种情况，同一个url，服务端对应的图片改了，此时，如果有缓存，那就和预期不符了。下面是禁用缓存的方法。
```
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
```

######4、指定图片格式（静态/Gif）
默认情况是不需要设置的，Glide会自动判断格式。
但是如果必须加载静态，可以用asBitmap，这样如果是gif，那么会显示第一帧。
```
Glide.with(this)
     .load(url)
     .asBitmap()
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .into(imageView);
```
指定Gif格式，如果图片不是Gif，那么会显示error图片。
```
Glide.with(this)
     .load(url)
     .asGif()
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .into(imageView);
```

######5、指定图片大小
使用override()方法指定图片大小，默认情况都是根据ImageView的大小来决定图片大小。
```
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .override(100, 100)
     .into(imageView);
```
Glide加载图片是用到多少加载多少，这样可以避免内存浪费。





写于：2020/12/03
参考：
1、[ Android图片加载框架最全解析（一），Glide的基本用法](https://guolin.blog.csdn.net/article/details/53759439)