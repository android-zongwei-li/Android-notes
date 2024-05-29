当前字数：37406

目标：精简到30000以下

# 一、概览

GitHub：https://github.com/bumptech/glide

Glide 是一个图片加载框架，使用简单，功能全、性能高。

**Glide优点**
1、支持多种数据源，本地、网络、assets、gif等都支持。
2、生命周期集成到Glide
3、高效处理Bitmap；使用Bitmap pool复用Bitmap
4、高效缓存，支持memory和disk图片缓存，默认使用二级缓存
5、图片加载过程可以监听
6、可配置度高，自适应高

## 功能概览

![img](images/Glide基本使用/webp-1706801581628-80.webp)

# 二、快速使用

## 添加依赖

```kotlin
// https://github.com/bumptech/glide
implementation 'com.github.bumptech.glide:glide:4.16.0'
```

添加权限

```kotlin
<uses-permission android:name="android.permission.INTERNET" />
```

## 加载图片

一张测试用的图片：http://cn.bing.com/az/hprichbg/rb/Dongdaemun_ZH-CN10736487148_1920x1080.jpg

一行核心代码就可以完成图片的加载与展示：

```kotlin
Glide.with(this).load(url).into(ivBg);
```
## 示例

**Activity**

```kotlin
public class MainActivity extends AppCompatActivity {

    ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
    }

    public void loadImage(View view) {
        String url = "http://cn.bing.com/az/hprichbg/rb/Dongdaemun_ZH-CN10736487148_1920x1080.jpg";
        Glide.with(this).load(url).into(imageView);
    }

}
```

**activity_main.xml**

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Load Image"
        android:onClick="loadImage"
        />

    <ImageView
        android:id="@+id/image_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

这样图片就加载完成了。

Glide最基本的使用方式，就是关键的三步：先with()，再load()，最后into()。

# 三、核心方法

## Glide.with()

创建一个加载图片的实例。with()方法可以接收Context、Activity、Fragment、View等类型的参数。

```dart
        .with(Context context)// 绑定Context
        .with(Activity activity);// 绑定Activity
        .with(FragmentActivity activity);// 绑定FragmentActivity
        .with(Fragment fragment);// 绑定Fragment
```

with() 方法传入的实例会决定Glide加载图片的生命周期：

- 如果传入的是Activity或者Fragment实例，那么当这个Activity或Fragment销毁时，图片加载也会停止。
- 如果传入的是ApplicationContext，那么只有当应用程序被杀掉的时候，图片加载才会停止。

## load()

指定要加载的图片资源，可以来自网络、本地、应用、二进制流、Uri对象等。

load 重载方法使用：

```kotlin
// 加载本地图片
File file = new File(getExternalCacheDir() + "/image.jpg");
Glide.with(this).load(file).into(imageView);

// 加载应用资源
int resource = R.drawable.image;
Glide.with(this).load(resource).into(imageView);

// 加载二进制流
byte[] image = getImageBytes();
Glide.with(this).load(image).into(imageView);

// 加载Uri对象
Uri imageUri = getImageUri();
Glide.with(this).load(imageUri).into(imageView);
```

## into()

让图片显示到指定控件。into()方法不仅仅是只能接收ImageView类型的参数，还支持很多更丰富的用法，后面会详述。

# 四、功能列表

## 占位图（加载占位图/异常占位图）

即在加载中以及加载失败时显示的图片

```kotlin
Glide.with(MainActivity.this).load(url)
				// 加载占位图
        .placeholder(R.mipmap.ic_loading)
				// 异常占位图
        .error(R.drawable.error)
        .into(ivBg);
```

由于 Glide 具有缓存机制，所以二次加载图片时，占位图可能来不及显示。

## 缓存

- 设置磁盘缓存策略

```csharp
Glide.with(this)
    .load(imageUrl)
    .diskCacheStrategy(DiskCacheStrategy.ALL)
    .into(imageView);

// 缓存参数说明
// DiskCacheStrategy.NONE：不缓存任何图片，即禁用磁盘缓存
// DiskCacheStrategy.ALL ：缓存原始图片 & 转换后的图片（默认）
// DiskCacheStrategy.SOURCE：只缓存原始图片（原来的全分辨率的图像，即不缓存转换后的图片）
// DiskCacheStrategy.RESULT：只缓存转换后的图片（即最终的图像：降低分辨率后 / 或者转换后 ，不缓存原始图片
```

有一种情况，同一个url，服务端对应的图片改了，此时，如果有缓存，那就和预期不符了。下面是禁用缓存的方法。

```kotlin
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
```

- 设置跳过内存缓存

```kotlin
Glide
  .with(this)
.load(imageUrl)
.skipMemoryCache(true)
.into(imageView);
// 设置跳过内存缓存
// 这意味着 Glide 将不会把这张图片放到内存缓存中去
// 只会影响内存缓存，Glide 仍然会利用磁盘缓存来避免重复的网络请求。
```

- 清理缓存

```kotlin
// 清理磁盘缓存 需要在子线程中执行
Glide.get(this).clearDiskCache();
// 清理内存缓存 可以在UI主线程中进行
Glide.get(this).clearMemory();
```

## 指定图片大小

> 我们平时在加载图片的时候很容易会造成内存浪费。什么叫内存浪费呢？比如说一张图片的尺寸是1000 * 1000像素，但是我们界面上的ImageView可能只有200 * 200像素，这个时候如果不对图片进行任何压缩就直接读取到内存中，这就属于内存浪费了，因为程序中根本就用不到这么高像素的图片。
>
> [Android高效加载大图、多图解决方案，有效避免程序OOM](https://blog.csdn.net/guolin_blog/article/details/9316683)

Glide 默认是根据 ImageView 的大小来决定图片大小（用到多少加载多少，可以避免内存浪费）。

可以通过override()方法指定图片大小。

```kotlin
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .override(100, 100)
     .into(imageView);
```

## 指定图片格式（静态/Gif）

默认情况是不需要设置的，Glide会自动判断格式。
但如果必须加载静态，可以用 asBitmap，这样如果是gif，那么会显示第一帧。

```kotlin
Glide.with(this)
     .load(url)
     .asBitmap()
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .into(imageView);
```

指定Gif格式，如果图片不是Gif，那么会显示error图片。

```kotlin
Glide.with(this)
     .load(url)
     .asGif()
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .into(imageView);
```

## 加载动画

```kotlin
Glide.with(this).load(imageUrl).animate(R.anim.item_alpha_in).into(imageView);
```

 *R.anim.item_alpha_in*

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha
        android:duration="500"
        android:fromAlpha="0.0"
        android:toAlpha="1.0"/>
</set>
```

api 提供了几个常用的动画：比如crossFade()

## 设置要加载的内容

如果需要先下载图片然后再做一些合成的功能，比如图文混排，实现如下：

```java
Glide.with(this).load(imageUrl).centerCrop().into(new SimpleTarget<GlideDrawable>() {
            @Override
            public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
                imageView.setImageDrawable(resource);
            }
        });
```

## 多样式的媒体加载

```csharp
   Glide
        .with(context)
        .load(imageUrl)；
        .thumbnail(0.1f)；//设置缩略图支持：先加载缩略图 然后在加载全图
                          //传了一个 0.1f 作为参数，Glide 将会显示原始图像的10%的大小。
                          //如果原始图像有 1000x1000 像素，那么缩略图将会有 100x100 像素。
        .asBitmap()//显示gif静态图片 
        .asGif();//显示gif动态图片
        .into(imageView)；
```

## 设置动态转换

```kotlin
Glide.with(this).load(imageUrl).centerCrop().into(imageView);
```

## 设置下载优先级

```kotlin
Glide.with(this).load(imageUrl).priority(Priority.NORMAL).into(imageView);
```

# 配置OkHttp请求网络

```kotlin
//Glide库
implementation 'com.github.bumptech.glide:glide:4.13160'

// kotlin项目先引入plugin，然后是用kapt引入，切记，不然可能导致无法生成自定义GlideModule的实现类
plugins {
    id 'kotlin-kapt'
}
kapt 'com.github.bumptech.glide:compiler:4.16.0'

// Glide集成OkHttp时需要使用的库，库已经将需要适配Okhhtp的大部分代码封装
implementation "com.github.bumptech.glide:okhttp3-integration:4.16.0"

// 提供 Log 拦截器实现类。HttpLoggingInterceptor() 
implementation "com.squareup.okhttp3:logging-interceptor:3.14.9"
```

实现类：GlideOkHttpModule

```kotlin
@GlideModule
class GlideOkHttpModule : AppGlideModule() {
    
    override fun registerComponents(context: Context, glide: Glide, registry: Registry) {
        val httpLoggingInterceptor = HttpLoggingInterceptor()
        httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY)
        
        Log.i(TAG, "registerComponents: ")
        val client: OkHttpClient = OkHttpClient.Builder()
                .retryOnConnectionFailure(true)
                .addInterceptor(httpLoggingInterceptor)
                .connectTimeout(6, TimeUnit.SECONDS)
                .build()
        
        registry.replace(GlideUrl::class.java, InputStream::class.java, OkHttpUrlLoader.Factory(client))
    }
}
```

# 回调和监听功能

## 自定义Target功能

使用 into() 方法时，通过自定义 Target 可以更自由地控制Glide的回调。

下面是 Glide 中Target的继承结构图：

![img](images/Glide基本使用/SouthEast.png)

如果自定义 Target，通常是基于 SimpleTarget 或 ViewTarget。

### 示例：基于 SimpleTarget

使用它可以获取到加载完成的图片，而不像 into(ImageView) 直接在 ImageView 上显示出来。

```kotlin
SimpleTarget<GlideDrawable> simpleTarget = new SimpleTarget<GlideDrawable>() {
    @Override
    public void onResourceReady(GlideDrawable resource, GlideAnimation glideAnimation) {
        imageView.setImageDrawable(resource);
    }
};

public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
         .load(url)
         .into(simpleTarget);
}
```

创建一个SimpleTarget的实例，并指定它的泛型是GlideDrawable，重写了onResourceReady()方法。在onResourceReady()方法中，就可以获取到Glide加载出来的图片对象GlideDrawable，有了这个对象后可以使用它进行任意的逻辑操作。

SimpleTarget中的泛型并不一定只能是GlideDrawable，如果能确定正在加载的是一张静态图而不是GIF图的话，还能直接拿到这张图的Bitmap对象，如下所示：

```kotlin
SimpleTarget<Bitmap> simpleTarget = new SimpleTarget<Bitmap>() {
    @Override
    public void onResourceReady(Bitmap resource, GlideAnimation glideAnimation) {
        imageView.setImageBitmap(resource);
    }
};

public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
         .load(url)
         .asBitmap()
         .into(simpleTarget);
}
```

**基于此，我们甚至还能实现图片下载功能，将保存逻辑放到 onResourceReady 中即可。**

### 示例：基于 ViewTarget

Glide在内部创建的GlideDrawableImageViewTarget就是ViewTarget的子类。只不过GlideDrawableImageViewTarget被限定只能作用在ImageView上，而ViewTarget的功能更加广泛，它可以作用在任意的View上。

首先创建一个自定义布局MyLayout：

```kotlin
public class MyLayout extends LinearLayout {
	private ViewTarget<MyLayout, GlideDrawable> viewTarget;

public MyLayout(Context context, AttributeSet attrs) {
    super(context, attrs);
    viewTarget = new ViewTarget<MyLayout, GlideDrawable>(this) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation glideAnimation) {
            MyLayout myLayout = getView();
            myLayout.setImageAsBackground(resource);
        }
    };
}

public ViewTarget<MyLayout, GlideDrawable> getTarget() {
    return viewTarget;
}

public void setImageAsBackground(GlideDrawable resource) {
    setBackground(resource);
}
}
```


在MyLayout的构造函数中，创建了一个ViewTarget的实例，并将Mylayout当前的实例this传了进去。ViewTarget中需要指定两个泛型，一个是View的类型，一个图片的类型（GlideDrawable或Bitmap）。在onResourceReady()方法中可以通过getView()方法获取到MyLayout的实例，并调用它的任意接口。

接下来看一下怎么使用这个Target吧，由于MyLayout中已经提供了getTarget()接口，只需要在加载图片的地方这样写就可以了：

```kotlin
public class MainActivity extends AppCompatActivity {
MyLayout myLayout;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    myLayout = (MyLayout) findViewById(R.id.background);
}

public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
         .load(url)
         .into(myLayout.getTarget());
}
}
```

# preload()方法

Glide加载图片虽说非常智能，它会自动判断该图片是否已经有缓存了，如果有的话就直接从缓存中读取，没有的话再从网络去下载。但是如果希望提前对图片进行一个预加载，等真正需要加载图片的时候就直接从缓存中读取，不想再等待慢长的网络加载时间了，这该怎么办呢？

preload()方法有两个方法重载，一个不带参数，表示将会加载图片的原始尺寸，另一个可以通过参数指定加载图片的宽和高。

```kotlin
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.SOURCE)
     .preload();
```

注意，如果使用了preload()方法，最好将diskCacheStrategy的缓存策略指定成DiskCacheStrategy.SOURCE。因为preload()方法默认是预加载的原始图片大小，而into()方法则默认会根据ImageView控件的大小来动态决定加载图片的大小。因此，如果不将diskCacheStrategy的缓存策略指定成DiskCacheStrategy.SOURCE的话，很容易会造成我们在预加载完成之后再使用into()方法加载图片，却仍然还是要从网络上去请求图片这种现象。

调用了预加载之后，再去加载这张图片就会非常快了，因为Glide会直接从缓存当中去读取图片并显示出来，代码如下所示：

```kotlin
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.SOURCE)
     .into(imageView);
