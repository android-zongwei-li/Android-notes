# 前言

- 在 `Android`开发中，性能优化策略十分重要
- 本文主要讲解性能优化中的**Bitmap 使用优化**，希望你们会喜欢

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-5c0bb3bc6fb9921e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 1. 优化原因

即 为什么要优化图片`Bitmap`资源，具体如下图：

![img](https:////upload-images.jianshu.io/upload_images/944365-e971c9d139a03b35.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图



------

# 2. 优化方向

本文将从 以下方面优化图片`Bitmap`资源的使用 & 内存管理

![img](https:////upload-images.jianshu.io/upload_images/944365-dc9e30c3c2e678b8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图



------

# 3. 具体优化方案

下面，我将详细讲解每个优化方向的具体优化方案

![img](https:////upload-images.jianshu.io/upload_images/944365-34d13c4ab292b8db.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

### 3.1 使用完毕后 释放图片资源

- 优化原因
   使用完毕后若不释放图片资源，容易造成**内存泄露，从而导致内存溢出**
- 优化方案
   a. 在 `Android2.3.3（API 10）`前，调用 `Bitmap.recycle()`方法
   b. 在 `Android2.3.3（API 10）`后，采用软引用`（SoftReference）`
- 具体描述
   在 `Android2.3.3（API 10）`前、后，Bitmap对象 & 其像素数据 的存储位置不同，从而导致释放图片资源的方式不同，具体如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-8b6b26416a085516.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

> 注：若调用了`Bitmap.recycle()后`，再绘制`Bitmap`，则会出现`Canvas: trying to use a recycled bitmap`错误

### 3.2 根据分辨率适配 & 缩放图片

- 优化原因
   若 `Bitmap` 与 当前设备的分辨率不匹配，则会拉伸`Bitmap`，而`Bitmap`分辨率增加后，所占用的内存也会相应增加

> 因为`Bitmap` 的内存占用 根据 `x`、`y`的大小来增加的

- 优化方案

![img](https:////upload-images.jianshu.io/upload_images/944365-484f123d279d8409.png?imageMogr2/auto-orient/strip|imageView2/2/w/1110/format/webp)

示意图

关于图片资源适配屏幕分辨率，具体请看文章：[Android 屏幕适配：最全面的解决方案](https://www.jianshu.com/p/ec5a1a30694b)

### 3.3 按需 选择合适的解码方式

- 优化原因
   不同的图片解码方式 对应的 内存占用大小 相差很大，具体如下

  ![img](https:////upload-images.jianshu.io/upload_images/944365-d79f856e0559076b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1020/format/webp)

  示意图

- 优化方案
   根据需求 选择合适的解码方式

> 1. 使用参数：`BitmapFactory.inPreferredConfig` 设置
> 2. 默认使用解码方式：`ARGB_8888`

### 3.4 设置 图片缓存

- 优化原因
   重复加载图片资源耗费太多资源（`CPU`、内存 & 流量）
- 优化方案

![img](https:////upload-images.jianshu.io/upload_images/944365-c81497ee8be27d81.png?imageMogr2/auto-orient/strip|imageView2/2/w/592/format/webp)

示意图

> 关于三级缓存机制，此处不作过多描述，具体请看文章：[三级缓存说明](https://www.jianshu.com/p/2cd59a79ed4a)

至此，关于图片资源`Bitmap`的使用优化讲解完毕

------

# 4. 总结

- 本文全面总结了图片资源`Bitmap`的使用优化，具体如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-c8a9bd51aad11e1a.png?imageMogr2/auto-orient/strip|imageView2/2/w/860/format/webp)



# 参考

[Android 性能优化：手把手教你优化Bitmap图片资源的使用](https://www.jianshu.com/p/7643c6aadb53)