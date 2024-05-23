# 前言

- 上文已经对当今**Android主流的图片加载库**进行了全面介绍 & 对比

> 如果你还没阅读，我建议你先[移步这里阅读](https://www.jianshu.com/p/97994c9693f9)

- 今天我们来学习其中一个Android主流的图片加载库的使用 - Picasso

> Carson带你学Android开源库系列文章：
>  [Carson带你学Android：主流开源图片加载库对比(UIL、Picasso、Glide、Fresco)](https://www.jianshu.com/p/97994c9693f9)
>  [Carson带你学Android：主流开源网络请求库对比(Volley、OkHttp、Retrofit)](https://www.jianshu.com/p/050c6db5af5a)
>  [Carson带你学Android：网络请求库Retrofit使用教程](https://www.jianshu.com/p/a3e162261ab6)
>  [Carson带你学Android：网络请求库Retrofit源码分析](https://www.jianshu.com/p/a3e162261ab6)
>  [Carson带你学Android：图片加载库Glide使用教程](https://www.jianshu.com/p/c3a5518b58b2)
>  [Carson带你学Android：图片加载库Glide源码分析](https://www.jianshu.com/p/c3a5518b58b2)
>  [Carson带你学Android：V-Layout，淘宝、天猫都在用的UI框架，赶紧用起来吧！](https://www.jianshu.com/p/6b658c8802d1)

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-8f518070eff52d05.png?imageMogr2/auto-orient/strip|imageView2/2/w/992/format/webp)

目录

------

# 1. 简介

- 介绍：Picasso，可译为“毕加索”，是Android中一个图片加载开源库

> 大概是因为其使用使用方法简单、优雅所以这样取名

- 主要作用：实现图片加载

# 2. 功能特点

## 2.1 功能列表

![img](https:////upload-images.jianshu.io/upload_images/944365-1464f8627b71f758.png?imageMogr2/auto-orient/strip|imageView2/2/w/691/format/webp)

功能列表

- 从上面可以看出，Picasso不仅实现了图片异步加载的功能，还解决了Android中加载图片时需要解决的一些常见问题
- 接下来，我会对Picasso的每个功能点进行详细的介绍

## 2.2 功能介绍

**2.2.1 图片的异步加载（最基础功能）**



```dart
ImageView targetImageView = (ImageView) findViewById(R.id.ImageView);
        String Url = "http://218.192.170.132/1.jpg";

//Picasso使用了流式接口的调用方式
//Picasso类是核心实现类。
//实现图片加载功能至少需要三个参数：
        Picasso
//with(Context context)
//Context对于很多Android API的调用都是必须的，这里就不多说了
                .with(context)

//load(String imageUrl)：被加载图像的Url地址。
//大多情况下，一个字符串代表一个网络图片的URL。
                .load(Url)

//into(ImageView targetImageView)：图片最终要展示的地方。
                .into(targetImageView);
```

**2.2.2 图片转换**
 使用最少的内存完成复杂的图片转换，转换图片以适合所显示的ImageView，来减少内存消耗



```csharp
Picasso.with(context)
  .load(url)
//裁剪图片尺寸
  .resize(50, 50)
//设置图片圆角
  .centerCrop()
  .into(imageView)
```

**2.2.3 加载过重 & 错误处理**
 Picasso支持加载过程中和加载错误时显示对应图片



```csharp
Picasso.with(context)
    .load(url)
//加载过程中的图片显示
    .placeholder(R.drawable.user_placeholder)
//加载失败中的图片显示
//如果重试3次（下载源代码可以根据需要修改）还是无法成功加载图片，则用错误占位符图片显示。
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
```

**2.2.4 在Adapter中的回收不在视野的ImageView和取消已经回收的ImageView下载进程**



```csharp
@Override 
public void getView(int position, View convertView, ViewGroup parent) {
  SquaredImageView view = (SquaredImageView) convertView;
  if (view == null) {
    view = new SquaredImageView(context);
  }
  String url = getItem(position);
 
  Picasso.with(context).load(url).into(view);
}
```

**2.2.5 从不同资源源加载**
 支持多种数据源  网络、本地、资源、Assets 等



```csharp
//加载资源文件
Picasso.with(context).load(R.drawable.landing_screen).into(imageView1);
//加载本地文件
Picasso.with(context).load(new File("/images/oprah_bees.gif")).into(imageView2);
```

**2.2.6 自动添加磁盘和内存二级缓存功能**

**2.2.7 支持优先级处理**
 每次任务调度前会选择优先级高的任务，比如 App 页面中 Banner 的优先级高于 Icon 时就很适用。

**2.2.8 支持飞行模式、并发线程数根据网络类型而变**
 手机切换到飞行模式或网络类型变换时会自动调整线程池最大并发数，比如 wifi 最大并发为 4， 4g 为 3，3g 为 2

**2.2.9 “无”本地缓存**
 无”本地缓存，不是说没有本地缓存，而是 Picasso 自己没有实现，交给了 Square 的另外一个网络库 okhttp 去实现，这样的好处是可以通过请求 Response Header 中的 Cache-Control 及 Expired 控制图片的过期时间。

------

# 3. Demo实例

没有Demo的代码讲解不是好文章，让我们来一步步学会使用Picasso。

**步骤1：在gradle添加依赖**



```bash
compile 'com.squareup.picasso:picasso:2.5.2'
```

**步骤2：添加网络权限**



```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

> 步骤1和步骤2是Picasso使用的前提，千万别忘了！！！！

**步骤3：在MainActivity中**



```dart
ImageView targetImageView = (ImageView) findViewById(R.id.ImageView);
        String Url = "http://218.192.170.132/1.jpg";

        Picasso
                .with(this)
                .load(Url)
                .into(targetImageView);
```

> 还有具体其他功能需要配置的自己按照我上面写的进行配置就好了~

这里再贴上Picasso的Github地址：[请点击这里](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsquare%2Fpicasso)

------

# 4. 特点

#### 4.1 优点

- 使用简单、方便（一行代码的事情）
- 由于同样是出品自Square的，Square 公司的其他开源库如 Retrofit 或者 OkHttp和Picasso搭配使用兼容性会更好些，占用体积也会少些

> 所以，如果项目已经使用了 Square 公司的其他开源库（如 Retrofit 或者 OkHttp），在满足需求的前提下建议使用Picasso

#### 4.2 缺点

- 功能较为简单-图片加载；
- 性能（加载速度等等）较其他图片加载库（Glide、Fresco）较差
- 自身无实现“本地缓存”

------

# 5. 总结

- Picasso使用起来是不是非常简单？相信你看完这篇文章后你能全面掌握Picasso的用法



# 参考

[Carson带你学Android：图片加载库Picasso全面讲解](https://www.jianshu.com/p/51dc758b52f9)