```

注意，这里仍然需要使用diskCacheStrategy()方法将硬盘缓存策略指定成DiskCacheStrategy.SOURCE，以保证Glide一定会去读取刚才预加载的图片缓存。

# downloadOnly()方法

一直以来，我们使用Glide都是为了将图片显示到界面上。虽然我们知道Glide会在图片的加载过程中对图片进行缓存，但是缓存文件到底是存在哪里的，以及如何去直接访问这些缓存文件？我们都还不知道。

其实Glide将图片加载接口设计成这样也是希望我们使用起来更加的方便，不用过多去考虑底层的实现细节。但如果我现在就是想要去访问图片的缓存文件该怎么办呢？这就需要用到downloadOnly()方法了。

和preload()方法类似，downloadOnly()方法也是可以替换into()方法的，不过downloadOnly()方法的用法明显要比preload()方法复杂不少。顾名思义，downloadOnly()方法表示只会下载图片，而不会对图片进行加载。当图片下载完成之后，我们可以得到图片的存储路径，以便后续进行操作。

那么首先我们还是先来看下基本用法。downloadOnly()方法是定义在DrawableTypeRequest类当中的，它有两个方法重载，一个接收图片的宽度和高度，另一个接收一个泛型对象，如下所示：

```kotlin
downloadOnly(int width, int height)
downloadOnly(Y target)
```

这两个方法各自有各自的应用场景，其中downloadOnly(int width, int height)是用于在子线程中下载图片的，而downloadOnly(Y target)是用于在主线程中下载图片的。

## downloadOnly(int width, int height)

示例

```kotlin
public void downloadImage(View view) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
                final Context context = getApplicationContext();
                FutureTarget<File> target = Glide.with(context)
                                                 .load(url)
                                                 .downloadOnly(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
                final File imageFile = target.get();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(context, imageFile.getPath(), Toast.LENGTH_LONG).show();
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }).start();
}
```

1. downloadOnly(int width, int height) 方法必须要用在子线程当中；

2. 在子线程当中，先获取了一个Application Context，这个时候不能再用Activity作为Context了，因为会有Activity销毁了但子线程还没执行完这种可能出现。

3. 当调用了downloadOnly(int width, int height)方法后会立即返回一个FutureTarget对象，然后Glide会在后台开始下载图片文件。

   接下来调用FutureTarget的get()方法就可以去获取下载好的图片文件了，如果此时图片还没有下载完，那么get()方法就会阻塞住，一直等到图片下载完成才会有值返回。

4. 最后使用runOnUiThread()切回到主线程，然后使用Toast将下载好的图片文件路径显示出来。

之后可以使用如下代码去加载这张图片，图片就会立即显示出来，而不用再去网络上请求了：

```kotlin
public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .diskCacheStrategy(DiskCacheStrategy.SOURCE)
            .into(imageView);
}
```

需要注意的是，这里必须将硬盘缓存策略指定成DiskCacheStrategy.SOURCE或者DiskCacheStrategy.ALL，否则Glide将无法使用刚才下载好的图片缓存文件。

## downloadOnly(Y target)方法

回想一下，其实downloadOnly(int width, int height)方法必须使用在子线程当中，最主要还是因为它在内部帮我们自动创建了一个RequestFutureTarget，是这个RequestFutureTarget要求必须在子线程当中执行。而downloadOnly(Y target)方法则要求我们传入一个自己创建的Target，因此就不受RequestFutureTarget的限制了。

但是downloadOnly(Y target)方法的用法也会相对更复杂一些，因为我们又要自己创建一个Target了，而且这次必须直接去实现最顶层的Target接口，比之前的SimpleTarget和ViewTarget都要复杂不少。

那么下面我们就来实现一个最简单的DownloadImageTarget吧，注意Target接口的泛型必须指定成File对象，这是downloadOnly(Y target)方法要求的，代码如下所示：

```kotlin
public class DownloadImageTarget implements Target<File> {
private static final String TAG = "DownloadImageTarget";

@Override
public void onStart() {
}

@Override
public void onStop() {
}

@Override
public void onDestroy() {
}

@Override
public void onLoadStarted(Drawable placeholder) {
}

@Override
public void onLoadFailed(Exception e, Drawable errorDrawable) {
}

@Override
public void onResourceReady(File resource, GlideAnimation<? super File> glideAnimation) {
    Log.d(TAG, resource.getPath());
}

@Override
public void onLoadCleared(Drawable placeholder) {
}

@Override
public void getSize(SizeReadyCallback cb) {
    cb.onSizeReady(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
}

@Override
public void setRequest(Request request) {
}

@Override
public Request getRequest() {
    return null;
}
}
```

由于是要直接实现Target接口，因此需要重写的方法非常多。这些方法大多是数Glide加载图片生命周期的一些回调，我们可以不用管它们，其中只有两个方法是必须实现的，一个是getSize()方法，一个是onResourceReady()方法。

在第二篇Glide源码解析的时候，我带着大家一起分析过，Glide在开始加载图片之前会先计算图片的大小，然后回调到onSizeReady()方法当中，之后才会开始执行图片加载。而这里，计算图片大小的任务就交给我们了。只不过这是一个最简单的Target实现，我在getSize()方法中就直接回调了Target.SIZE_ORIGINAL，表示图片的原始尺寸。

然后onResourceReady()方法我们就非常熟悉了，图片下载完成之后就会回调到这里，我在这个方法中只是打印了一下下载的图片文件的路径。

这样一个最简单的DownloadImageTarget就定义好了，使用它也非常的简单，我们不用再考虑什么线程的问题了，而是直接把它的实例传入downloadOnly(Y target)方法中即可，如下所示：

```kotlin
public void downloadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .downloadOnly(new DownloadImageTarget());
}
```


现在重新运行一下代码并点击Download Image按钮，然后观察控制台日志的输出，结果如下图所示。


这样我们就使用了downloadOnly(Y target)方法同样获取到下载的图片文件的缓存路径了。

好的，那么关于downloadOnly()方法我们就学到这里。

listener()方法
今天学习的内容已经够多了，下面我们就以一个简单的知识点结尾吧，Glide回调与监听的最后一部分——listener()方法。

其实listener()方法的作用非常普遍，它可以用来监听Glide加载图片的状态。举个例子，比如说我们刚才使用了preload()方法来对图片进行预加载，但是我怎样确定预加载有没有完成呢？还有如果Glide加载图片失败了，我该怎样调试错误的原因呢？答案都在listener()方法当中。

首先来看下listener()方法的基本用法吧，不同于刚才几个方法都是要替换into()方法的，listener()是结合into()方法一起使用的，当然也可以结合preload()方法一起使用。最基本的用法如下所示：

```kotlin
public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .listener(new RequestListener<String, GlideDrawable>() {
                @Override
                public boolean onException(Exception e, String model, Target<GlideDrawable> target,
                    boolean isFirstResource) {
                    return false;
                }

            @Override
            public boolean onResourceReady(GlideDrawable resource, String model,
                Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
                return false;
            }
       })
  	.into(imageView);

}
```

这里我们在into()方法之前串接了一个listener()方法，然后实现了一个RequestListener的实例。其中RequestListener需要实现两个方法，一个onResourceReady()方法，一个onException()方法。从方法名上就可以看出来了，当图片加载完成的时候就会回调onResourceReady()方法，而当图片加载失败的时候就会回调onException()方法，onException()方法中会将失败的Exception参数传进来，这样我们就可以定位具体失败的原因了。

没错，listener()方法就是这么简单。不过还有一点需要处理，onResourceReady()方法和onException()方法都有一个布尔值的返回值，返回false就表示这个事件没有被处理，还会继续向下传递，返回true就表示这个事件已经被处理掉了，从而不会再继续向下传递。举个简单点的例子，如果我们在RequestListener的onResourceReady()方法中返回了true，那么就不会再回调Target的onResourceReady()方法了。

关于listener()方法的用法就讲这么多，不过还是老规矩，我们再来看一下它的源码是怎么实现的吧。

首先，listener()方法是定义在GenericRequestBuilder类当中的，而我们传入到listener()方法中的实例则会赋值到一个requestListener变量当中，如下所示：

```kotlin
public class GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> implements Cloneable {

private RequestListener<? super ModelType, TranscodeType> requestListener;
...

public GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> listener(
        RequestListener<? super ModelType, TranscodeType> requestListener) {
    this.requestListener = requestListener;
    return this;
}

...

}
```

接下来在构建GenericRequest的时候这个变量也会被一起传进去，最后在图片加载完成的时候，我们会看到如下逻辑：

```kotlin
public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {

private RequestListener<? super A, R> requestListener;
...

private void onResourceReady(Resource<?> resource, R result) {
    boolean isFirstResource = isFirstReadyResource();
    status = Status.COMPLETE;
    this.resource = resource;
    if (requestListener == null || !requestListener.onResourceReady(result, model, target,
            loadedFromMemoryCache, isFirstResource)) {
        GlideAnimation<R> animation = animationFactory.build(loadedFromMemoryCache, isFirstResource);
        target.onResourceReady(result, animation);
    }
    notifyLoadSuccess();
}
...

}
```

可以看到，这里在第11行会先回调requestListener的onResourceReady()方法，只有当这个onResourceReady()方法返回false的时候，才会继续调用Target的onResourceReady()方法，这也就是listener()方法的实现原理。

另外一个onException()方法的实现机制也是一模一样的，代码同样是在GenericRequest类中，如下所示：

```kotlin
public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {
    ...

@Override
public void onException(Exception e) {
    status = Status.FAILED;
    if (requestListener == null || 
            !requestListener.onException(e, model, target, isFirstReadyResource())) {
        setErrorPlaceholder(e);
    }
}

...

}
```

可以看到，这里会在第9行回调requestListener的onException()方法，只有在onException()方法返回false的情况下才会继续调用setErrorPlaceholder()方法。也就是说，如果我们在onException()方法中返回了true，那么Glide请求中使用error(int resourceId)方法设置的异常占位图就失效了。

这样我们也就将listener()方法的全部实现原理都分析完了。

# Android图片加载框架最全解析（五），Glide强大的图片变换功能

如果你还没有阅读过前面四篇文章的话，那么可以点击后面的链接，依次向前阅读 Android图片加载框架最全解析（四），玩转Glide的回调与监听。

不过Glide的这个框架的功能实在是太强大了，它所能做的事情远远不止于目前我们所学的这些。因此，今天我们就再来学习一个新的功能模块，并且是一个非常重要的模块——Glide的图片变化功能。

一个问题
在正式开始学习Glide的图片变化功能之前，我们先来看一个问题，这个问题可能有不少人都在使用Glide的时候都遇到过，正好在本篇内容的主题之下我们顺带着将这个问题给解决了。

首先我们尝试使用Glide来加载一张图片，图片URL地址是：

https://www.baidu.com/img/bd_logo1.png
1
这是百度首页logo的一张图片，图片尺寸是540*258像素。

接下来我们编写一个非常简单的布局文件，如下所示：

<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Load Image"
        android:onClick="loadImage"
        />
    
    <ImageView
        android:id="@+id/image_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />
</LinearLayout>

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
布局文件中只有一个按钮和一个用于显示图片的ImageView。注意，ImageView的宽和高这里设置的都是wrap_content。

然后编写如下的代码来加载图片：

public class MainActivity extends AppCompatActivity {

    ImageView imageView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
    }
    
    public void loadImage(View view) {
        String url = "https://www.baidu.com/img/bd_logo1.png";
        Glide.with(this)
             .load(url)
             .into(imageView);
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
这些简单的代码对于现在的你而言应该都是小儿科了，相信我也不用再做什么解释。现在运行一下程序并点击加载图片按钮，效果如下图所示。


图片是正常加载出来了，不过大家有没有发现一个问题。百度这张logo图片的尺寸只有540*258像素，但是我的手机的分辨率却是1080*1920像素，而我们将ImageView的宽高设置的都是wrap_content，那么图片的宽度应该只有手机屏幕宽度的一半而已，但是这里却充满了全屏，这是为什么呢？

如果你之前也被这个问题困扰过，那么恭喜，本篇文章正是你所需要的。之所以会出现这个现象，就是因为Glide的图片变换功能所导致的。那么接下来我们会先分析如何解决这个问题，然后再深入学习Glide图片变化的更多功能。

稍微对Android有点了解的人应该都知道ImageView有scaleType这个属性，但是可能大多数人却不知道，如果在没有指定scaleType属性的情况下，ImageView默认的scaleType是什么？

这个问题如果直接问我，我也答不上来。不过动手才是检验真理的唯一标准，想知道答案，自己动手试一下就知道了。

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";
    
    ImageView imageView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
        Log.d(TAG, "imageView scaleType is " + imageView.getScaleType());
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
可以看到，我们在onCreate()方法中打印了ImageView默认的scaleType，然后重新运行一下程序，结果如下图所示：


由此我们可以得知，在没有明确指定的情况下，ImageView默认的scaleType是FIT_CENTER。

有了这个前提条件，我们就可以继续去分析Glide的源码了。当然，本文中的源码还是建在第二篇源码分析的基础之上，还没有看过这篇文章的朋友，建议先去阅读 Android图片加载框架最全解析（二），从源码的角度理解Glide的执行流程 。

回顾一下第二篇文章中我们分析过的into()方法，它是在GenericRequestBuilder类当中的，代码如下所示：

public Target<TranscodeType> into(ImageView view) {
    Util.assertMainThread();
    if (view == null) {
        throw new IllegalArgumentException("You must pass in a non null View");
    }
    if (!isTransformationSet && view.getScaleType() != null) {
        switch (view.getScaleType()) {
            case CENTER_CROP:
                applyCenterCrop();
                break;
            case FIT_CENTER:
            case FIT_START:
            case FIT_END:
                applyFitCenter();
                break;
            //$CASES-OMITTED$
            default:
                // Do nothing.
        }
    }
    return into(glide.buildImageViewTarget(view, transcodeClass));
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
还记得我们当初分析这段代码的时候，直接跳过前面的所有代码，直奔最后一行。因为那个时候我们的主要任务是分析Glide的主线执行流程，而不去仔细阅读它的细节，但是现在我们是时候应该阅读一下细节了。

可以看到，这里在第7行会进行一个switch判断，如果ImageView的scaleType是CENTER_CROP，则会去调用applyCenterCrop()方法，如果scaleType是FIT_CENTER、FIT_START或FIT_END，则会去调用applyFitCenter()方法。这里的applyCenterCrop()和applyFitCenter()方法其实就是向Glide的加载流程中添加了一个图片变换操作，具体的源码我们就不跟进去看了。

那么现在我们就基本清楚了，由于ImageView默认的scaleType是FIT_CENTER，因此会自动添加一个FitCenter的图片变换，而在这个图片变换过程中做了某些操作，导致图片充满了全屏。

那么我们该如何解决这个问题呢？最直白的一种办法就是看着源码来改。当ImageView的scaleType是CENTER_CROP、FIT_CENTER、FIT_START或FIT_END时不是会自动添加一个图片变换操作吗？那我们把scaleType改成其他值不就可以了。ImageView的scaleType可选值还有CENTER、CENTER_INSIDE、FIT_XY等。这当然是一种解决方案，不过只能说是一种比较笨的解决方案，因为我们为了解决这个问题而去改动了ImageView原有的scaleType，那如果你真的需要ImageView的scaleType为CENTER_CROP或FIT_CENTER时可能就傻眼了。

上面只是我们通过分析源码得到的一种解决方案，并不推荐大家使用。实际上，Glide给我们提供了专门的API来添加和取消图片变换，想要解决这个问题只需要使用如下代码即可：

Glide.with(this)
     .load(url)
     .dontTransform()
     .into(imageView);
1
2
3
4
可以看到，这里调用了一个dontTransform()方法，表示让Glide在加载图片的过程中不进行图片变换，这样刚才调用的applyCenterCrop()、applyFitCenter()就统统无效了。

现在我们重新运行一下代码，效果如下图所示：


这样图片就只会占据半个屏幕的宽度了，说明我们的代码奏效了。

但是使用dontTransform()方法存在着一个问题，就是调用这个方法之后，所有的图片变换操作就全部失效了，那如果我有一些图片变换操作是必须要执行的该怎么办呢？不用担心，总归是有办法的，这种情况下我们只需要借助override()方法强制将图片尺寸指定成原始大小就可以了，代码如下所示：

Glide.with(this)
     .load(url)
     .override(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL)
     .into(imageView);
1
2
3
4
通过override()方法将图片的宽和高都指定成Target.SIZE_ORIGINAL，问题同样被解决了。程序的最终运行结果和上图是完全一样的，我就不再重新截图了。

由此我们可以看出，之所以会出现这个问题，和Glide的图片变换功能是撇不开关系的。那么也是通过这个问题，我们对Glide的图片变换有了一个最基本的认识。接下来，就让我们正式开始进入本篇文章的正题吧。

图片变换的基本用法
顾名思义，图片变换的意思就是说，Glide从加载了原始图片到最终展示给用户之前，又进行了一些变换处理，从而能够实现一些更加丰富的图片效果，如图片圆角化、圆形化、模糊化等等。

添加图片变换的用法非常简单，我们只需要调用transform()方法，并将想要执行的图片变换操作作为参数传入transform()方法即可，如下所示：

Glide.with(this)
     .load(url)
     .transform(...)
     .into(imageView);
1
2
3
4
至于具体要进行什么样的图片变换操作，这个通常都是需要我们自己来写的。不过Glide已经内置了两种图片变换操作，我们可以直接拿来使用，一个是CenterCrop，一个是FitCenter。

但这两种内置的图片变换操作其实都不需要使用transform()方法，Glide为了方便我们使用直接提供了现成的API：

Glide.with(this)
     .load(url)
     .centerCrop()
     .into(imageView);

Glide.with(this)
     .load(url)
     .fitCenter()
     .into(imageView);
1
2
3
4
5
6
7
8
9
当然，centerCrop()和fitCenter()方法其实也只是对transform()方法进行了一层封装而已，它们背后的源码仍然还是借助transform()方法来实现的，如下所示：

public class DrawableRequestBuilder<ModelType>
        extends GenericRequestBuilder<ModelType, ImageVideoWrapper, GifBitmapWrapper, GlideDrawable>
        implements BitmapOptions, DrawableOptions {
    ...

    /**
     * Transform {@link GlideDrawable}s using {@link com.bumptech.glide.load.resource.bitmap.CenterCrop}.
     *
     * @see #fitCenter()
     * @see #transform(BitmapTransformation...)
     * @see #bitmapTransform(Transformation[])
     * @see #transform(Transformation[])
     *
     * @return This request builder.
     */
    @SuppressWarnings("unchecked")
    public DrawableRequestBuilder<ModelType> centerCrop() {
        return transform(glide.getDrawableCenterCrop());
    }
    
    /**
     * Transform {@link GlideDrawable}s using {@link com.bumptech.glide.load.resource.bitmap.FitCenter}.
     *
     * @see #centerCrop()
     * @see #transform(BitmapTransformation...)
     * @see #bitmapTransform(Transformation[])
     * @see #transform(Transformation[])
     *
     * @return This request builder.
     */
    @SuppressWarnings("unchecked")
    public DrawableRequestBuilder<ModelType> fitCenter() {
        return transform(glide.getDrawableFitCenter());
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
那么这两种内置的图片变换操作到底能实现什么样的效果呢？FitCenter的效果其实刚才我们已经见识过了，就是会将图片按照原始的长宽比充满全屏。那么CenterCrop又是什么样的效果呢？我们来动手试一下就知道了。

为了让效果更加明显，这里我就不使用百度首页的Logo图了，而是换成必应首页的一张美图。在不应用任何图片变换的情况下，使用Glide加载必应这张图片效果如下所示。


现在我们添加一个CenterCrop的图片变换操作，代码如下：

String url = "http://cn.bing.com/az/hprichbg/rb/AvalancheCreek_ROW11173354624_1920x1080.jpg";
Glide.with(this)
     .load(url)
     .centerCrop()
     .into(imageView);
1
2
3
4
5
重新运行一下程序并点击加载图片按钮，效果如下图所示。


可以看到，现在展示的图片是对原图的中心区域进行裁剪后得到的图片。

另外，centerCrop()方法还可以配合override()方法来实现更加丰富的效果，比如指定图片裁剪的比例：

String url = "http://cn.bing.com/az/hprichbg/rb/AvalancheCreek_ROW11173354624_1920x1080.jpg";
Glide.with(this)
     .load(url)
     .override(500, 500)
     .centerCrop()
     .into(imageView);
1
2
3
4
5
6
可以看到，这里我们将图片的尺寸设定为500*500像素，那么裁剪的比例也就变成1：1了，现在重新运行一下程序，效果如下图所示。


这样我们就把Glide内置的图片变换接口的用法都掌握了。不过不得不说，Glide内置的图片变换接口功能十分单一且有限，完全没有办法满足我们平时的开发需求。因此，掌握自定义图片变换功能就显得尤为重要了。

不过，在正式开始学习自定义图片变换功能之前，我们先来探究一下CenterCrop这种图片变换的源码，理解了它的源码我们再来进行自定义图片变换就能更加得心应手了。

源码分析
那么就话不多说，我们直接打开CenterCrop类来看一下它的源码吧，如下所示：

public class CenterCrop extends BitmapTransformation {

    public CenterCrop(Context context) {
        super(context);
    }
    
    public CenterCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }
    
    // Bitmap doesn't implement equals, so == and .equals are equivalent here.
    @SuppressWarnings("PMD.CompareObjectsWithEquals")
    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        final Bitmap toReuse = pool.get(outWidth, outHeight, toTransform.getConfig() != null
                ? toTransform.getConfig() : Bitmap.Config.ARGB_8888);
        Bitmap transformed = TransformationUtils.centerCrop(toReuse, toTransform, outWidth, outHeight);
        if (toReuse != null && toReuse != transformed && !pool.put(toReuse)) {
            toReuse.recycle();
        }
        return transformed;
    }
    
    @Override
    public String getId() {
        return "CenterCrop.com.bumptech.glide.load.resource.bitmap";
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
这段代码并不长，但是我还是要划下重点，这样大家看起来的时候会更加轻松。

首先，CenterCrop是继承自BitmapTransformation的，这个是重中之重，因为整个图片变换功能都是建立在这个继承结构基础上的。

接下来CenterCrop中最重要的就是transform()方法，其他的方法我们可以暂时忽略。transform()方法中有四个参数，每一个都很重要，我们来一一解读下。第一个参数pool，这个是Glide中的一个Bitmap缓存池，用于对Bitmap对象进行重用，否则每次图片变换都重新创建Bitmap对象将会非常消耗内存。第二个参数toTransform，这个是原始图片的Bitmap对象，我们就是要对它来进行图片变换。第三和第四个参数比较简单，分别代表图片变换后的宽度和高度，其实也就是override()方法中传入的宽和高的值了。

下面我们来看一下transform()方法的细节，首先第一行就从Bitmap缓存池中尝试获取一个可重用的Bitmap对象，然后把这个对象连同toTransform、outWidth、outHeight参数一起传入到了TransformationUtils.centerCrop()方法当中。那么我们就跟进去来看一下这个方法的源码，如下所示：

public final class TransformationUtils {
    ...

    public static Bitmap centerCrop(Bitmap recycled, Bitmap toCrop, int width, int height) {
        if (toCrop == null) {
            return null;
        } else if (toCrop.getWidth() == width && toCrop.getHeight() == height) {
            return toCrop;
        }
        // From ImageView/Bitmap.createScaledBitmap.
        final float scale;
        float dx = 0, dy = 0;
        Matrix m = new Matrix();
        if (toCrop.getWidth() * height > width * toCrop.getHeight()) {
            scale = (float) height / (float) toCrop.getHeight();
            dx = (width - toCrop.getWidth() * scale) * 0.5f;
        } else {
            scale = (float) width / (float) toCrop.getWidth();
            dy = (height - toCrop.getHeight() * scale) * 0.5f;
        }
        m.setScale(scale, scale);
        m.postTranslate((int) (dx + 0.5f), (int) (dy + 0.5f));
    
        final Bitmap result;
        if (recycled != null) {
            result = recycled;
        } else {
            result = Bitmap.createBitmap(width, height, getSafeConfig(toCrop));
        }
    
        // We don't add or remove alpha, so keep the alpha setting of the Bitmap we were given.
        TransformationUtils.setAlpha(toCrop, result);
    
        Canvas canvas = new Canvas(result);
        Paint paint = new Paint(PAINT_FLAGS);
        canvas.drawBitmap(toCrop, m, paint);
        return result;
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
这段代码就是整个图片变换功能的核心代码了。可以看到，第5-9行主要是先做了一些校验，如果原图为空，或者原图的尺寸和目标裁剪尺寸相同，那么就放弃裁剪。接下来第11-22行是通过数学计算来算出画布的缩放的比例以及偏移值。第24-29行是判断缓存池中取出的Bitmap对象是否为空，如果不为空就可以直接使用，如果为空则要创建一个新的Bitmap对象。第32行是将原图Bitmap对象的alpha值复制到裁剪Bitmap对象上面。最后第34-37行是裁剪Bitmap对象进行绘制，并将最终的结果进行返回。全部的逻辑就是这样，总体来说还是比较简单的，可能也就是数学计算那边需要稍微动下脑筋。

那么现在得到了裁剪后的Bitmap对象，我们再回到CenterCrop当中，你会看到，在最终返回这个Bitmap对象之前，还会尝试将复用的Bitmap对象重新放回到缓存池当中，以便下次继续使用。

好的，这样我们就将CenterCrop图片变换的工作原理完整地分析了一遍，FitCenter的源码也是基本类似的，这里就不再重复分析了。了解了这些内容之后，接下来我们就可以开始学习自定义图片变换功能了。

自定义图片变换
Glide给我们定制好了一个图片变换的框架，大致的流程是我们可以获取到原始的图片，然后对图片进行变换，再将变换完成后的图片返回给Glide，最终由Glide将图片显示出来。理论上，在对图片进行变换这个步骤中我们可以进行任何的操作，你想对图片怎么样都可以。包括圆角化、圆形化、黑白化、模糊化等等，甚至你将原图片完全替换成另外一张图都是可以的。

但是这里显然我不可能向大家演示所有图片变换的可能，图片变换的可能性也是无限的。因此这里我们就选择一种常用的图片变换效果来进行自定义吧——对图片进行圆形化变换。

图片圆形化的功能现在在手机应用中非常常见，比如手机QQ就会将用户的头像进行圆形化变换，从而使得界面变得更加好看。

自定义图片变换功能的实现逻辑比较固定，我们刚才看过CenterCrop的源码之后，相信你已经基本了解整个自定义的过程了。其实就是自定义一个类让它继承自BitmapTransformation ，然后重写transform()方法，并在这里去实现具体的图片变换逻辑就可以了。一个空的图片变换实现大概如下所示：

public class CircleCrop extends BitmapTransformation {

    public CircleCrop(Context context) {
        super(context);
    }
    
    public CircleCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }
    
    @Override
    public String getId() {
        return "com.example.glidetest.CircleCrop";
    }
    
    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return null;
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
这里有一点需要注意，就是getId()方法中要求返回一个唯一的字符串来作为id，以和其他的图片变换做区分。通常情况下，我们直接返回当前类的完整类名就可以了。

另外，这里我们选择继承BitmapTransformation还有一个限制，就是只能对静态图进行图片变换。当然，这已经足够覆盖日常95%以上的开发需求了。如果你有特殊的需求要对GIF图进行图片变换，那就得去自己实现Transformation接口才可以了。不过这个就非常复杂了，不在我们今天的讨论范围。

好了，那么我们继续实现对图片进行圆形化变换的功能，接下来只需要在transform()方法中去做具体的逻辑实现就可以了，代码如下所示：

public class CircleCrop extends BitmapTransformation {

    public CircleCrop(Context context) {
        super(context);
    }
    
    public CircleCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }
    
    @Override
    public String getId() {
        return "com.example.glidetest.CircleCrop";
    }
    
    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        int diameter = Math.min(toTransform.getWidth(), toTransform.getHeight());
    
        final Bitmap toReuse = pool.get(outWidth, outHeight, Bitmap.Config.ARGB_8888);
        final Bitmap result;
        if (toReuse != null) {
            result = toReuse;
        } else {
            result = Bitmap.createBitmap(diameter, diameter, Bitmap.Config.ARGB_8888);
        }
    
        int dx = (toTransform.getWidth() - diameter) / 2;
        int dy = (toTransform.getHeight() - diameter) / 2;
        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        BitmapShader shader = new BitmapShader(toTransform, BitmapShader.TileMode.CLAMP, 
                                            BitmapShader.TileMode.CLAMP);
        if (dx != 0 || dy != 0) {
            Matrix matrix = new Matrix();
            matrix.setTranslate(-dx, -dy);
            shader.setLocalMatrix(matrix);
        }
        paint.setShader(shader);
        paint.setAntiAlias(true);
        float radius = diameter / 2f;
        canvas.drawCircle(radius, radius, radius, paint);
    
        if (toReuse != null && !pool.put(toReuse)) {
            toReuse.recycle();
        }
        return result;
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
下面我来对transform()方法中的逻辑做下简单的解释。首先第18行先算出原图宽度和高度中较小的值，因为对图片进行圆形化变换肯定要以较小的那个值作为直径来进行裁剪。第20-26行则和刚才一样，从Bitmap缓存池中尝试获取一个Bitmap对象来进行重用，如果没有可重用的Bitmap对象的话就创建一个。第28-41行是具体进行圆形化变换的部分，这里算出了画布的偏移值，并且根据刚才得到的直径算出半径来进行画圆。最后，尝试将复用的Bitmap对象重新放回到缓存池当中，并将圆形化变换后的Bitmap对象进行返回。

这样，一个自定义图片变换的功能就写好了，那么现在我们就来尝试使用一下它吧。使用方法非常简单，刚才已经介绍过了，就是把这个自定义图片变换的实例传入到transform()方法中即可，如下所示：

Glide.with(this)
     .load(url)
     .transform(new CircleCrop(this))
     .into(imageView);
1
2
3
4
现在我们重新运行一下程序，效果如下图所示。


更多图片变换功能
虽说Glide的图片变换功能框架已经很强大了，使得我们可以轻松地自定义图片变换效果，但是如果每一种图片变换都要我们自己去写还是蛮吃力的。事实上，确实也没有必要完全靠自己去实现各种各样的图片变换效果，因为大多数的图片变换都是比较通用的，各个项目会用到的效果都差不多，我们每一个都自己去重新实现无异于重复造轮子。

也正是因此，网上出现了很多Glide的图片变换开源库，其中做的最出色的应该要数glide-transformations这个库了。它实现了很多通用的图片变换效果，如裁剪变换、颜色变换、模糊变换等等，使得我们可以非常轻松地进行各种各样的图片变换。

glide-transformations的项目主页地址是 https://github.com/wasabeef/glide-transformations 。

下面我们就来体验一下这个库的强大功能吧。首先需要将这个库引入到我们的项目当中，在app/build.gradle文件当中添加如下依赖：

dependencies {
    compile 'jp.wasabeef:glide-transformations:2.0.2'
}
1
2
3
现在如果我想对图片进行模糊化处理，那么就可以使用glide-transformations库中的BlurTransformation这个类，代码如下所示：

Glide.with(this)
     .load(url)
     .bitmapTransform(new BlurTransformation(this))
     .into(imageView);
1
2
3
4
注意这里我们调用的是bitmapTransform()方法而不是transform()方法，因为glide-transformations库都是专门针对静态图片变换来进行设计的。现在重新运行一下程度，效果如下图所示。


没错，我们就这样轻松地实现模糊化的效果了。

接下来我们再试一下图片黑白化的效果，使用的是GrayscaleTransformation这个类，代码如下所示：

Glide.with(this)
     .load(url)
     .bitmapTransform(new GrayscaleTransformation(this))
     .into(imageView);
1
2
3
4
现在重新运行一下程度，效果如下图所示。


而且我们还可以将多个图片变换效果组合在一起使用，比如同时执行模糊化和黑白化的变换：

Glide.with(this)
     .load(url)
     .bitmapTransform(new BlurTransformation(this), new GrayscaleTransformation(this))
     .into(imageView);
1
2
3
4
可以看到，同时执行多种图片变换的时候，只需要将它们都传入到bitmapTransform()方法中即可。现在重新运行一下程序，效果如下图所示。


当然，这些只是glide-transformations库的一小部分功能而已，更多的图片变换效果你可以到它的GitHub项目主页去学习，所有变换的用法都是这么简单哦。
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

原文链接：

# Android图片加载框架最全解析（六），探究Glide的自定义模块功能

不知不觉中，我们的Glide系列教程已经到了第六篇了，距离第一篇Glide的基本用法发布已经过去了半年的时间。在这半年中，我们通过用法讲解和源码分析配合学习的方式，将Glide的方方面面都研究了个遍，相信一直能看到这里的朋友现在已经是一位Glide高手了。

整个Glide系列预计总共会有八篇文章，现在也是逐步进入尾声了。不过，越是到后面，我们探究的内容也越是更加深入。那么今天，我们就来一起探究一下Glide中一个比较深入，但同时也是非常重要的一个功能——自定义模块。

自定义模块的基本用法
学到这里相信你已经知道，Glide的用法是非常非常简单的，大多数情况下，我们想要实现的图片加载效果只需要一行代码就能解决了。但是Glide过于简洁的API也造成了一个问题，就是如果我们想要更改Glide的某些默认配置项应该怎么操作呢？很难想象如何将更改Glide配置项的操作串联到一行经典的Glide图片加载语句中当中吧？没错，这个时候就需要用到自定义模块功能了。

自定义模块功能可以将更改Glide配置，替换Glide组件等操作独立出来，使得我们能轻松地对Glide的各种配置进行自定义，并且又和Glide的图片加载逻辑没有任何交集，这也是一种低耦合编程方式的体现。那么接下来我们就学习一下自定义模块的基本用法。

首先需要定义一个我们自己的模块类，并让它实现GlideModule接口，如下所示：

public class MyGlideModule implements GlideModule {
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
    }

    @Override
    public void registerComponents(Context context, Glide glide) {
    }
}
1
2
3
4
5
6
7
8
9
10
可以看到，在MyGlideModule类当中，我们重写了applyOptions()和registerComponents()方法，这两个方法分别就是用来更改Glide和配置以及替换Glide组件的。我们待会儿只需要在这两个方法中加入具体的逻辑，就能实现更改Glide配置或者替换Glide组件的功能了。

不过，目前Glide还无法识别我们自定义的MyGlideModule，如果想要让它生效，还得在AndroidManifest.xml文件当中加入如下配置才行：

<manifest>

    ...
    
    <application>
    
        <meta-data
            android:name="com.example.glidetest.MyGlideModule"
            android:value="GlideModule" />
    
        ...
    
    </application>
</manifest>  
1
2
3
4
5
6
7
8
9
10
11
12
13
14
在<application>标签中加入一个meta-data配置项，其中android:name指定成我们自定义的MyGlideModule的完整路径，android:value必须指定成GlideModule，这个是固定值。

这样的话，我们就将Glide自定义模块的功能完成了，是不是非常简单？现在Glide已经能够识别我们自定义的这个MyGlideModule了，但是在编写具体的功能之前，我们还是按照老规矩阅读一下源码，从源码的层面上来分析一下，Glide到底是如何识别出这个自定义的MyGlideModule的。

自定义模块的原理
这里我不会带着大家从Glide代码执行的第一步一行行重头去解析Glide的源码，而是只分析和自定义模块相关的部分。如果你想将Glide的源码通读一遍的话，可以去看本系列的第二篇文章 Android图片加载框架最全解析（二），从源码的角度理解Glide的执行流程 。

显然我们已经用惯了Glide.with(context).load(url).into(imageView)这样一行简洁的Glide图片加载语句，但是我们好像从来没有注意过Glide这个类本身的实例。然而事实上，Glide类确实是有创建实例的，只不过是在内部由Glide自动帮我们创建和管理了，对于开发者而言，大多数情况下是不用关心它的，只需要调用它的静态方法就可以了。

那么Glide的实例到底是在哪里创建的呢？我们来看下Glide类中的get()方法的源码，如下所示：

public class Glide {

    private static volatile Glide glide;
    
    ...
    
    public static Glide get(Context context) {
        if (glide == null) {
            synchronized (Glide.class) {
                if (glide == null) {
                    Context applicationContext = context.getApplicationContext();
                    List<GlideModule> modules = new ManifestParser(applicationContext).parse();
                    GlideBuilder builder = new GlideBuilder(applicationContext);
                    for (GlideModule module : modules) {
                        module.applyOptions(applicationContext, builder);
                    }
                    glide = builder.createGlide();
                    for (GlideModule module : modules) {
                        module.registerComponents(applicationContext, glide);
                    }
                }
            }
        }
        return glide;
    }
    
    ...
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
我们来仔细看一下上面这段代码。首先这里使用了一个单例模式来获取Glide对象的实例，可以看到，这是一个非常典型的双重锁模式。然后在第12行，调用ManifestParser的parse()方法去解析AndroidManifest.xml文件中的配置，实际上就是将AndroidManifest中所有值为GlideModule的meta-data配置读取出来，并将相应的自定义模块实例化。由于你可以自定义任意多个模块，因此这里我们将会得到一个GlideModule的List集合。

接下来在第13行创建了一个GlideBuilder对象，并通过一个循环调用了每一个GlideModule的applyOptions()方法，同时也把GlideBuilder对象作为参数传入到这个方法中。而applyOptions()方法就是我们可以加入自己的逻辑的地方了，虽然目前为止我们还没有编写任何逻辑。

再往下的一步就非常关键了，这里调用了GlideBuilder的createGlide()方法，并返回了一个Glide对象。也就是说，Glide对象的实例就是在这里创建的了，那么我们跟到这个方法当中瞧一瞧：

public class GlideBuilder {
    private final Context context;

    private Engine engine;
    private BitmapPool bitmapPool;
    private MemoryCache memoryCache;
    private ExecutorService sourceService;
    private ExecutorService diskCacheService;
    private DecodeFormat decodeFormat;
    private DiskCache.Factory diskCacheFactory;
    
    ...
    
    Glide createGlide() {
        if (sourceService == null) {
            final int cores = Math.max(1, Runtime.getRuntime().availableProcessors());
            sourceService = new FifoPriorityThreadPoolExecutor(cores);
        }
        if (diskCacheService == null) {
            diskCacheService = new FifoPriorityThreadPoolExecutor(1);
        }
        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        if (bitmapPool == null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
                int size = calculator.getBitmapPoolSize();
                bitmapPool = new LruBitmapPool(size);
            } else {
                bitmapPool = new BitmapPoolAdapter();
            }
        }
        if (memoryCache == null) {
            memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
        }
        if (diskCacheFactory == null) {
            diskCacheFactory = new InternalCacheDiskCacheFactory(context);
        }
        if (engine == null) {
            engine = new Engine(memoryCache, diskCacheFactory, diskCacheService, sourceService);
        }
        if (decodeFormat == null) {
            decodeFormat = DecodeFormat.DEFAULT;
        }
        return new Glide(engine, memoryCache, bitmapPool, context, decodeFormat);
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
这个方法中会创建BitmapPool、MemoryCache、DiskCache、DecodeFormat等对象的实例，并在最后一行创建一个Glide对象的实例，然后将前面创建的这些实例传入到Glide对象当中，以供后续的图片加载操作使用。

但是大家有没有注意到一个细节，createGlide()方法中创建任何对象的时候都做了一个空检查，只有在对象为空的时候才会去创建它的实例。也就是说，如果我们可以在applyOptions()方法中提前就给这些对象初始化并赋值，那么在createGlide()方法中就不会再去重新创建它们的实例了，从而也就实现了更改Glide配置的功能。关于这个功能我们待会儿会进行具体的演示。

现在继续回到Glide的get()方法中，得到了Glide对象的实例之后，接下来又通过一个循环调用了每一个GlideModule的registerComponents()方法，在这里我们可以加入替换Glide的组件的逻辑。

好了，这就是Glide自定义模块的全部工作原理。了解了它的工作原理之后，接下来所有的问题就集中在我们到底如何在applyOptions()和registerComponents()这两个方法中加入具体的逻辑了，下面我们马上就来学习一下。

更改Glide配置
刚才在分析自定义模式工作原理的时候其实就已经提到了，如果想要更改Glide的默认配置，其实只需要在applyOptions()方法中提前将Glide的配置项进行初始化就可以了。那么Glide一共有哪些配置项呢？这里我给大家做了一个列举：

setMemoryCache()
用于配置Glide的内存缓存策略，默认配置是LruResourceCache。

setBitmapPool()
用于配置Glide的Bitmap缓存池，默认配置是LruBitmapPool。

setDiskCache()
用于配置Glide的硬盘缓存策略，默认配置是InternalCacheDiskCacheFactory。

setDiskCacheService()
用于配置Glide读取缓存中图片的异步执行器，默认配置是FifoPriorityThreadPoolExecutor，也就是先入先出原则。

setResizeService()
用于配置Glide读取非缓存中图片的异步执行器，默认配置也是FifoPriorityThreadPoolExecutor。

setDecodeFormat()
用于配置Glide加载图片的解码模式，默认配置是RGB_565。

其实Glide的这些默认配置都非常科学且合理，使用的缓存算法也都是效率极高的，因此在绝大多数情况下我们并不需要去修改这些默认配置，这也是Glide用法能如此简洁的一个原因。

但是Glide科学的默认配置并不影响我们去学习自定义Glide模块的功能，因此总有某些情况下，默认的配置可能将无法满足你，这个时候就需要我们自己动手来修改默认配置了。

下面就通过具体的实例来看一下吧。刚才说到，Glide默认的硬盘缓存策略使用的是InternalCacheDiskCacheFactory，这种缓存会将所有Glide加载的图片都存储到当前应用的私有目录下。这是一种非常安全的做法，但同时这种做法也造成了一些不便，因为私有目录下即使是开发者自己也是无法查看的，如果我想要去验证一下图片到底有没有成功缓存下来，这就有点不太好办了。

这种情况下，就非常适合使用自定义模块来更改Glide的默认配置。我们完全可以自己去实现DiskCache.Factory接口来自定义一个硬盘缓存策略，不过却大大没有必要这么做，因为Glide本身就内置了一个ExternalCacheDiskCacheFactory，可以允许将加载的图片都缓存到SD卡。

那么接下来，我们就尝试使用这个ExternalCacheDiskCacheFactory来替换默认的InternalCacheDiskCacheFactory，从而将所有Glide加载的图片都缓存到SD卡上。

由于在前面我们已经创建好了一个自定义模块MyGlideModule，那么现在就可以直接在这里编写逻辑了，代码如下所示：

public class MyGlideModule implements GlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        builder.setDiskCache(new ExternalCacheDiskCacheFactory(context));
    }
    
    @Override
    public void registerComponents(Context context, Glide glide) {
    
    }

}
1
2
3
4
5
6
7
8
9
10
11
12
13
没错，就是这么简单，现在所有Glide加载的图片都会缓存到SD卡上了。

另外，InternalCacheDiskCacheFactory和ExternalCacheDiskCacheFactory的默认硬盘缓存大小都是250M。也就是说，如果你的应用缓存的图片总大小超出了250M，那么Glide就会按照DiskLruCache算法的原则来清理缓存的图片。

当然，我们是可以对这个默认的缓存大小进行修改的，而且修改方式非常简单，如下所示：

public class MyGlideModule implements GlideModule {

    public static final int DISK_CACHE_SIZE = 500 * 1024 * 1024;
    
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, DISK_CACHE_SIZE));
    }
    
    @Override
    public void registerComponents(Context context, Glide glide) {
    
    }

}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
只需要向ExternalCacheDiskCacheFactory或者InternalCacheDiskCacheFactory再传入一个参数就可以了，现在我们就将Glide硬盘缓存的大小调整成了500M。

好了，更改Glide配置的功能就是这么简单，那么接下来我们就来验证一下更改的配置到底有没有生效吧。

这里还是使用最基本的Glide加载语句来去加载一张网络图片：

String url = "http://guolin.tech/book.png";
Glide.with(this)
     .load(url)
     .into(imageView);
1
2
3
4
运行一下程序，效果如下图所示：


OK，现在图片已经加载出现了，那么我们去找一找它的缓存吧。

ExternalCacheDiskCacheFactory的默认缓存路径是在sdcard/Android/包名/cache/image_manager_disk_cache目录当中，我们使用文件浏览器进入到这个目录，结果如下图所示。


可以看到，这里有两个文件，其中journal文件是DiskLruCache算法的日志文件，这个文件必不可少，且只会有一个。想了解更多关于DiskLruCache算法的朋友，可以去阅读我的这篇博客 Android DiskLruCache完全解析，硬盘缓存的最佳方案 。

而另外一个文件就是那张缓存的图片了，它的文件名虽然看上去很奇怪，但是我们只需要把这个文件的后缀改成.png，然后用图片浏览器打开，结果就一目了然了，如下图所示。


由此证明，我们已经成功将Glide的硬盘缓存路径修改到SD卡上了。

另外这里再提一点，我们都知道Glide和Picasso的用法是非常相似的，但是有一点差别却很大。Glide加载图片的默认格式是RGB_565，而Picasso加载图片的默认格式是ARGB_8888。ARGB_8888格式的图片效果会更加细腻，但是内存开销会比较大。而RGB_565格式的图片则更加节省内存，但是图片效果上会差一些。

Glide和Picasso各自采取的默认图片格式谈不上熟优熟劣，只能说各自的取舍不一样。但是如果你希望Glide也能使用ARGB_8888的图片格式，这当然也是可以的。我们只需要在MyGlideModule中更改一下默认配置即可，如下所示：

public class MyGlideModule implements GlideModule {

    public static final int DISK_CACHE_SIZE = 500 * 1024 * 1024;
    
    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, DISK_CACHE_SIZE));
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
    }
    
    @Override
    public void registerComponents(Context context, Glide glide) {
    
    }

}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
通过这样配置之后，使用Glide加载的所有图片都将会使用ARGB_8888的格式，虽然图片质量变好了，但同时内存开销也会明显增大，所以你要做好心理准备哦。

好了，关于更改Glide配置的内容就介绍这么多，接下来就让我们进入到下一个非常重要的主题，替换Glide组件。

替换Glide组件
替换Glide组件功能需要在自定义模块的registerComponents()方法中加入具体的替换逻辑。相比于更改Glide配置，替换Glide组件这个功能的难度就明显大了不少。Glide中的组件非常繁多，也非常复杂，但其实大多数情况下并不需要我们去做什么替换。不过，有一个组件却有着比较大的替换需求，那就是Glide的HTTP通讯组件。

默认情况下，Glide使用的是基于原生HttpURLConnection进行订制的HTTP通讯组件，但是现在大多数的Android开发者都更喜欢使用OkHttp，因此将Glide中的HTTP通讯组件修改成OkHttp的这个需求比较常见，那么今天我们也会以这个功能来作为例子进行讲解。

首先来看一下Glide中目前有哪些组件吧，在Glide类的构造方法当中，如下所示：

public class Glide {

    Glide(Engine engine, MemoryCache memoryCache, BitmapPool bitmapPool, Context context, DecodeFormat decodeFormat) {
        ...
    
        register(File.class, ParcelFileDescriptor.class, new FileDescriptorFileLoader.Factory());
        register(File.class, InputStream.class, new StreamFileLoader.Factory());
        register(int.class, ParcelFileDescriptor.class, new FileDescriptorResourceLoader.Factory());
        register(int.class, InputStream.class, new StreamResourceLoader.Factory());
        register(Integer.class, ParcelFileDescriptor.class, new FileDescriptorResourceLoader.Factory());
        register(Integer.class, InputStream.class, new StreamResourceLoader.Factory());
        register(String.class, ParcelFileDescriptor.class, new FileDescriptorStringLoader.Factory());
        register(String.class, InputStream.class, new StreamStringLoader.Factory());
        register(Uri.class, ParcelFileDescriptor.class, new FileDescriptorUriLoader.Factory());
        register(Uri.class, InputStream.class, new StreamUriLoader.Factory());
        register(URL.class, InputStream.class, new StreamUrlLoader.Factory());
        register(GlideUrl.class, InputStream.class, new HttpUrlGlideUrlLoader.Factory());
        register(byte[].class, InputStream.class, new StreamByteArrayLoader.Factory());
    
        ...
    }

}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
可以看到，这里都是以调用register()方法的方式来注册一个组件，register()方法中传入的参数表示Glide支持使用哪种参数类型来加载图片，以及如何去处理这种类型的图片加载。举个例子：

register(GlideUrl.class, InputStream.class, new HttpUrlGlideUrlLoader.Factory());
1
这句代码就表示，我们可以使用Glide.with(context).load(new GlideUrl("url...")).into(imageView)的方式来加载图片，而HttpUrlGlideUrlLoader.Factory则是要负责处理具体的网络通讯逻辑。如果我们想要将Glide的HTTP通讯组件替换成OkHttp的话，那么只需要在自定义模块当中重新注册一个GlideUrl类型的组件就行了。

说到这里有的朋友可能会疑问了，我们平时使用Glide加载图片时，大多数情况下都是直接将图片的URL字符串传入到load()方法当中的，很少会将它封装成GlideUrl对象之后再传入到load()方法当中，那为什么只需要重新注册一个GlideUrl类型的组件，而不需要去重新注册一个String类型的组件呢？其实道理很简单，因为load(String)方法只是Glide给我们提供一种简易的API封装而已，它的底层仍然还是调用的GlideUrl组件，因此我们在替换组件的时候只需要直接替换最底层的，这样就一步到位了。

那么接下来我们就开始学习到底如何将Glide的HTTP通讯组件替换成OkHttp。

首先第一步，不用多说，肯定是要先将OkHttp的库引入到当前项目中，如下所示：

dependencies {
    compile 'com.squareup.okhttp3:okhttp:3.9.0'
}
1
2
3
然后接下来该怎么做呢？我们只要依葫芦画瓢就可以了。刚才不是说Glide的网络通讯逻辑是由HttpUrlGlideUrlLoader.Factory来负责的吗，那么我们就来看一下它的源码：

public class HttpUrlGlideUrlLoader implements ModelLoader<GlideUrl, InputStream> {

    private final ModelCache<GlideUrl, GlideUrl> modelCache;
    
    public static class Factory implements ModelLoaderFactory<GlideUrl, InputStream> {
        private final ModelCache<GlideUrl, GlideUrl> modelCache = new ModelCache<GlideUrl, GlideUrl>(500);
    
        @Override
        public ModelLoader<GlideUrl, InputStream> build(Context context, GenericLoaderFactory factories) {
            return new HttpUrlGlideUrlLoader(modelCache);
        }
    
        @Override
        public void teardown() {
        }
    }
    
    public HttpUrlGlideUrlLoader() {
        this(null);
    }
    
    public HttpUrlGlideUrlLoader(ModelCache<GlideUrl, GlideUrl> modelCache) {
        this.modelCache = modelCache;
    }
    
    @Override
    public DataFetcher<InputStream> getResourceFetcher(GlideUrl model, int width, int height) {
        GlideUrl url = model;
        if (modelCache != null) {
            url = modelCache.get(model, 0, 0);
            if (url == null) {
                modelCache.put(model, 0, 0, model);
                url = model;
            }
        }
        return new HttpUrlFetcher(url);
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
可以看到，HttpUrlGlideUrlLoader.Factory是一个内部类，外层的HttpUrlGlideUrlLoader类实现了ModelLoader<GlideUrl, InputStream>这个接口，并重写了getResourceFetcher()方法。而在getResourceFetcher()方法中，又创建了一个HttpUrlFetcher的实例，在这里才是真正处理具体网络通讯逻辑的地方，代码如下所示：

public class HttpUrlFetcher implements DataFetcher<InputStream> {
    private static final String TAG = "HttpUrlFetcher";
    private static final int MAXIMUM_REDIRECTS = 5;
    private static final HttpUrlConnectionFactory DEFAULT_CONNECTION_FACTORY = new DefaultHttpUrlConnectionFactory();

    private final GlideUrl glideUrl;
    private final HttpUrlConnectionFactory connectionFactory;
    
    private HttpURLConnection urlConnection;
    private InputStream stream;
    private volatile boolean isCancelled;
    
    public HttpUrlFetcher(GlideUrl glideUrl) {
        this(glideUrl, DEFAULT_CONNECTION_FACTORY);
    }
    
    HttpUrlFetcher(GlideUrl glideUrl, HttpUrlConnectionFactory connectionFactory) {
        this.glideUrl = glideUrl;
        this.connectionFactory = connectionFactory;
    }
    
    @Override
    public InputStream loadData(Priority priority) throws Exception {
        return loadDataWithRedirects(glideUrl.toURL(), 0 , null , glideUrl.getHeaders());
    }
    
    private InputStream loadDataWithRedirects(URL url, int redirects, URL lastUrl, Map<String, String> headers)
            throws IOException {
        if (redirects >= MAXIMUM_REDIRECTS) {
            throw new IOException("Too many (> " + MAXIMUM_REDIRECTS + ") redirects!");
        } else {
            try {
                if (lastUrl != null && url.toURI().equals(lastUrl.toURI())) {
                    throw new IOException("In re-direct loop");
                }
            } catch (URISyntaxException e) {
            }
        }
        urlConnection = connectionFactory.build(url);
        for (Map.Entry<String, String> headerEntry : headers.entrySet()) {
          urlConnection.addRequestProperty(headerEntry.getKey(), headerEntry.getValue());
        }
        urlConnection.setConnectTimeout(2500);
        urlConnection.setReadTimeout(2500);
        urlConnection.setUseCaches(false);
        urlConnection.connect();
        if (isCancelled) {
            return null;
        }
        final int statusCode = urlConnection.getResponseCode();
        if (statusCode / 100 == 2) {
            return getStreamForSuccessfulRequest(urlConnection);
        } else if (statusCode / 100 == 3) {
            String redirectUrlString = urlConnection.getHeaderField("Location");
            if (TextUtils.isEmpty(redirectUrlString)) {
                throw new IOException("Received empty or null redirect url");
            }
            URL redirectUrl = new URL(url, redirectUrlString);
            return loadDataWithRedirects(redirectUrl, redirects + 1, url, headers);
        } else {
            if (statusCode == -1) {
                throw new IOException("Unable to retrieve response code from HttpUrlConnection.");
            }
            throw new IOException("Request failed " + statusCode + ": " + urlConnection.getResponseMessage());
        }
    }
    
    private InputStream getStreamForSuccessfulRequest(HttpURLConnection urlConnection)
            throws IOException {
        if (TextUtils.isEmpty(urlConnection.getContentEncoding())) {
            int contentLength = urlConnection.getContentLength();
            stream = ContentLengthInputStream.obtain(urlConnection.getInputStream(), contentLength);
        } else {
            stream = urlConnection.getInputStream();
        }
        return stream;
    }
    
    @Override
    public void cleanup() {
        if (stream != null) {
            try {
                stream.close();
            } catch (IOException e) {
            }
        }
        if (urlConnection != null) {
            urlConnection.disconnect();
        }
    }
    
    @Override
    public String getId() {
        return glideUrl.getCacheKey();
    }
    
    @Override
    public void cancel() {
        isCancelled = true;
    }
    
    interface HttpUrlConnectionFactory {
        HttpURLConnection build(URL url) throws IOException;
    }
    
    private static class DefaultHttpUrlConnectionFactory implements HttpUrlConnectionFactory {
        @Override
        public HttpURLConnection build(URL url) throws IOException {
            return (HttpURLConnection) url.openConnection();
        }
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
上面这段代码看上去应该不费力吧？其实就是一些HttpURLConnection的用法而已。那么我们只需要仿照着HttpUrlFetcher的代码来写，并且把HTTP的通讯组件替换成OkHttp就可以了。

现在新建一个OkHttpFetcher类，并且同样实现DataFetcher<InputStream>接口，代码如下所示：

public class OkHttpFetcher implements DataFetcher<InputStream> {

    private final OkHttpClient client;
    private final GlideUrl url;
    private InputStream stream;
    private ResponseBody responseBody;
    private volatile boolean isCancelled;
    
    public OkHttpFetcher(OkHttpClient client, GlideUrl url) {
        this.client = client;
        this.url = url;
    }
    
    @Override
    public InputStream loadData(Priority priority) throws Exception {
        Request.Builder requestBuilder = new Request.Builder()
                .url(url.toStringUrl());
        for (Map.Entry<String, String> headerEntry : url.getHeaders().entrySet()) {
            String key = headerEntry.getKey();
            requestBuilder.addHeader(key, headerEntry.getValue());
        }
        requestBuilder.addHeader("httplib", "OkHttp");
        Request request = requestBuilder.build();
        if (isCancelled) {
            return null;
        }
        Response response = client.newCall(request).execute();
        responseBody = response.body();
        if (!response.isSuccessful() || responseBody == null) {
            throw new IOException("Request failed with code: " + response.code());
        }
        stream = ContentLengthInputStream.obtain(responseBody.byteStream(),
                responseBody.contentLength());
        return stream;
    }
    
    @Override
    public void cleanup() {
        try {
            if (stream != null) {
                stream.close();
            }
            if (responseBody != null) {
                responseBody.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public String getId() {
        return url.getCacheKey();
    }
    
    @Override
    public void cancel() {
        isCancelled = true;
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
上面这段代码完全就是我照着HttpUrlFetcher依葫芦画瓢写出来的，用的也都是一些OkHttp的基本用法，相信不需要再做什么解释了吧。可以看到，使用OkHttp来编写网络通讯的代码要比使用HttpURLConnection简单很多，代码行数也少了很多。注意在第22行，我添加了一个httplib: OkHttp的请求头，这个是待会儿我们用来进行测试验证的，大家实际项目中的代码无须添加这个请求头。

那么我们就继续发挥依葫芦画瓢的精神，仿照着HttpUrlGlideUrlLoader再写一个OkHttpGlideUrlLoader吧。新建一个OkHttpGlideUrlLoader类，并且实现ModelLoader<GlideUrl, InputStream>接口，代码如下所示：

public class OkHttpGlideUrlLoader implements ModelLoader<GlideUrl, InputStream> {

    private OkHttpClient okHttpClient;
    
    public static class Factory implements ModelLoaderFactory<GlideUrl, InputStream> {
    
        private OkHttpClient client;
    
        public Factory() {
        }
    
        public Factory(OkHttpClient client) {
            this.client = client;
        }
    
        private synchronized OkHttpClient getOkHttpClient() {
            if (client == null) {
                client = new OkHttpClient();
            }
            return client;
        }
    
        @Override
        public ModelLoader<GlideUrl, InputStream> build(Context context, GenericLoaderFactory factories) {
            return new OkHttpGlideUrlLoader(getOkHttpClient());
        }
    
        @Override
        public void teardown() {
        }
    }
    
    public OkHttpGlideUrlLoader(OkHttpClient client) {
        this.okHttpClient = client;
    }
    
    @Override
    public DataFetcher<InputStream> getResourceFetcher(GlideUrl model, int width, int height) {
        return new OkHttpFetcher(okHttpClient, model);
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
注意这里的Factory我提供了两个构造方法，一个是不带任何参数的，一个是带OkHttpClient参数的。如果对OkHttp不需要进行任何自定义的配置，那么就调用无参的Factory构造函数即可，这样会在内部自动创建一个OkHttpClient实例。但如果你需要想添加拦截器，或者修改OkHttp的默认超时等等配置，那么就自己创建一个OkHttpClient的实例，然后传入到Factory的构造方法当中就行了。

好了，现在就只差最后一步，将我们刚刚创建的OkHttpGlideUrlLoader和OkHttpFetcher注册到Glide当中，将原来的HTTP通讯组件给替换掉，如下所示：

public class MyGlideModule implements GlideModule {

    ...
    
    @Override
    public void registerComponents(Context context, Glide glide) {
        glide.register(GlideUrl.class, InputStream.class, new OkHttpGlideUrlLoader.Factory());
    }

}
1
2
3
4
5
6
7
8
9
10
可以看到，这里也是调用了Glide的register()方法来注册组件的。register()方法中使用的Map类型来存储已注册的组件，因此我们这里重新注册了一遍GlideUrl.class类型的组件，就把原来的组件给替换掉了。

理论上来说，现在我们已经成功将Glide的HTTP通讯组件替换成OkHttp了，现在唯一的问题就是我们该如何去验证一下到底有没有替换成功呢？

验证的方式我倒是想了很多种，比如添加OkHttp拦截器，或者自己架设一个测试用的服务器都是可以的。不过为了让大家最直接地看到验证结果，这里我准备使用Fiddler这个抓包工具来进行验证。这个工具的用法非常简单，但是限于篇幅我就不在本篇文章中介绍这个工具的用法了，还没用过这个工具的朋友们可以通过 这篇文章 了解一下。

在开始验证之前，我们还得要再修改一下Glide加载图片的代码才行，如下所示：

String url = "http://guolin.tech/book.png";
Glide.with(this)
     .load(url)
     .skipMemoryCache(true)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
1
2
3
4
5
6
这里我把Glide的内存缓存和硬盘缓存都禁用掉了，不然的话，Glide可能会直接读取刚才缓存的图片，而不会再重新发起网终请求。

好的，现在我们重新使用Glide加载一下图片，然后观察Fiddler中的抓包情况，如下图所示。


可以看到，在HTTP请求头中确实有我们刚才自己添加的httplib: OkHttp。也就说明，Glide的HTTP通讯组件的确被替换成功了。

更简单的组件替换
上述方法是我们纯手工地将Glide的HTTP通讯组件进行了替换，如果你不想这么麻烦也是可以的，Glide官方给我们提供了非常简便的HTTP组件替换方式。并且除了支持OkHttp3之外，还支持OkHttp2和Volley。

我们只需要在gradle当中添加几行库的配置就行了。比如使用OkHttp3来作为HTTP通讯组件的配置如下：

dependencies {
    compile 'com.squareup.okhttp3:okhttp:3.9.0'
    compile 'com.github.bumptech.glide:okhttp3-integration:1.5.0@aar'
}
1
2
3
4
使用OkHttp2来作为HTTP通讯组件的配置如下：

dependencies {
    compile 'com.github.bumptech.glide:okhttp-integration:1.5.0@aar'
    compile 'com.squareup.okhttp:okhttp:2.7.5'
}
1
2
3
4
使用Volley来作为HTTP通讯组件的配置如下：

dependencies {
    compile 'com.github.bumptech.glide:volley-integration:1.5.0@aar'  
    compile 'com.mcxiaoke.volley:library:1.0.19'  
}
1
2
3
4
当然了，这些库背后的工作原理和我们刚才自己手动实现替换HTTP组件的原理是一模一样的。而学会了手动替换组件的原理我们就能更加轻松地扩展更多丰富的功能，因此掌握这一技能还是非常重要的。
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

原文链接：

# Android图片加载框架最全解析（七），实现带进度的Glide图片加载功能

我们的Glide系列文章终于要进入收尾篇了。从我开始写这个系列的第一篇文章时，我就知道这会是一个很长的系列，只是没有想到竟然会写这么久。

在前面的六篇文章中，我们对Glide的方方面面都进行了学习，包括基本用法、源码解析、缓存机制、回调与监听、图片变换以及自定义模块。而今天，我们就要综合利用之前所学到的知识，来对Glide进行一个比较大的功能扩展，希望大家都已经好好阅读过了前面的六篇文章，并且有了不错的理解。

扩展目标
首先来确立一下功能扩展的目标。虽说Glide本身就已经十分强大了，但是有一个功能却长期以来都不支持，那就是监听下载进度功能。

我们都知道，使用Glide来加载一张网络上的图片是非常简单的，但是让人头疼的是，我们却无从得知当前图片的下载进度。如果这张图片很小的话，那么问题也不大，反正很快就会被加载出来。但如果这是一张比较大的GIF图，用户耐心等了很久结果图片还没显示出来，这个时候你就会觉得下载进度功能是十分有必要的了。

好的，那么我们今天的目标就是对Glide进行功能扩展，使其支持监听图片下载进度的功能。

开始
今天这篇文章我会带着大家从零去创建一个新的项目，一步步地进行实现，最终完成一个带进度的Glide图片加载的Demo。当然，在本篇文章的最后我会提供这个Demo的完整源码，但是这里我仍然希望大家能用心跟着我一步步来编写。

那么我们现在就开始吧，首先创建一个新项目，就叫做GlideProgressTest吧。

项目创建完成后的第一件事就是要将必要的依赖库引入到当前的项目当中，目前我们必须要依赖的两个库就是Glide和OkHttp。在app/build.gradle文件当中添加如下配置：

dependencies { 
    compile 'com.github.bumptech.glide:glide:3.7.0' 
    compile 'com.squareup.okhttp3:okhttp:3.9.0' 
}
1
2
3
4
另外，由于Glide和OkHttp都需要用到网络功能，因此我们还得在AndroidManifest.xml中声明一下网络权限才行：

<uses-permission android:name="android.permission.INTERNET" />
1
好了，这样准备工作就完成了。

替换通讯组件
通过第二篇文章的源码分析，我们知道了Glide内部HTTP通讯组件的底层实现是基于HttpUrlConnection来进行定制的。但是HttpUrlConnection的可扩展性比较有限，我们在它的基础之上无法实现监听下载进度的功能，因此今天的第一个大动作就是要将Glide中的HTTP通讯组件替换成OkHttp。

关于HTTP通讯组件的替换原理和替换方式，我在第六篇文章当中都介绍得比较清楚了，这里就不再赘述。下面我们就来开始快速地替换一下。

新建一个OkHttpFetcher类，并且实现DataFetcher接口，代码如下所示：

public class OkHttpFetcher implements DataFetcher<InputStream> { 

    private final OkHttpClient client; 
    private final GlideUrl url; 
    private InputStream stream; 
    private ResponseBody responseBody; 
    private volatile boolean isCancelled; 
    
    public OkHttpFetcher(OkHttpClient client, GlideUrl url) { 
        this.client = client; 
        this.url = url; 
    } 
    
    @Override 
    public InputStream loadData(Priority priority) throws Exception { 
        Request.Builder requestBuilder = new Request.Builder() 
                .url(url.toStringUrl()); 
        for (Map.Entry<String, String> headerEntry : url.getHeaders().entrySet()) {
            String key = headerEntry.getKey(); 
            requestBuilder.addHeader(key, headerEntry.getValue()); 
        } 
        Request request = requestBuilder.build(); 
        if (isCancelled) { 
            return null; 
        } 
        Response response = client.newCall(request).execute(); 
        responseBody = response.body(); 
        if (!response.isSuccessful() || responseBody == null) { 
            throw new IOException("Request failed with code: " + response.code());
        } 
        stream = ContentLengthInputStream.obtain(responseBody.byteStream(), 
                responseBody.contentLength()); 
        return stream; 
    } 
    
    @Override 
    public void cleanup() { 
        try { 
            if (stream != null) { 
                stream.close(); 
            } 
            if (responseBody != null) { 
                responseBody.close(); 
            } 
        } catch (IOException e) { 
            e.printStackTrace(); 
        } 
    } 
    
    @Override 
    public String getId() { 
        return url.getCacheKey(); 
    } 
    
    @Override 
    public void cancel() { 
        isCancelled = true; 
    } 
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
然后新建一个OkHttpGlideUrlLoader类，并且实现ModelLoader

public class OkHttpGlideUrlLoader implements ModelLoader<GlideUrl, InputStream> { 

    private OkHttpClient okHttpClient; 
    
    public static class Factory implements ModelLoaderFactory<GlideUrl, InputStream> { 
    
        private OkHttpClient client; 
    
        public Factory() { 
        } 
    
        public Factory(OkHttpClient client) { 
            this.client = client; 
        } 
    
        private synchronized OkHttpClient getOkHttpClient() { 
            if (client == null) { 
                client = new OkHttpClient(); 
            } 
            return client; 
        } 
    
        @Override 
        public ModelLoader<GlideUrl, InputStream> build(Context context, GenericLoaderFactory factories) {
            return new OkHttpGlideUrlLoader(getOkHttpClient()); 
        } 
    
        @Override 
        public void teardown() { 
        } 
    } 
    
    public OkHttpGlideUrlLoader(OkHttpClient client) { 
        this.okHttpClient = client; 
    } 
    
    @Override 
    public DataFetcher<InputStream> getResourceFetcher(GlideUrl model, int width, int height) { 
        return new OkHttpFetcher(okHttpClient, model); 
    } 
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
接下来，新建一个MyGlideModule类并实现GlideModule接口，然后在registerComponents()方法中将我们刚刚创建的OkHttpGlideUrlLoader和OkHttpFetcher注册到Glide当中，将原来的HTTP通讯组件给替换掉，如下所示：

public class MyGlideModule implements GlideModule { 
    @Override 
    public void applyOptions(Context context, GlideBuilder builder) { 
    } 

    @Override 
    public void registerComponents(Context context, Glide glide) { 
        glide.register(GlideUrl.class, InputStream.class, new OkHttpGlideUrlLoader.Factory());
    } 
}
1
2
3
4
5
6
7
8
9
10
最后，为了让Glide能够识别我们自定义的MyGlideModule，还得在AndroidManifest.xml文件当中加入如下配置才行：

<manifest> 
    ... 
    <application> 
        <meta-data 
            android:name="com.example.glideprogresstest.MyGlideModule" 
            android:value="GlideModule" /> 
        ... 
    </application> 
</manifest>
1
2
3
4
5
6
7
8
9
OK，这样我们就把Glide中的HTTP通讯组件成功替换成OkHttp了。

实现下载进度监听
那么，将HTTP通讯组件替换成OkHttp之后，我们又该如何去实现监听下载进度的功能呢？这就要依靠OkHttp强大的拦截器机制了。

我们只要向OkHttp中添加一个自定义的拦截器，就可以在拦截器中捕获到整个HTTP的通讯过程，然后加入一些自己的逻辑来计算下载进度，这样就可以实现下载进度监听的功能了。

拦截器属于OkHttp的高级功能，不过即使你之前并没有接触过拦截器，我相信你也能轻松看懂本篇文章的，因为它本身并不难。

确定了实现思路之后，那我们就开始动手吧。首先创建一个没有任何逻辑的空拦截器，新建ProgressInterceptor类并实现Interceptor接口，代码如下所示：

public class ProgressInterceptor implements Interceptor { 

    @Override 
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request(); 
        Response response = chain.proceed(request); 
        return response; 
    } 

}
1
2
3
4
5
6
7
8
9
10
这个拦截器中我们可以说是什么都没有做。就是拦截到了OkHttp的请求，然后调用proceed()方法去处理这个请求，最终将服务器响应的Response返回。

接下来我们需要启用这个拦截器，修改MyGlideModule中的代码，如下所示：

public class MyGlideModule implements GlideModule { 
    @Override 
    public void applyOptions(Context context, GlideBuilder builder) { 
    } 

    @Override 
    public void registerComponents(Context context, Glide glide) { 
        OkHttpClient.Builder builder = new OkHttpClient.Builder(); 
        builder.addInterceptor(new ProgressInterceptor()); 
        OkHttpClient okHttpClient = builder.build(); 
        glide.register(GlideUrl.class, InputStream.class, new OkHttpGlideUrlLoader.Factory(okHttpClient));
    } 
}
1
2
3
4
5
6
7
8
9
10
11
12
13
这里我们创建了一个OkHttpClient.Builder，然后调用addInterceptor()方法将刚才创建的ProgressInterceptor添加进去，最后将构建出来的新OkHttpClient对象传入到OkHttpGlideUrlLoader.Factory中即可。

好的，现在自定义的拦截器已经启用了，接下来就可以开始去实现下载进度监听的具体逻辑了。首先新建一个ProgressListener接口，用于作为进度监听回调的工具，如下所示：

public interface ProgressListener {

    void onProgress(int progress);

}
1
2
3
4
5
然后我们在ProgressInterceptor中加入注册下载监听和取消注册下载监听的方法。修改ProgressInterceptor中的代码，如下所示：

public class ProgressInterceptor implements Interceptor { 

    static final Map<String, ProgressListener> LISTENER_MAP = new HashMap<>();
    
    public static void addListener(String url, ProgressListener listener) {
        LISTENER_MAP.put(url, listener); 
    } 
    
    public static void removeListener(String url) { 
        LISTENER_MAP.remove(url); 
    } 
    
    @Override 
    public Response intercept(Chain chain) throws IOException { 
        Request request = chain.request(); 
        Response response = chain.proceed(request); 
        return response; 
    } 

}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
可以看到，这里使用了一个Map来保存注册的监听器，Map的键是一个URL地址。之所以要这么做，是因为你可能会使用Glide同时加载很多张图片，而这种情况下，必须要能区分出来每个下载进度的回调到底是对应哪个图片URL地址的。

接下来就要到今天最复杂的部分了，也就是下载进度的具体计算。我们需要新建一个ProgressResponseBody类，并让它继承自OkHttp的ResponseBody，然后在这个类当中去编写具体的监听下载进度的逻辑，代码如下所示：

public class ProgressResponseBody extends ResponseBody {

    private static final String TAG = "ProgressResponseBody";
    
    private BufferedSource bufferedSource;
    
    private ResponseBody responseBody;
    
    private ProgressListener listener;
    
    public ProgressResponseBody(String url, ResponseBody responseBody) {
        this.responseBody = responseBody;
        listener = ProgressInterceptor.LISTENER_MAP.get(url);
    }
    
    @Override
    public MediaType contentType() {
        return responseBody.contentType();
    }
    
    @Override
    public long contentLength() {
        return responseBody.contentLength();
    }
    
    @Override 
    public BufferedSource source() {
        if (bufferedSource == null) {
            bufferedSource = Okio.buffer(new ProgressSource(responseBody.source()));
        }
        return bufferedSource;
    }
    
    private class ProgressSource extends ForwardingSource {
    
        long totalBytesRead = 0;
    
        int currentProgress;
    
        ProgressSource(Source source) {
            super(source);
        }
    
        @Override 
        public long read(Buffer sink, long byteCount) throws IOException {
            long bytesRead = super.read(sink, byteCount);
            long fullLength = responseBody.contentLength();
            if (bytesRead == -1) {
                totalBytesRead = fullLength;
            } else {
                totalBytesRead += bytesRead;
            }
            int progress = (int) (100f * totalBytesRead / fullLength);
            Log.d(TAG, "download progress is " + progress);
            if (listener != null && progress != currentProgress) {
                listener.onProgress(progress);
            }
            if (listener != null && totalBytesRead == fullLength) {
                listener = null;
            }
            currentProgress = progress;
            return bytesRead;
        }
    }

}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
其实这段代码也不是很难，下面我来简单解释一下。首先，我们定义了一个ProgressResponseBody的构造方法，该构造方法中要求传入一个url参数和一个ResponseBody参数。那么很显然，url参数就是图片的url地址了，而ResponseBody参数则是OkHttp拦截到的原始的ResponseBody对象。然后在构造方法中，我们调用了ProgressInterceptor中的LISTENER_MAP来去获取该url对应的监听器回调对象，有了这个对象，待会就可以回调计算出来的下载进度了。

由于继承了ResponseBody类之后一定要重写contentType()、contentLength()和source()这三个方法，我们在contentType()和contentLength()方法中直接就调用传入的原始ResponseBody的contentType()和contentLength()方法即可，这相当于一种委托模式。但是在source()方法中，我们就必须加入点自己的逻辑了，因为这里要涉及到具体的下载进度计算。

那么我们具体看一下source()方法，这里先是调用了原始ResponseBody的source()方法来去获取Source对象，接下来将这个Source对象封装到了一个ProgressSource对象当中，最终再用Okio的buffer()方法封装成BufferedSource对象返回。

那么这个ProgressSource是什么呢？它是一个我们自定义的继承自ForwardingSource的实现类。ForwardingSource也是一个使用委托模式的工具，它不处理任何具体的逻辑，只是负责将传入的原始Source对象进行中转。但是，我们使用ProgressSource继承自ForwardingSource，那么就可以在中转的过程中加入自己的逻辑了。

可以看到，在ProgressSource中我们重写了read()方法，然后在read()方法中获取该次读取到的字节数以及下载文件的总字节数，并进行一些简单的数学计算就能算出当前的下载进度了。这里我先使用Log工具将算出的结果打印了一下，再通过前面获取到的回调监听器对象将结果进行回调。

好的，现在计算下载进度的逻辑已经完成了，那么我们快点在拦截器当中使用它吧。修改ProgressInterceptor中的代码，如下所示：

public class ProgressInterceptor implements Interceptor { 

    ... 
    
    @Override 
    public Response intercept(Chain chain) throws IOException { 
        Request request = chain.request(); 
        Response response = chain.proceed(request); 
        String url = request.url().toString(); 
        ResponseBody body = response.body(); 
        Response newResponse = response.newBuilder().body(new ProgressResponseBody(url, body)).build();
        return newResponse; 
    } 

}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
这里也都是一些OkHttp的简单用法。我们通过Response的newBuilder()方法来创建一个新的Response对象，并把它的body替换成刚才实现的ProgressResponseBody，最终将新的Response对象进行返回，这样计算下载进度的逻辑就能生效了。

代码写到这里，我们就可以来运行一下程序了。现在无论是加载任何网络上的图片，都应该是可以监听到它的下载进度的。

修改activity_main.xml中的代码，如下所示：

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="match_parent" 
    android:layout_height="match_parent" 
    android:orientation="vertical"> 

    <Button 
        android:layout_width="wrap_content" 
        android:layout_height="wrap_content" 
        android:text="Load Image" 
        android:onClick="loadImage" 
        /> 
    
    <ImageView 
        android:id="@+id/image" 
        android:layout_width="wrap_content" 
        android:layout_height="wrap_content" /> 
</LinearLayout>

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
很简单，这里使用了一个Button按钮来加载图片，使用了一个ImageView来展示图片。

然后修改MainActivity中的代码，如下所示：

public class MainActivity extends AppCompatActivity { 

    String url = "http://guolin.tech/book.png"; 
    
    ImageView image; 
    
    @Override 
    protected void onCreate(Bundle savedInstanceState) { 
        super.onCreate(savedInstanceState); 
        setContentView(R.layout.activity_main); 
        image = (ImageView) findViewById(R.id.image); 
    } 
    
    public void loadImage(View view) { 
        Glide.with(this) 
             .load(url) 
             .diskCacheStrategy(DiskCacheStrategy.NONE)
             .override(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL)
             .into(image); 
    } 
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
现在就可以运行一下程序了，效果如下图所示。


OK，图片已经加载出来了。那么怎么验证有没有成功监听到图片的下载进度呢？还记得我们刚才在ProgressResponseBody中加的打印日志吗？现在只要去logcat中观察一下就知道了，如下图所示：


由此可见，下载进度监听功能已经成功实现了。

进度显示
虽然现在我们已经能够监听到图片的下载进度了，但是这个进度目前还只能显示在控制台打印当中，这对于用户来说是没有任何意义的，因此我们下一步就是要想办法将下载进度显示到界面上。

现在修改MainActivity中的代码，如下所示：

public class MainActivity extends AppCompatActivity {

    String url = "http://guolin.tech/book.png";
    
    ImageView image;
    
    ProgressDialog progressDialog;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        image = (ImageView) findViewById(R.id.image);
        progressDialog = new ProgressDialog(this);
        progressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
        progressDialog.setMessage("加载中"); 
    }
    
    public void loadImage(View view) {
        ProgressInterceptor.addListener(url, new ProgressListener() {
            @Override
            public void onProgress(int progress) {
                progressDialog.setProgress(progress);
            }
        });
        Glide.with(this)
             .load(url)
             .diskCacheStrategy(DiskCacheStrategy.NONE)
             .override(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL)
             .into(new GlideDrawableImageViewTarget(image) {
                 @Override
                 public void onLoadStarted(Drawable placeholder) {
                     super.onLoadStarted(placeholder);
                     progressDialog.show();
                 }
    
                 @Override 
                 public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> animation) {
                     super.onResourceReady(resource, animation);
                     progressDialog.dismiss();
                     ProgressInterceptor.removeListener(url);
                 }
             });
    }

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
代码并不复杂。这里我们新增了一个ProgressDialog用来显示下载进度，然后在loadImage()方法中，调用了ProgressInterceptor.addListener()方法来去注册一个下载监听器，并在onProgress()回调方法中更新当前的下载进度。

最后，Glide的into()方法也做了修改，这次是into到了一个GlideDrawableImageViewTarget当中。我们重写了它的onLoadStarted()方法和onResourceReady()方法，从而实现当图片开始加载的时候显示进度对话框，当图片加载完成时关闭进度对话框的功能。

现在重新运行一下程序，效果如下图所示。


当然，不仅仅是静态图片，体积比较大的GIF图也是可以成功监听到下载进度的。比如我们把图片的url地址换成http://guolin.tech/test.gif，重新运行程序，效果如下图所示。

好了，这样我们就把带进度的Glide图片加载功能完整地实现了一遍。虽然这个例子当中的界面都比较粗糙，下载进度框也是使用的最简陋的，不过只要将功能学会了，界面那都不是事，大家后期可以自己进行各种界面优化。
————————————————



# Android图片加载框架最全解析（八），带你全面了解Glide 4的用法

Glide 4概述
刚才有说到，有些朋友觉得Glide 4相对于Glide 3改动非常大，其实不然。之所以大家会有这种错觉，是因为你将Glide 3的用法直接搬到Glide 4中去使用，结果IDE全面报错，然后大家可能就觉得Glide 4的用法完全变掉了。

其实Glide 4相对于Glide 3的变动并不大，只是你还没有了解它的变动规则而已。一旦你掌握了Glide 4的变动规则之后，你会发现大多数Glide 3的用法放到Glide 4上都还是通用的。

我对Glide 4进行了一个大概的研究之后，发现Glide 4并不能算是有什么突破性的升级，而更多是一些API工整方面的优化。相比于Glide 3的API，Glide 4进行了更加科学合理地调整，使得易读性、易写性、可扩展性等方面都有了不错的提升。但如果你已经对Glide 3非常熟悉的话，并不是就必须要切换到Glide 4上来，因为Glide 4上能实现的功能Glide 3也都能实现，而且Glide 4在性能方面也并没有什么提升。

但是对于新接触Glide的朋友而言，那就没必要再去学习Glide 3了，直接上手Glide 4就是最佳的选择了。

好了，对Glide 4进行一个基本的概述之后，接下来我们就要正式开始学习它的用法了。刚才我已经说了，Glide 4的用法相对于Glide 3其实改动并不大。在前面的七篇文章中，我们已经学习了Glide 3的基本用法、缓存机制、回调与监听、图片变换、自定义模块等用法，那么今天这篇文章的目标就很简单了，就是要掌握如何在Glide 4上实现之前所学习过的所有功能，那么我们现在就开始吧。

开始
要想使用Glide，首先需要将这个库引入到我们的项目当中。新建一个Glide4Test项目，然后在app/build.gradle文件当中添加如下依赖：

dependencies {
    implementation 'com.github.bumptech.glide:glide:4.4.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.4.0'
}
1
2
3
4
注意，相比于Glide 3，这里要多添加一个compiler的库，这个库是用于生成Generated API的，待会我们会讲到它。

另外，Glide中需要用到网络功能，因此你还得在AndroidManifest.xml中声明一下网络权限才行：

<uses-permission android:name="android.permission.INTERNET" />
1
就是这么简单，然后我们就可以自由地使用Glide中的任意功能了。

加载图片
现在我们就来尝试一下如何使用Glide来加载图片吧。比如这是一张图片的地址：

http://guolin.tech/book.png
1
然后我们想要在程序当中去加载这张图片。

那么首先打开项目的布局文件，在布局当中加入一个Button和一个ImageView，如下所示：

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Load Image"
        android:onClick="loadImage"
        />
    
    <ImageView
        android:id="@+id/image_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
为了让用户点击Button的时候能够将刚才的图片显示在ImageView上，我们需要修改MainActivity中的代码，如下所示：

public class MainActivity extends AppCompatActivity {

    ImageView imageView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
    }
    
    public void loadImage(View view) {
        String url = "http://guolin.tech/book.png";
        Glide.with(this).load(url).into(imageView);
    }

}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
没错，就是这么简单。现在我们来运行一下程序，效果如下图所示：


可以看到，一张网络上的图片已经被成功下载，并且展示到ImageView上了。

你会发现，到目前为止，Glide 4的用法和Glide 3是完全一样的，实际上核心的代码就只有这一行而已：

Glide.with(this).load(url).into(imageView);
1
仍然还是传统的三步走：先with()，再load()，最后into()。对这行代码的解读，我在 Android图片加载框架最全解析（一），Glide的基本用法 这篇文章中讲解的很清楚了，这里就不再赘述。

好了，现在你已经成功入门Glide 4了，那么接下来就让我们学习一下Glide 4的更多用法吧。

占位图
观察刚才加载网络图片的效果，你会发现，点击了Load Image按钮之后，要稍微等一会图片才会显示出来。这其实很容易理解，因为从网络上下载图片本来就是需要时间的。那么我们有没有办法再优化一下用户体验呢？当然可以，Glide提供了各种各样非常丰富的API支持，其中就包括了占位图功能。

顾名思义，占位图就是指在图片的加载过程中，我们先显示一张临时的图片，等图片加载出来了再替换成要加载的图片。

下面我们就来学习一下Glide占位图功能的使用方法，首先我事先准备好了一张loading.jpg图片，用来作为占位图显示。然后修改Glide加载部分的代码，如下所示：

RequestOptions options = new RequestOptions()
        .placeholder(R.drawable.loading);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
没错，就是这么简单。这里我们先创建了一个RequestOptions对象，然后调用它的placeholder()方法来指定占位图，再将占位图片的资源id传入到这个方法中。最后，在Glide的三步走之间加入一个apply()方法，来应用我们刚才创建的RequestOptions对象。

不过如果你现在重新运行一下代码并点击Load Image，很可能是根本看不到占位图效果的。因为Glide有非常强大的缓存机制，我们刚才加载图片的时候Glide自动就已经将它缓存下来了，下次加载的时候将会直接从缓存中读取，不会再去网络下载了，因而加载的速度非常快，所以占位图可能根本来不及显示。

因此这里我们还需要稍微做一点修改，来让占位图能有机会显示出来，修改代码如下所示：

RequestOptions options = new RequestOptions()
        .placeholder(R.drawable.loading)
        .diskCacheStrategy(DiskCacheStrategy.NONE);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
7
可以看到，这里在RequestOptions对象中又串接了一个diskCacheStrategy()方法，并传入DiskCacheStrategy.NONE参数，这样就可以禁用掉Glide的缓存功能。

关于Glide缓存方面的内容我们待会儿会进行更详细的讲解，这里只是为了测试占位图功能而加的一个额外配置，暂时你只需要知道禁用缓存必须这么写就可以了。

现在重新运行一下代码，效果如下图所示：


可以看到，当点击Load Image按钮之后会立即显示一张占位图，然后等真正的图片加载完成之后会将占位图替换掉。

除了这种加载占位图之外，还有一种异常占位图。异常占位图就是指，如果因为某些异常情况导致图片加载失败，比如说手机网络信号不好，这个时候就显示这张异常占位图。

异常占位图的用法相信你已经可以猜到了，首先准备一张error.jpg图片，然后修改Glide加载部分的代码，如下所示：

RequestOptions options = new RequestOptions()
        .placeholder(R.drawable.ic_launcher_background)
        .error(R.drawable.error)
        .diskCacheStrategy(DiskCacheStrategy.NONE);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
7
8
很简单，这里又串接了一个error()方法就可以指定异常占位图了。

其实看到这里，如果你熟悉Glide 3的话，相信你已经掌握Glide 4的变化规律了。在Glide 3当中，像placeholder()、error()、diskCacheStrategy()等等一系列的API，都是直接串联在Glide三步走方法中使用的。

而Glide 4中引入了一个RequestOptions对象，将这一系列的API都移动到了RequestOptions当中。这样做的好处是可以使我们摆脱冗长的Glide加载语句，而且还能进行自己的API封装，因为RequestOptions是可以作为参数传入到方法中的。

比如你就可以写出这样的Glide加载工具类：

public class GlideUtil {

    public static void load(Context context,
                            String url,
                            ImageView imageView,
                            RequestOptions options) {
        Glide.with(context)
             .load(url)
             .apply(options)
             .into(imageView);
    }

}
1
2
3
4
5
6
7
8
9
10
11
12
13
指定图片大小
实际上，使用Glide在大多数情况下我们都是不需要指定图片大小的，因为Glide会自动根据ImageView的大小来决定图片的大小，以此保证图片不会占用过多的内存从而引发OOM。

不过，如果你真的有这样的需求，必须给图片指定一个固定的大小，Glide仍然是支持这个功能的。修改Glide加载部分的代码，如下所示：

RequestOptions options = new RequestOptions()
        .override(200, 100);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
仍然非常简单，这里使用override()方法指定了一个图片的尺寸。也就是说，Glide现在只会将图片加载成200*100像素的尺寸，而不会管你的ImageView的大小是多少了。

如果你想加载一张图片的原始尺寸的话，可以使用Target.SIZE_ORIGINAL关键字，如下所示：

RequestOptions options = new RequestOptions()
        .override(Target.SIZE_ORIGINAL);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
这样的话，Glide就不会再去自动压缩图片，而是会去加载图片的原始尺寸。当然，这种写法也会面临着更高的OOM风险。

缓存机制
Glide的缓存设计可以说是非常先进的，考虑的场景也很周全。在缓存这一功能上，Glide又将它分成了两个模块，一个是内存缓存，一个是硬盘缓存。

这两个缓存模块的作用各不相同，内存缓存的主要作用是防止应用重复将图片数据读取到内存当中，而硬盘缓存的主要作用是防止应用重复从网络或其他地方重复下载和读取数据。

内存缓存和硬盘缓存的相互结合才构成了Glide极佳的图片缓存效果，那么接下来我们就来分别学习一下这两种缓存的使用方法。

首先来看内存缓存。

你要知道，默认情况下，Glide自动就是开启内存缓存的。也就是说，当我们使用Glide加载了一张图片之后，这张图片就会被缓存到内存当中，只要在它还没从内存中被清除之前，下次使用Glide再加载这张图片都会直接从内存当中读取，而不用重新从网络或硬盘上读取了，这样无疑就可以大幅度提升图片的加载效率。比方说你在一个RecyclerView当中反复上下滑动，RecyclerView中只要是Glide加载过的图片都可以直接从内存当中迅速读取并展示出来，从而大大提升了用户体验。

而Glide最为人性化的是，你甚至不需要编写任何额外的代码就能自动享受到这个极为便利的内存缓存功能，因为Glide默认就已经将它开启了。

那么既然已经默认开启了这个功能，还有什么可讲的用法呢？只有一点，如果你有什么特殊的原因需要禁用内存缓存功能，Glide对此提供了接口：

RequestOptions options = new RequestOptions()
        .skipMemoryCache(true);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
可以看到，只需要调用skipMemoryCache()方法并传入true，就表示禁用掉Glide的内存缓存功能。

接下来我们开始学习硬盘缓存方面的内容。

其实在刚刚学习占位图功能的时候，我们就使用过硬盘缓存的功能了。当时为了禁止Glide对图片进行硬盘缓存而使用了如下代码：

RequestOptions options = new RequestOptions()
        .diskCacheStrategy(DiskCacheStrategy.NONE);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
1
2
3
4
5
6
调用diskCacheStrategy()方法并传入DiskCacheStrategy.NONE，就可以禁用掉Glide的硬盘缓存功能了。

这个diskCacheStrategy()方法基本上就是Glide硬盘缓存功能的一切，它可以接收五种参数：

DiskCacheStrategy.NONE： 表示不缓存任何内容。
DiskCacheStrategy.DATA： 表示只缓存原始图片。
DiskCacheStrategy.RESOURCE： 表示只缓存转换过后的图片。
DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。
DiskCacheStrategy.AUTOMATIC： 表示让Glide根据图片资源智能地选择使用哪一种缓存策略（默认选项）。
其中，DiskCacheStrategy.DATA对应Glide 3中的DiskCacheStrategy.SOURCE，DiskCacheStrategy.RESOURCE对应Glide 3中的DiskCacheStrategy.RESULT。而DiskCacheStrategy.AUTOMATIC是Glide 4中新增的一种缓存策略，并且在不指定diskCacheStrategy的情况下默认使用就是的这种缓存策略。

上面五种参数的解释本身并没有什么难理解的地方，但是关于转换过后的图片这个概念大家可能需要了解一下。就是当我们使用Glide去加载一张图片的时候，Glide默认并不会将原始图片展示出来，而是会对图片进行压缩和转换（我们会在稍后学习这方面的内容）。总之就是经过种种一系列操作之后得到的图片，就叫转换过后的图片。

好的，关于Glide 4硬盘缓存的内容就讲到这里。想要了解更多Glide缓存方面的知识，可以参考 Android图片加载框架最全解析（三），深入探究Glide的缓存机制 这篇文章。

指定加载格式
我们都知道，Glide其中一个非常亮眼的功能就是可以加载GIF图片，而同样作为非常出色的图片加载框架的Picasso是不支持这个功能的。

而且使用Glide加载GIF图并不需要编写什么额外的代码，Glide内部会自动判断图片格式。比如我们将加载图片的URL地址改成一张GIF图，如下所示：

Glide.with(this)
     .load("http://guolin.tech/test.gif")
     .into(imageView);
1
2
3
现在重新运行一下代码，效果如下图所示：


也就是说，不管我们传入的是一张普通图片，还是一张GIF图片，Glide都会自动进行判断，并且可以正确地把它解析并展示出来。

但是如果我想指定加载格式该怎么办呢？就比如说，我希望加载的这张图必须是一张静态图片，我不需要Glide自动帮我判断它到底是静图还是GIF图。

想实现这个功能仍然非常简单，我们只需要再串接一个新的方法就可以了，如下所示：

Glide.with(this)
     .asBitmap()
     .load("http://guolin.tech/test.gif")
     .into(imageView);
1
2
3
4
可以看到，这里在with()方法的后面加入了一个asBitmap()方法，这个方法的意思就是说这里只允许加载静态图片，不需要Glide去帮我们自动进行图片格式的判断了。如果你传入的还是一张GIF图的话，Glide会展示这张GIF图的第一帧，而不会去播放它。

熟悉Glide 3的朋友对asBitmap()方法肯定不会陌生对吧？但是千万不要觉得这里就没有陷阱了，在Glide 3中的语法是先load()再asBitmap()的，而在Glide 4中是先asBitmap()再load()的。乍一看可能分辨不出来有什么区别，但如果你写错了顺序就肯定会报错了。

那么类似地，既然我们能强制指定加载静态图片，就也能强制指定加载动态图片，对应的方法是asGif()。而Glide 4中又新增了asFile()方法和asDrawable()方法，分别用于强制指定文件格式的加载和Drawable格式的加载，用法都比较简单，就不再进行演示了。

回调与监听
回调与监听这部分的内容稍微有点多，我们分成四部分来学习一下。

1. into()方法
我们都知道Glide的into()方法中是可以传入ImageView的。那么into()方法还可以传入别的参数吗？我们可以让Glide加载出来的图片不显示到ImageView上吗？答案是肯定的，这就需要用到自定义Target功能。

Glide中的Target功能多样且复杂，下面我就先简单演示一种SimpleTarget的用法吧，代码如下所示：

SimpleTarget<Drawable> simpleTarget = new SimpleTarget<Drawable>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<? super Drawable> transition) {
        imageView.setImageDrawable(resource);
    }
};

public void loadImage(View view) {
    Glide.with(this)
         .load("http://guolin.tech/book.png")
         .into(simpleTarget);
}
1
2
3
4
5
6
7
8
9
10
11
12
这里我们创建了一个SimpleTarget的实例，并且指定它的泛型是Drawable，然后重写了onResourceReady()方法。在onResourceReady()方法中，我们就可以获取到Glide加载出来的图片对象了，也就是方法参数中传过来的Drawable对象。有了这个对象之后你可以使用它进行任意的逻辑操作，这里我只是简单地把它显示到了ImageView上。

SimpleTarget的实现创建好了，那么只需要在加载图片的时候将它传入到into()方法中就可以了。

这里限于篇幅原因我只演示了自定义Target的简单用法，想学习更多相关的内容可以去阅读 Android图片加载框架最全解析（四），玩转Glide的回调与监听 。

2. preload()方法
Glide加载图片虽说非常智能，它会自动判断该图片是否已经有缓存了，如果有的话就直接从缓存中读取，没有的话再从网络去下载。但是如果我希望提前对图片进行一个预加载，等真正需要加载图片的时候就直接从缓存中读取，不想再等待慢长的网络加载时间了，这该怎么办呢？

不用担心，Glide专门给我们提供了预加载的接口，也就是preload()方法，我们只需要直接使用就可以了。

preload()方法有两个方法重载，一个不带参数，表示将会加载图片的原始尺寸，另一个可以通过参数指定加载图片的宽和高。

preload()方法的用法也非常简单，直接使用它来替换into()方法即可，如下所示：

Glide.with(this)
     .load("http://guolin.tech/book.png")
     .preload();
1
2
3
调用了预加载之后，我们以后想再去加载这张图片就会非常快了，因为Glide会直接从缓存当中去读取图片并显示出来，代码如下所示：

Glide.with(this)
     .load("http://guolin.tech/book.png")
     .into(imageView);
1
2
3
3. submit()方法
一直以来，我们使用Glide都是为了将图片显示到界面上。虽然我们知道Glide会在图片的加载过程中对图片进行缓存，但是缓存文件到底是存在哪里的，以及如何去直接访问这些缓存文件？我们都还不知道。

其实Glide将图片加载接口设计成这样也是希望我们使用起来更加的方便，不用过多去考虑底层的实现细节。但如果我现在就是想要去访问图片的缓存文件该怎么办呢？这就需要用到submit()方法了。

submit()方法其实就是对应的Glide 3中的downloadOnly()方法，和preload()方法类似，submit()方法也是可以替换into()方法的，不过submit()方法的用法明显要比preload()方法复杂不少。这个方法只会下载图片，而不会对图片进行加载。当图片下载完成之后，我们可以得到图片的存储路径，以便后续进行操作。

那么首先我们还是先来看下基本用法。submit()方法有两个方法重载：

submit()
submit(int width, int height)
其中submit()方法是用于下载原始尺寸的图片，而submit(int width, int height)则可以指定下载图片的尺寸。

这里就以submit()方法来举例。当调用了submit()方法后会立即返回一个FutureTarget对象，然后Glide会在后台开始下载图片文件。接下来我们调用FutureTarget的get()方法就可以去获取下载好的图片文件了，如果此时图片还没有下载完，那么get()方法就会阻塞住，一直等到图片下载完成才会有值返回。

下面我们通过一个例子来演示一下吧，代码如下所示：

public void downloadImage() {
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String url = "http://www.guolin.tech/book.png";
                final Context context = getApplicationContext();
                FutureTarget<File> target = Glide.with(context)
                        .asFile()
                        .load(url)
                        .submit();
                final File imageFile = target.get();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(context, imageFile.getPath(), Toast.LENGTH_LONG).show();
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }).start();
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
这段代码稍微有一点点长，我带着大家解读一下。首先，submit()方法必须要用在子线程当中，因为刚才说了FutureTarget的get()方法是会阻塞线程的，因此这里的第一步就是new了一个Thread。在子线程当中，我们先获取了一个Application Context，这个时候不能再用Activity作为Context了，因为会有Activity销毁了但子线程还没执行完这种可能出现。

接下来就是Glide的基本用法，只不过将into()方法替换成了submit()方法，并且还使用了一个asFile()方法来指定加载格式。submit()方法会返回一个FutureTarget对象，这个时候其实Glide已经开始在后台下载图片了，我们随时都可以调用FutureTarget的get()方法来获取下载的图片文件，只不过如果图片还没下载好线程会暂时阻塞住，等下载完成了才会把图片的File对象返回。

最后，我们使用runOnUiThread()切回到主线程，然后使用Toast将下载好的图片文件路径显示出来。

现在重新运行一下代码，效果如下图所示。


这样我们就能清晰地看出来图片完整的缓存路径是什么了。

4. listener()方法
其实listener()方法的作用非常普遍，它可以用来监听Glide加载图片的状态。举个例子，比如说我们刚才使用了preload()方法来对图片进行预加载，但是我怎样确定预加载有没有完成呢？还有如果Glide加载图片失败了，我该怎样调试错误的原因呢？答案都在listener()方法当中。

下面来看下listener()方法的基本用法吧，不同于刚才几个方法都是要替换into()方法的，listener()是结合into()方法一起使用的，当然也可以结合preload()方法一起使用。最基本的用法如下所示：

Glide.with(this)
     .load("http://www.guolin.tech/book.png")
     .listener(new RequestListener<Drawable>() {
         @Override
         public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
             return false;
         }

         @Override
         public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
             return false;
         }
     })
     .into(imageView);

这里我们在into()方法之前串接了一个listener()方法，然后实现了一个RequestListener的实例。其中RequestListener需要实现两个方法，一个onResourceReady()方法，一个onLoadFailed()方法。从方法名上就可以看出来了，当图片加载完成的时候就会回调onResourceReady()方法，而当图片加载失败的时候就会回调onLoadFailed()方法，onLoadFailed()方法中会将失败的GlideException参数传进来，这样我们就可以定位具体失败的原因了。

没错，listener()方法就是这么简单。不过还有一点需要处理，onResourceReady()方法和onLoadFailed()方法都有一个布尔值的返回值，返回false就表示这个事件没有被处理，还会继续向下传递，返回true就表示这个事件已经被处理掉了，从而不会再继续向下传递。举个简单点的例子，如果我们在RequestListener的onResourceReady()方法中返回了true，那么就不会再回调Target的onResourceReady()方法了。

关于回调与监听的内容就讲这么多吧，如果想要学习更多深入的内容以及源码解析，还是请参考这篇文章 Android图片加载框架最全解析（四），玩转Glide的回调与监听 。

图片变换
图片变换的意思就是说，Glide从加载了原始图片到最终展示给用户之前，又进行了一些变换处理，从而能够实现一些更加丰富的图片效果，如图片圆角化、圆形化、模糊化等等。

添加图片变换的用法非常简单，我们只需要在RequestOptions中串接transforms()方法，并将想要执行的图片变换操作作为参数传入transforms()方法即可，如下所示：

```kotlin
RequestOptions options = new RequestOptions()
        .transforms(...);
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

