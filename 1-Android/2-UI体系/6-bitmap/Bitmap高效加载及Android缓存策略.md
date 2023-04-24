大图加载原理也涉及到了Bitmap的使用。

#一、Bitmap（位图）基本概念
1、Bitmap是Android系统中**图像处理**最重要的类之一。
2、通过Bitmap可以获取到图片的信息（像素信息）。
3、获取到信息后，可以对其进行缩放、裁剪等操作。

总之：Bitmap为我们提供了对图像文件的操作支持。
就像File类，为我们提供了对本地文件的操作一样。



#二、Bitmap加载方式
Bitmap的加载主要通过BitmapFactory类来完成，部分方法如下：
```
        BitmapFactory.decodeResource();
        BitmapFactory.decodeStream();
        BitmapFactory.decodeFile();
        BitmapFactory.decodeByteArray();
        BitmapFactory.decodeFileDescriptor();
        BitmapFactory.decodeResourceStream();
```



#三、为什么要高效的加载Bitmap？
1、防止内存溢出
2、尽可能的节省内存开销
3、使我们的应用跑的更加顺畅

也可以理解为如果不高效加载，会带来什么问题。



#四、Bitmap如何高效加载
1、Bitmap的高效加载主要依赖于：**BitmapFactory.Options**类

2、BitmapFactory.Options详解
1）有几个重要的属性需要知道
* inJustDecodeBounds
表示只解码出图片信息
* outWidth & outHeight
* inSampleSize
采样率



#五、Android缓存
###1、缓存的概念
缓存就是将从服务器请求到的数据（Json、File）等保存到本地，这就是缓存。

###2、缓存常见的使用场景和优势
优势：
1）对一些不是经常发生变化的数据，直接使用本地缓存，提升应用响应速度
2）不再频繁的请求服务器，可以降低服务器的负载压力
3）一些特殊场景下的使用，例如：离线阅读
使用场景：
1）对Bitmap和File等大数据进行缓存，无需每次都去下载，尤其是使用ListView时
2）数据更新不需要实时更新，采用缓存机制。

###3、缓存策略
主要涉及三种：
* Android LruCache
* Android DiskLruCache
* SQLite实现缓存（不是很重要）

#####1）LruCache
1.LruCache缓存的概念
* Lru是计算机科学经常使用的一种近期最少使用算法
* LruCache内部采用的是LinkedHashMap
* LruCache的出现是为了取代SoftReference

2.使用中要注意的问题
* 内存缓存使用的还是内存，因此要根据系统动态的调整大小
* 主要版本适配，尽量使用support v4中的LruCache

3.LruCache的使用
使用步骤：
（1）创建并初始化LruCache对象
（2）定义加载步骤，先从内存中获取，再从网络获取
* 内存中有，对mLruCache进行get操作
* 内存中没有，从网络获取

（3）从网络获取后，对mLruCache进行put操作

```
/**
 * 用来加载网络图片，并缓存图片到本地
 *
 * @author Li Zongwei
 * @date 2020/9/14
 **/
public class SimpleImageLoader {
    private static SimpleImageLoader mLoader;

    public static SimpleImageLoader getInstance() {
        if (mLoader == null) {
            synchronized (SimpleImageLoader.class) {
                if (mLoader == null) {
                    mLoader = new SimpleImageLoader();
                }
            }
        }
        return mLoader;
    }

    private LruCache<String, Bitmap> mLruCache;

    /**
     * 用来初始化缓存对象
     */
    private SimpleImageLoader() {
        //作为内存缓存大小
        int maxSize = (int) (Runtime.getRuntime().maxMemory() / 8);
        mLruCache = new LruCache<String, Bitmap>(maxSize) {
            @Override
            protected int sizeOf(String key, Bitmap value) {
                return value.getByteCount();
            }
        };
    }

    /**
     * 用来加载网络图片
     *
     * @param view
     * @param url
     */
    public void displayImage(Activity activity, ImageView view, String url) {
        Bitmap bitmap = getBitmapFromCache(url);
        if (bitmap != null) {
            LogUtils.d("SimpleImageLoader","从内存中加载");
            view.setImageBitmap(bitmap);
        } else {
            LogUtils.d("SimpleImageLoader","通过网络加载");
            downloadImage(activity, view, url);
        }
    }

    /**
     * 从缓存中读取图片
     *
     * @param url
     * @return
     */
    private Bitmap getBitmapFromCache(String url) {
        return mLruCache.get(url);
    }

    /**
     * 将下载下来的图片保存到缓存中
     *
     * @param bitmap
     * @param url
     */
    private void putBitmapToCache(Bitmap bitmap, String url) {
        if (bitmap != null) {
            mLruCache.put(url, bitmap);
        }
    }

    private void downloadImage(final Activity activity, final ImageView imageView, final String url) {
        HttpUtil.sendOkHttpRequest(url, new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull IOException e) {
            }

            @Override
            public void onResponse(@NotNull Call call, @NotNull final Response response)
                    throws IOException {
                activity.runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        InputStream is = response.body().byteStream();
                        Bitmap bitmap = BitmapFactory.decodeStream(is);
                        imageView.setImageBitmap(bitmap);

                        putBitmapToCache(bitmap,url);
                    }
                });
            }
        });
    }
}
```

#####2）DiskLruCache的使用
1、基本概念
它可以方便的将数据缓存到本地

2、用法
i、通过DiskLruCache.open去初始化一个缓存对象
ii、通过DiskCache.get(String key)去获取到对应key下的缓存数据
iii、通过DiskCache.Editor对象将数据保存到本地

3、使用时要注意的问题
i、根据有无外置存储设置合适的缓存路径
有外置：/sdcard/Android/data/<application package>/cache
无外置：/data/data/Android/data/<application package>/cache
ii、缓存文件时对key有特殊的要求
只能字母/数字。

4、示例
i、添加依赖
```
implementation 'com.jakewharton:disklrucache:2.0.2'
```
ii、demo
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        final TextView tvDisplayValue = findViewById(R.id.tv_display_value);

        Button btnAddValue = findViewById(R.id.add_value);
        btnAddValue.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                addDiskCache("data1", "我是第一条缓存数据");
            }
        });

        Button btnGetValue = findViewById(R.id.get_value);
        btnGetValue.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                String value = getDiskCache("data1");
                tvDisplayValue.setText(value);
            }
        });
    }

    /**
     * 添加一条缓存
     *
     * @param key
     * @param value
     */
    public void addDiskCache(String key, String value) {
        try {
            DiskLruCache diskLruCache = DiskLruCache.open(getCacheDir(), 1, 1, 5 * 1024 * 1024);
            DiskLruCache.Editor editor = diskLruCache.edit(key);
            editor.newOutputStream(0).write(value.getBytes());
            editor.commit();
            diskLruCache.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取缓存
     * @param key
     * @return
     */
    public String getDiskCache(String key) {
        try {
            DiskLruCache diskLruCache = DiskLruCache.open(getCacheDir(), 1, 1, 5 * 1024 * 1024);
            String value = diskLruCache.get(key).getString(0);
            diskLruCache.close();

            return value;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "获取缓存失败";
    }
```

总结，DiskLruCache一般都用来缓存网络图片等，这里就过以下它的基本用法。

#六、总结
本文主要学习了Bitmap高效加载的原理：主要是依靠BitmapFactory.Options类的配置来完成的。另外，通过缓存将一些网络数据放到内存、本地也可以提高加载效率。