至于具体要进行什么样的图片变换操作，这个通常都是需要我们自己来写的。不过Glide已经内置了几种图片变换操作，我们可以直接拿来使用，比如CenterCrop、FitCenter、CircleCrop等。

但所有的内置图片变换操作其实都不需要使用transform()方法，Glide为了方便我们使用直接提供了现成的API：

```kotlin
RequestOptions options = new RequestOptions()
        .centerCrop();

RequestOptions options = new RequestOptions()
        .fitCenter();

RequestOptions options = new RequestOptions()
        .circleCrop();
```

当然，这些内置的图片变换API其实也只是对transform()方法进行了一层封装而已，它们背后的源码仍然还是借助transform()方法来实现的。

这里我们就选择其中一种内置的图片变换操作来演示一下吧，circleCrop()方法是用来对图片进行圆形化裁剪的，我们动手试一下，代码如下所示：

```kotlin
String url = "http://guolin.tech/book.png";
RequestOptions options = new RequestOptions()
        .circleCrop();
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

重新运行一下程序并点击加载图片按钮，效果如下图所示。


可以看到，现在展示的图片是对原图进行圆形化裁剪后得到的图片。

当然，除了使用内置的图片变换操作之外，我们完全可以自定义自己的图片变换操作。理论上，在对图片进行变换这个步骤中我们可以进行任何的操作，你想对图片怎么样都可以。包括圆角化、圆形化、黑白化、模糊化等等，甚至你将原图片完全替换成另外一张图都是可以的。

不过由于这部分内容相对于Glide 3没有任何的变化，因此就不再重复进行讲解了。想学习自定义图片变换操作的朋友们可以参考这篇文章 Android图片加载框架最全解析（五），Glide强大的图片变换功能 。

关于图片变换，最后我们再来看一个非常优秀的开源库，glide-transformations。它实现了很多通用的图片变换效果，如裁剪变换、颜色变换、模糊变换等等，使得我们可以非常轻松地进行各种各样的图片变换。

glide-transformations的项目主页地址是 https://github.com/wasabeef/glide-transformations 。

下面我们就来体验一下这个库的强大功能吧。首先需要将这个库引入到我们的项目当中，在app/build.gradle文件当中添加如下依赖：

```kotlin
dependencies {
    implementation 'jp.wasabeef:glide-transformations:3.0.1'
}
```

我们可以对图片进行单个变换处理，也可以将多种图片变换叠加在一起使用。比如我想同时对图片进行模糊化和黑白化处理，就可以这么写：

```kotlin
String url = "http://guolin.tech/book.png";
RequestOptions options = new RequestOptions()
        .transforms(new BlurTransformation(), new GrayscaleTransformation());
Glide.with(this)
     .load(url)
     .apply(options)
     .into(imageView);
```

可以看到，同时执行多种图片变换的时候，只需要将它们都传入到transforms()方法中即可。现在重新运行一下程序，效果如下图所示。


当然，这只是glide-transformations库的一小部分功能而已，更多的图片变换效果你可以到它的GitHub项目主页去学习。

自定义模块
自定义模块属于Glide中的高级功能，同时也是难度比较高的一部分内容。

这里我不可能在这一篇文章中将自定义模块的内容全讲一遍，限于篇幅的限制我只能讲一讲Glide 4中变化的这部分内容。关于Glide自定义模块的全部内容，请大家去参考 Android图片加载框架最全解析（六），探究Glide的自定义模块功能 这篇文章。

自定义模块功能可以将更改Glide配置，替换Glide组件等操作独立出来，使得我们能轻松地对Glide的各种配置进行自定义，并且又和Glide的图片加载逻辑没有任何交集，这也是一种低耦合编程方式的体现。下面我们就来学习一下自定义模块要如何实现。

首先定义一个我们自己的模块类，并让它继承自AppGlideModule，如下所示：

```kotlin
@GlideModule
public class MyAppGlideModule extends AppGlideModule {

@Override
public void applyOptions(Context context, GlideBuilder builder) {

}

@Override
public void registerComponents(Context context, Glide glide, Registry registry) {

}

}
```

可以看到，在MyAppGlideModule类当中，我们重写了applyOptions()和registerComponents()方法，这两个方法分别就是用来更改Glide配置以及替换Glide组件的。

注意在MyAppGlideModule类在上面，我们加入了一个@GlideModule的注解，这是Gilde 4和Glide 3最大的一个不同之处。在Glide 3中，我们定义了自定义模块之后，还必须在AndroidManifest.xml文件中去注册它才能生效，而在Glide 4中是不需要的，因为@GlideModule这个注解已经能够让Glide识别到这个自定义模块了。

这样的话，我们就将Glide自定义模块的功能完成了。后面只需要在applyOptions()和registerComponents()这两个方法中加入具体的逻辑，就能实现更改Glide配置或者替换Glide组件的功能了。详情还是请参考 Android图片加载框架最全解析（六），探究Glide的自定义模块功能 这篇文章，这里就不再展开讨论了。

使用Generated API
Generated API是Glide 4中全新引入的一个功能，它的工作原理是使用注解处理器 (Annotation Processor) 来生成出一个API，在Application模块中可使用该流式API一次性调用到RequestBuilder，RequestOptions和集成库中所有的选项。

这么解释有点拗口，简单点说，就是Glide 4仍然给我们提供了一套和Glide 3一模一样的流式API接口。毕竟有些人还是觉得Glide 3的API更好用一些，比如说我。

Generated API对于熟悉Glide 3的朋友来说那是再简单不过了，基本上就是和Glide 3一模一样的用法，只不过需要把Glide关键字替换成GlideApp关键字，如下所示：

```kotlin
GlideApp.with(this)
        .load(url)
        .placeholder(R.drawable.loading)
        .error(R.drawable.error)
        .skipMemoryCache(true)
        .diskCacheStrategy(DiskCacheStrategy.NONE)
        .override(Target.SIZE_ORIGINAL)
        .circleCrop()
        .into(imageView);
```

不过，有可能你的IDE中会提示找不到GlideApp这个类。这个类是通过编译时注解自动生成的，首先确保你的代码中有一个自定义的模块，并且给它加上了@GlideModule注解，也就是我们在上一节所讲的内容。然后在Android Studio中点击菜单栏Build -> Rebuild Project，GlideApp这个类就会自动生成了。

当然，Generated API所能做到的并不只是这些而已，它还可以对现有的API进行扩展，定制出任何属于你自己的API。

下面我来具体举个例子，比如说我们要求项目中所有图片的缓存策略全部都要缓存原始图片，那么每次在使用Glide加载图片的时候，都去指定diskCacheStrategy(DiskCacheStrategy.DATA)这么长长的一串代码，确实是让人比较心烦。这种情况我们就可以去定制一个自己的API了。

定制自己的API需要借助@GlideExtension和@GlideOption这两个注解。创建一个我们自定义的扩展类，代码如下所示：

```kotlin
@GlideExtension
public class MyGlideExtension {

private MyGlideExtension() {

}

@GlideOption
public static void cacheSource(RequestOptions options) {
    options.diskCacheStrategy(DiskCacheStrategy.DATA);
}

}
```

这里我们定义了一个MyGlideExtension类，并且给加上了一个@GlideExtension注解，然后要将这个类的构造函数声明成private，这都是必须要求的写法。

接下来就可以开始自定义API了，这里我们定义了一个cacheSource()方法，表示只缓存原始图片，并给这个方法加上了@GlideOption注解。注意自定义API的方法都必须是静态方法，而且第一个参数必须是RequestOptions，后面你可以加入任意多个你想自定义的参数。

在cacheSource()方法中，我们仍然还是调用的diskCacheStrategy(DiskCacheStrategy.DATA)方法，所以说cacheSource()就是一层简化API的封装而已。

然后在Android Studio中点击菜单栏Build -> Rebuild Project，神奇的事情就会发生了，你会发现你已经可以使用这样的语句来加载图片了：

```kotlin
GlideApp.with(this)
        .load(url)
        .cacheSource()
        .into(imageView);
```

有了这个强大的功能之后，我们使用Glide就能变得更加灵活了。



# 6. 应用场景

根据Glide的特点和与其他图片加载库的对比，可以得出其使用场景：

- 需要更多的内容表现形式(如Gif)；
- 更高的性能要求（缓存 & 加载速度）；

# 参考

- [郭霖：Glide最全解析](https://blog.csdn.net/guolin_blog/category_9268670.html)

  [Android图片加载框架最全解析（一），Glide的基本用法](https://guolin.blog.csdn.net/article/details/53759439)

  [Android图片加载框架最全解析（四），玩转Glide的回调与监听](https://blog.csdn.net/guolin_blog/article/details/70215985)

  [Android图片加载框架最全解析（五），Glide强大的图片变换功能](https://blog.csdn.net/guolin_blog/article/details/71524668)

  [Android图片加载框架最全解析（六），探究Glide的自定义模块功能](https://blog.csdn.net/guolin_blog/article/details/78179422)

  [Android图片加载框架最全解析（七），实现带进度的Glide图片加载功能](https://blog.csdn.net/guolin_blog/article/details/78357251)

  [Android图片加载框架最全解析（八），带你全面了解Glide 4的用法](https://blog.csdn.net/guolin_blog/article/details/78582548)

  二三两篇是原理，在另外的笔记中整理。

[Carson带你学Android：图片加载库Glide使用教程](https://www.jianshu.com/p/c3a5518b58b2)

[Glide(版本4.13.0)配置OkHttp请求网络](https://blog.csdn.net/sinat_34388320/article/details/124800403)