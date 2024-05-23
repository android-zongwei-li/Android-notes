# 前言

- 现在很多`App`里都内置了Web网页（`Hybrid App`），比如说很多电商平台，淘宝、京东、聚划算等等，如下图

  ![img](https:////upload-images.jianshu.io/upload_images/944365-4512e70dbf996c34.png?imageMogr2/auto-orient/strip|imageView2/2/w/297/format/webp)

  京东首页

  

- 那么这种该如何实现呢？其实这是`Android`里一个叫`WebView`组件实现

- 今天，我将献上一份全面 & 详细的 `WebView`攻略，含具体介绍、使用教程、与前端`JS`交互、缓存机制构建等等，希望您们会喜欢。

> **Carson带你学WebView系列文章**
>  [Carson带你学Android：这是一份全面&详细的WebView学习攻略](https://www.jianshu.com/p/3c94ae673e2a)
>  [Carson带你学Android：最全面、最易懂的Webview使用详解](https://www.jianshu.com/p/3c94ae673e2a)
>  [Carson带你学Android：全面总结WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)
>  [Carson带你学Android：手把手构建WebView缓存机制及资源预加载方案](https://www.jianshu.com/p/5e7075f4875f)
>  [Carson带你学Android：盘点你不知道的WebView漏洞](https://www.jianshu.com/p/3a345d27cd42)

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-7bd8554be2869181.png?imageMogr2/auto-orient/strip|imageView2/2/w/996/format/webp)

示意图

------

# 1. 简介

一个基于`webkit`引擎、展现`web`页面的控件

> a. `Android 4.4`前：`Android Webview`在低版本 & 高版本采用了不同的`webkit`版本的内核
>  b. `Android 4.4`后：直接使用了`Chrome`内核

------

# 2. 作用

- 在 `Android` 客户端上加载`h5`页面
- 在本地 与 `h5`页面实现交互 & 调用
- 其他：对 `url` 请求、页面加载、渲染、对话框 进行额外处理。

------

# 3. 具体使用

- `Webview`的使用主要包括：`Webview`类 及其 工具类（`WebSettings`类、`WebViewClient`类、`WebChromeClient`类）

![img](https:////upload-images.jianshu.io/upload_images/944365-6e249d5a3cc7d343.png?imageMogr2/auto-orient/strip|imageView2/2/w/730/format/webp)

示意图

- 下面我将详细介绍上述4个使用类 & 使用方法
- 具体请看文章：[Carson带你学Android：最全面、最易懂的Webview使用详解](https://www.jianshu.com/p/3c94ae673e2a)

------

# 4. WebView与 JS 的交互方式

- 在`Android WebView`的使用中，与前端`h5`页面交互的需求十分常见
- `Android` 与 `JS` 通过WebView互相调用方法，实际上是：`Android` 去调用`JS`的代码 + `JS`去调用`Android`的代码

> 二者沟通的桥梁是`WebView`

![img](https:////upload-images.jianshu.io/upload_images/944365-2757946772481525.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- 具体介绍请看文章：[Carson带你学Android：全面总结WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)

------

# 5. 使用漏洞

- `WebView` 使用过程中存在许多漏洞，容易造成用户数据泄露等等危险，而很多人往往会忽视这个问题
- `WebView`中，主要漏洞有3类：**任意代码执行漏洞、密码明文存储漏洞、域控制不严格漏洞**
- 漏洞具体介绍 & 修复方式请看文章：[Carson带你学Android：盘点你不知道的WebView漏洞](https://www.jianshu.com/p/3a345d27cd42)

------

# 6. 缓存机制构建

- `Android WebView`由于前端`h5`本身的原因，存在加载效率慢 & 流量耗费的性能问题，具体介绍如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-d0e842a6e92eef2c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

- 本文通过 **`H5`缓存机制 + 资源预加载 + 资源拦截**的方式 构建了一套`WebView`缓存机制，从而解决`Android WebView`的性能问题，最终提高用户使用体验
- 具体缓存机制的讲解请看文章：[Carson带你学Android：手把手构建WebView缓存机制及资源预加载方案](https://www.jianshu.com/p/5e7075f4875f)

至此，关于`Android WebView`的所有知识讲解完毕。

------

# 7. 总结

- 本文全面讲解了 `WebView`的相关知识，含具体介绍、使用教程、与前端`JS`交互、缓存机制构建等等，相信你对`Android WebView`的使用已经非常熟悉了。



# 前言

- 现在很多`App`里都内置了Web网页（`Hybrid App`），比如说很多电商平台，淘宝、京东、聚划算等等，如下图

  ![img](https:////upload-images.jianshu.io/upload_images/944365-4512e70dbf996c34.png?imageMogr2/auto-orient/strip|imageView2/2/w/297/format/webp)

  京东首页

  

- 那么这种该如何实现呢？其实这是`Android`里一个叫`WebView`组件实现

- 今天，我将献上一份全面介绍 `WebView`的常见用法。

> **Carson带你学WebView系列文章**
>  [Carson带你学Android：这是一份全面&详细的WebView学习攻略](https://www.jianshu.com/p/3c94ae673e2a)
>  [Carson带你学Android：最全面、易懂的Webview使用教程](https://www.jianshu.com/p/3c94ae673e2a)
>  [Carson带你学Android：全面总结WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)
>  [Carson带你学Android：手把手构建WebView缓存机制及资源预加载方案](https://www.jianshu.com/p/5e7075f4875f)
>  [Carson带你学Android：盘点你不知道的WebView漏洞](https://www.jianshu.com/p/3a345d27cd42)

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-b163bc8e9d87c16c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 1. 简介

`WebView`是一个基于`webkit`引擎、展现`web`页面的控件。

> Android的Webview在低版本和高版本采用了不同的webkit版本内核，4.4后直接使用了Chrome。

------

# 2. 作用

- 显示和渲染Web页面
- 直接使用html文件（网络上或本地assets中）作布局
- 可和JavaScript交互调用

> WebView控件功能强大，除了具有一般View的属性和设置外，还可以对url请求、页面加载、渲染、页面交互进行强大的处理。

------

# 3. 使用介绍

一般来说Webview可单独使用，可联合其工具类一起使用，所以接下来，我会介绍：

- Webview类自身的常见方法
- Webview的最常用的工具类：WebSettings类、WebViewClient类、WebChromeClient类
- Android 和 Js的交互

## 3.1 Webview类常用方法

#### 3.1.1 加载url

加载方式根据资源分为三种



```dart
  //方式1. 加载一个网页：
  webView.loadUrl("http://www.google.com/");

  //方式2：加载apk包中的html页面
  webView.loadUrl("file:///android_asset/test.html");

  //方式3：加载手机本地的html页面
   webView.loadUrl("content://com.android.htmlfileprovider/sdcard/test.html");

   // 方式4： 加载 HTML 页面的一小段内容
  WebView.loadData(String data, String mimeType, String encoding)
// 参数说明：
// 参数1：需要截取展示的内容
// 内容里不能出现 ’#’, ‘%’, ‘\’ , ‘?’ 这四个字符，若出现了需用 %23, %25, %27, %3f 对应来替代，否则会出现异常
// 参数2：展示内容的类型
// 参数3：字节码
```

#### 3.1.1 WebView的状态



```cpp
//激活WebView为活跃状态，能正常执行网页的响应
webView.onResume() ；

//当页面被失去焦点被切换到后台不可见状态，需要执行onPause
//通过onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。
webView.onPause()；

//当应用程序(存在webview)被切换到后台时，这个方法不仅仅针对当前的webview而是全局的全应用程序的webview
//它会暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。
webView.pauseTimers()
//恢复pauseTimers状态
webView.resumeTimers()；

//销毁Webview
//在关闭了Activity时，如果Webview的音乐或视频，还在播放。就必须销毁Webview
//但是注意：webview调用destory时,webview仍绑定在Activity上
//这是由于自定义webview构建时传入了该Activity的context对象
//因此需要先从父容器中移除webview,然后再销毁webview:
rootLayout.removeView(webView); 
webView.destroy();
```

#### 3.1.2 关于前进 / 后退网页



```cpp
//是否可以后退
Webview.canGoBack() 
//后退网页
Webview.goBack()

//是否可以前进                     
Webview.canGoForward()
//前进网页
Webview.goForward()

//以当前的index为起始点前进或者后退到历史记录中指定的steps
//如果steps为负数则为后退，正数则为前进
Webview.goBackOrForward(intsteps) 
```

**常见用法：Back键控制网页后退**

- 问题：在不做任何处理前提下 ，浏览网页时点击系统的“Back”键,整个 Browser 会调用 finish()而结束自身
- 目标：点击返回后，是网页回退而不是推出浏览器
- 解决方案：在当前Activity中处理并消费掉该 Back 事件



```csharp
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if ((keyCode == KEYCODE_BACK) && mWebView.canGoBack()) { 
        mWebView.goBack();
        return true;
    }
    return super.onKeyDown(keyCode, event);
}
```

#### 3.1.3 清除缓存数据



```cpp
//清除网页访问留下的缓存
//由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.
Webview.clearCache(true);

//清除当前webview访问的历史记录
//只会webview访问历史记录里的所有记录除了当前访问记录
Webview.clearHistory()；

//这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据
Webview.clearFormData()；
```

## 3.2 常用工具类

#### 3.2.1 WebSettings类

- 作用：对WebView进行配置和管理
- 配置步骤 & 常见方法：

**配置步骤1：添加访问网络权限**（AndroidManifest.xml）

> 这是前提！这是前提！这是前提！



```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

**配置步骤2：生成一个WebView组件（有两种方式）**



```cpp
//方式1：直接在在Activity中生成
WebView webView = new WebView(this)

//方法2：在Activity的layout文件里添加webview控件：
WebView webview = (WebView) findViewById(R.id.webView1);
```

**配置步骤3：进行配置-利用WebSettings子类**（常见方法）



```cpp
//声明WebSettings子类
WebSettings webSettings = webView.getSettings();

//如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript
webSettings.setJavaScriptEnabled(true);  
// 若加载的 html 里有JS 在执行动画等操作，会造成资源浪费（CPU、电量）
// 在 onStop 和 onResume 里分别把 setJavaScriptEnabled() 给设置成 false 和 true 即可

//支持插件
webSettings.setPluginsEnabled(true); 

//设置自适应屏幕，两者合用
webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小 
webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小

//缩放操作
webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件

//其他细节操作
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); //关闭webview中缓存 
webSettings.setAllowFileAccess(true); //设置可以访问文件 
webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口 
webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片
webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式
```

**常见用法：设置WebView缓存**

- 当加载 html 页面时，WebView会在/data/data/包名目录下生成 database 与 cache 两个文件夹
- 请求的 URL记录保存在 WebViewCache.db，而 URL的内容是保存在 WebViewCache 文件夹下
- 是否启用缓存：



```cpp
    //优先使用缓存: 
    WebView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); 
        //缓存模式如下：
        //LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
        //LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
        //LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
        //LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。

    //不使用缓存: 
    WebView.getSettings().setCacheMode(WebSettings.LOAD_NO_CACHE);
```

- 结合使用（离线加载）



```dart
if (NetStatusUtil.isConnected(getApplicationContext())) {
    webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);//根据cache-control决定是否从网络上取数据。
} else {
    webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);//没网，则从本地获取，即离线加载
}

webSettings.setDomStorageEnabled(true); // 开启 DOM storage API 功能
webSettings.setDatabaseEnabled(true);   //开启 database storage API 功能
webSettings.setAppCacheEnabled(true);//开启 Application Caches 功能

String cacheDirPath = getFilesDir().getAbsolutePath() + APP_CACAHE_DIRNAME;
webSettings.setAppCachePath(cacheDirPath); //设置  Application Caches 缓存目录
```

**注意：** 每个 Application 只调用一次 WebSettings.setAppCachePath()，WebSettings.setAppCacheMaxSize()

#### 3.2.2 WebViewClient类

- 作用：处理各种通知 & 请求事件
- 常见方法：

**常见方法1：shouldOverrideUrlLoading()**

- 作用：打开网页时不调用系统浏览器， 而是在本WebView中显示；在网页上的所有加载都经过这个方法,这个函数我们可以做很多操作。



```java
//步骤1. 定义Webview组件
Webview webview = (WebView) findViewById(R.id.webView1);

//步骤2. 选择加载方式
  //方式1. 加载一个网页：
  webView.loadUrl("http://www.google.com/");

  //方式2：加载apk包中的html页面
  webView.loadUrl("file:///android_asset/test.html");

  //方式3：加载手机本地的html页面
   webView.loadUrl("content://com.android.htmlfileprovider/sdcard/test.html");

//步骤3. 复写shouldOverrideUrlLoading()方法，使得打开网页时不调用系统浏览器， 而是在本WebView中显示
    webView.setWebViewClient(new WebViewClient(){
      @Override
      public boolean shouldOverrideUrlLoading(WebView view, String url) {
          view.loadUrl(url);
      return true;
      }
  });
```

**常见方法2：onPageStarted()**

- 作用：开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。



```java
   webView.setWebViewClient(new WebViewClient(){
     @Override
     public void  onPageStarted(WebView view, String url, Bitmap favicon) {
        //设定加载开始的操作
     }
 });
```

**常见方法3：onPageFinished()**

- 作用：在页面加载结束时调用。我们可以关闭loading 条，切换程序动作。



```java
    webView.setWebViewClient(new WebViewClient(){
      @Override
      public void onPageFinished(WebView view, String url) {
         //设定加载结束的操作
      }
  });
```

**常见方法4：onLoadResource()**

- 作用：在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。



```java
    webView.setWebViewClient(new WebViewClient(){
      @Override
      public boolean onLoadResource(WebView view, String url) {
         //设定加载资源的操作
      }
  });
```

**常见方法5：onReceivedError（）**

- 作用：加载页面的服务器出现错误时（如404）调用。

> App里面使用webview控件的时候遇到了诸如404这类的错误的时候，若也显示浏览器里面的那种错误提示页面就显得很丑陋了，那么这个时候我们的app就需要加载一个本地的错误提示页面，即webview如何加载一个本地的页面



```dart
//步骤1：写一个html文件（error_handle.html），用于出错时展示给用户看的提示页面
//步骤2：将该html文件放置到代码根目录的assets文件夹下

//步骤3：复写WebViewClient的onRecievedError方法
//该方法传回了错误码，根据错误类型可以进行不同的错误分类处理
    webView.setWebViewClient(new WebViewClient(){
      @Override
      public void onReceivedError(WebView view, int errorCode, String description, String failingUrl){
switch(errorCode)
                {
                case HttpStatus.SC_NOT_FOUND:
                    view.loadUrl("file:///android_assets/error_handle.html");
                    break;
                }
            }
        });
```

**常见方法6：onReceivedSslError()**

- 作用：处理https请求

> webView默认是不处理https请求的，页面显示空白，需要进行如下设置：



```java
webView.setWebViewClient(new WebViewClient() {    
        @Override    
        public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {    
            handler.proceed();    //表示等待证书响应
        // handler.cancel();      //表示挂起连接，为默认方式
        // handler.handleMessage(null);    //可做其他处理
        }    
    });  

// 特别注意：5.1以上默认禁止了https和http混用，以下方式是开启
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
mWebView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
}
```

#### 3.2.3 WebChromeClient类

- 作用：辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等。
- 常见使用：

**常见方法1： onProgressChanged（）**

- 作用：获得网页的加载进度并显示



```java
webview.setWebChromeClient(new WebChromeClient(){

      @Override
      public void onProgressChanged(WebView view, int newProgress) {
          if (newProgress < 100) {
              String progress = newProgress + "%";
              progress.setText(progress);
            } else {
        }
    });
```

**常见方法2： onReceivedTitle（）**

- 作用：获取Web页中的标题

> 每个网页的页面都有一个标题，比如[www.baidu.com](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.baidu.com)这个页面的标题即“百度一下，你就知道”，那么如何知道当前webview正在加载的页面的title并进行设置呢？



```java
webview.setWebChromeClient(new WebChromeClient(){

    @Override
    public void onReceivedTitle(WebView view, String title) {
       titleview.setText(title)；
    }
```

**常见方法3： onJsAlert（）**

- 作用：支持javascript的警告框

> 一般情况下在 Android 中为 Toast，在文本里面加入\n就可以换行



```java
webview.setWebChromeClient(new WebChromeClient() {
            
            @Override
            public boolean onJsAlert(WebView view, String url, String message, final JsResult result)  {
    new AlertDialog.Builder(MainActivity.this)
            .setTitle("JsAlert")
            .setMessage(message)
            .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();
                }
            })
            .setCancelable(false)
            .show();
    return true;
            }
```

**常见方法4： onJsConfirm（）**

- 作用：支持javascript的确认框



```java
webview.setWebChromeClient(new WebChromeClient() {
        
            @Override
public boolean onJsConfirm(WebView view, String url, String message, final JsResult result) {
    new AlertDialog.Builder(MainActivity.this)
            .setTitle("JsConfirm")
            .setMessage(message)
            .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();
                }
            })
            .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.cancel();
                }
            })
            .setCancelable(false)
            .show();
// 返回布尔值：判断点击时确认还是取消
// true表示点击了确认；false表示点击了取消；
    return true;
}

            
```

**常见方法5： onJsPrompt（）**

- 作用：支持javascript输入框

> 点击确认返回输入框中的值，点击取消返回 null。



```java
webview.setWebChromeClient(new WebChromeClient() {
            @Override
            public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, final JsPromptResult result) {
    final EditText et = new EditText(MainActivity.this);
    et.setText(defaultValue);
    new AlertDialog.Builder(MainActivity.this)
            .setTitle(message)
            .setView(et)
            .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm(et.getText().toString());
                }
            })
            .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.cancel();
                }
            })
            .setCancelable(false)
            .show();

    return true;
}
            
```

# 3.3 WebView与JavaScript的交互

具体请看我写的文章：[最全面 & 最详细的 Android WebView与JS的交互方式 汇总](https://www.jianshu.com/p/345f4d8a5cfa)

# 3.4 注意事项：如何避免WebView内存泄露？

**3.4.1 不在xml中定义 Webview ，而是在需要的时候在Activity中创建，并且Context使用 getApplicationgContext()**



```csharp
LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
        mWebView = new WebView(getApplicationContext());
        mWebView.setLayoutParams(params);
        mLayout.addView(mWebView);
```

**3.4.2 在 Activity 销毁（ WebView ）的时候，先让 WebView 加载null内容，然后移除 WebView，再销毁 WebView，最后置空。**



```java
@Override
    protected void onDestroy() {
        if (mWebView != null) {
            mWebView.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
            mWebView.clearHistory();

            ((ViewGroup) mWebView.getParent()).removeView(mWebView);
            mWebView.destroy();
            mWebView = null;
        }
        super.onDestroy();
    }
```

------

# 4. 实例

- 目标：实现显示“[www.baidu.com](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.baidu.com)”、获取其标题、提示加载开始 & 结束和获取加载进度
- 具体实现：

**步骤1：添加访问网络权限**

> 这是前提！这是前提！这是前提！

*AndroidManifest.xml*



```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

**步骤2：主布局**
 *activity_main.xml*



```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"

    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.carson_ho.webview_demo.MainActivity">


   <!-- 获取网站的标题-->
    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""/>

    <!--开始加载提示-->
    <TextView
        android:id="@+id/text_beginLoading"
        android:layout_below="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""/>

    <!--获取加载进度-->
    <TextView
        android:layout_below="@+id/text_beginLoading"
        android:id="@+id/text_Loading"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""/>

    <!--结束加载提示-->
    <TextView
        android:layout_below="@+id/text_Loading"
        android:id="@+id/text_endLoading"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""/>
    
    <!--显示网页区域-->
    <WebView
        android:id="@+id/webView1"
        android:layout_below="@+id/text_endLoading"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_marginTop="10dp" />
</RelativeLayout>
```

**步骤3：根据需要实现的功能从而使用相应的子类及其方法（注释很清楚了）**
 *MainActivity.java*



```java
package com.example.carson_ho.webview_demo;

import android.graphics.Bitmap;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.KeyEvent;
import android.view.ViewGroup;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.TextView;


public class MainActivity extends AppCompatActivity {
    WebView mWebview;
    WebSettings mWebSettings;
    TextView beginLoading,endLoading,loading,mtitle;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        mWebview = (WebView) findViewById(R.id.webView1);
        beginLoading = (TextView) findViewById(R.id.text_beginLoading);
        endLoading = (TextView) findViewById(R.id.text_endLoading);
        loading = (TextView) findViewById(R.id.text_Loading);
        mtitle = (TextView) findViewById(R.id.title);

        mWebSettings = mWebview.getSettings();

        mWebview.loadUrl("http://www.baidu.com/");

        
        //设置不用系统浏览器打开,直接显示在当前Webview
        mWebview.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                view.loadUrl(url);
                return true;
            }
        });

        //设置WebChromeClient类
        mWebview.setWebChromeClient(new WebChromeClient() {


            //获取网站标题
            @Override
            public void onReceivedTitle(WebView view, String title) {
                System.out.println("标题在这里");
                mtitle.setText(title);
            }


            //获取加载进度
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                if (newProgress < 100) {
                    String progress = newProgress + "%";
                    loading.setText(progress);
                } else if (newProgress == 100) {
                    String progress = newProgress + "%";
                    loading.setText(progress);
                }
            }
        });


        //设置WebViewClient类
        mWebview.setWebViewClient(new WebViewClient() {
            //设置加载前的函数
            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                System.out.println("开始加载了");
                beginLoading.setText("开始加载了");

            }

            //设置结束加载函数
            @Override
            public void onPageFinished(WebView view, String url) {
                endLoading.setText("结束加载了");

            }
        });
    }

    //点击返回上一页面而不是退出浏览器
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK && mWebview.canGoBack()) {
            mWebview.goBack();
            return true;
        }

        return super.onKeyDown(keyCode, event);
    }

    //销毁Webview
    @Override
    protected void onDestroy() {
        if (mWebview != null) {
            mWebview.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
            mWebview.clearHistory();

            ((ViewGroup) mWebview.getParent()).removeView(mWebview);
            mWebview.destroy();
            mWebview = null;
        }
        super.onDestroy();
    }
}
```

### Demo地址

源代码：[Carson_Ho的Github-WebviewDemo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FCarson-Ho%2FWebview_Cache)

------

# 5. 总结

- 本文全面介绍了`Webview`，总结如下

![img](https:////upload-images.jianshu.io/upload_images/944365-6e249d5a3cc7d343.png?imageMogr2/auto-orient/strip|imageView2/2/w/730/format/webp)





# Carson带你学Android：全面总结WebView与 JS 的交互方式

# 前言

- 现在很多App里都内置了Web网页（Hybrid App），比如说很多电商平台，淘宝、京东、聚划算等等，如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-4512e70dbf996c34.png?imageMogr2/auto-orient/strip|imageView2/2/w/297/format/webp)

京东首页

- 上述功能是由Android的WebView实现的，其中涉及到Android客户端与Web网页交互的实现
- 今天我将全面介绍**Android通过WebView与JS交互**的全面方式

> **Carson带你学WebView系列文章**
>  [Carson带你学Android：这是一份全面&详细的WebView学习攻略](https://www.jianshu.com/p/3c94ae673e2a)
>  [Carson带你学Android：最全面、易懂的Webview使用教程](https://www.jianshu.com/p/3c94ae673e2a)
>  [Carson带你学Android：全面总结WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)
>  [Carson带你学Android：手把手构建WebView缓存机制及资源预加载方案](https://www.jianshu.com/p/5e7075f4875f)
>  [Carson带你学Android：盘点你不知道的WebView漏洞](https://www.jianshu.com/p/3a345d27cd42)

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-29c6a46c81304f4f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

目录

------

# 1. 交互方式总结

Android与JS通过WebView互相调用方法，实际上是：

- Android去调用JS的代码
- JS去调用Android的代码

> 二者沟通的桥梁是WebView

**对于Android调用JS代码的方法有2种：**

1. 通过`WebView`的`loadUrl（）`
2. 通过`WebView`的`evaluateJavascript（）`

**对于JS调用Android代码的方法有3种：**

1. 通过`WebView`的`addJavascriptInterface（）`进行对象映射
2. 通过 `WebViewClient` 的`shouldOverrideUrlLoading ()`方法回调拦截 url
3. 通过 `WebChromeClient` 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt（）`方法回调拦截JS对话框`alert()`、`confirm()`、`prompt（）` 消息

------

# 2. 具体分析

## 2.1  Android通过WebView调用 JS 代码

对于Android调用JS代码的方法有2种：

1. 通过`WebView`的`loadUrl（）`
2. 通过`WebView`的`evaluateJavascript（）`

### 方式1：通过`WebView`的`loadUrl（）`

- 实例介绍：点击Android按钮，即调用WebView JS（文本名为`javascript`）中callJS（）
- 具体使用：

**步骤1：将需要调用的JS代码以`.html`格式放到src/main/assets文件夹里**

> 1. 为了方便展示，本文是采用Andorid调用本地JS代码说明；
> 2. 实际情况时，Android更多的是调用远程JS代码，即将加载的JS代码路径改成url即可

*需要加载JS代码：javascript.html*



```xml
// 文本名：javascript
<!DOCTYPE html>
<html>

   <head>
      <meta charset="utf-8">
      <title>Carson_Ho</title>
      
// JS代码
     <script>
// Android需要调用的方法
   function callJS(){
      alert("Android调用了JS的callJS方法");
   }
</script>

   </head>

</html>
```

**步骤2：在Android里通过WebView设置调用JS代码**

*Android代码：MainActivity.java*

> 注释已经非常清楚



```java
 public class MainActivity extends AppCompatActivity {

    WebView mWebView;
    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWebView =(WebView) findViewById(R.id.webview);

        WebSettings webSettings = mWebView.getSettings();

        // 设置与Js交互的权限
        webSettings.setJavaScriptEnabled(true);
        // 设置允许JS弹窗
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

        // 先载入JS代码
        // 格式规定为:file:///android_asset/文件名.html
        mWebView.loadUrl("file:///android_asset/javascript.html");

        button = (Button) findViewById(R.id.button);


        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 通过Handler发送消息
                mWebView.post(new Runnable() {
                    @Override
                    public void run() {

                        // 注意调用的JS方法名要对应上
                        // 调用javascript的callJS()方法
                        mWebView.loadUrl("javascript:callJS()");
                    }
                });
                
            }
        });

        // 由于设置了弹窗检验调用结果,所以需要支持js对话框
        // webview只是载体，内容的渲染需要使用webviewChromClient类去实现
        // 通过设置WebChromeClient对象处理JavaScript的对话框
        //设置响应js 的Alert()函数
        mWebView.setWebChromeClient(new WebChromeClient() {
            @Override
            public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
                AlertDialog.Builder b = new AlertDialog.Builder(MainActivity.this);
                b.setTitle("Alert");
                b.setMessage(message);
                b.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        result.confirm();
                    }
                });
                b.setCancelable(false);
                b.create().show();
                return true;
            }

        });


    }
}
```

![img](https:////upload-images.jianshu.io/upload_images/944365-826d0aa065f70cb1.png?imageMogr2/auto-orient/strip|imageView2/2/w/657/format/webp)

效果图

**特别注意：JS代码调用一定要在 `onPageFinished（）` 回调之后才能调用，否则不会调用。**

> `onPageFinished()`属于WebViewClient类的方法，主要在页面加载结束时调用

# 方式2：通过`WebView`的`evaluateJavascript（）`

- 优点：该方法比第一种方法效率更高、使用更简洁。

> 1. 因为该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会。
> 2. Android 4.4 后才可使用

- 具体使用



```tsx
// 只需要将第一种方法的loadUrl()换成下面该方法即可
    mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
}
```

## 2.1.2 方法对比

![img](https:////upload-images.jianshu.io/upload_images/944365-30f095d4c9e638fd.png?imageMogr2/auto-orient/strip|imageView2/2/w/760/format/webp)

方式对比图

## 2.1.3 使用建议

两种方法混合使用，即Android 4.4以下使用方法1，Android 4.4以上方法2



```dart
// Android版本变量
final int version = Build.VERSION.SDK_INT;
// 因为该方法在 Android 4.4 版本才可使用，所以使用时需进行版本判断
if (version < 18) {
    mWebView.loadUrl("javascript:callJS()");
} else {
    mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
}
```

# 2.2 JS通过WebView调用 Android 代码

对于JS调用Android代码的方法有3种：

1. 通过`WebView`的`addJavascriptInterface（）`进行对象映射
2. 通过 `WebViewClient` 的`shouldOverrideUrlLoading ()`方法回调拦截 url
3. 通过 `WebChromeClient` 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt（）`方法回调拦截JS对话框`alert()`、`confirm()`、`prompt（）` 消息

## 2.2.1 方法分析

#### 方式1：通过 `WebView`的`addJavascriptInterface（）`进行对象映射

**步骤1：定义一个与JS对象映射关系的Android类：AndroidtoJs**

*AndroidtoJs.java*（注释已经非常清楚）



```java
// 继承自Object类
public class AndroidtoJs extends Object {

    // 定义JS需要调用的方法
    // 被JS调用的方法必须加入@JavascriptInterface注解
    @JavascriptInterface
    public void hello(String msg) {
        System.out.println("JS调用了Android的hello方法");
    }
}
```

**步骤2：将需要调用的JS代码以`.html`格式放到src/main/assets文件夹里**

*需要加载JS代码：javascript.html*



```xml
<!DOCTYPE html>
<html>
   <head>
      <meta charset="utf-8">
      <title>Carson</title>  
      <script>
         
        
         function callAndroid(){
        // 由于对象映射，所以调用test对象等于调用Android映射的对象
            test.hello("js调用了android中的hello方法");
         }
      </script>
   </head>
   <body>
      //点击按钮则调用callAndroid函数
      <button type="button" id="button1" onclick="callAndroid()"></button>
   </body>
</html>
```

**步骤3：在Android里通过WebView设置Android类与JS代码的映射**

> 详细请看注释



```java
public class MainActivity extends AppCompatActivity {

    WebView mWebView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWebView = (WebView) findViewById(R.id.webview);
        WebSettings webSettings = mWebView.getSettings();

        // 设置与Js交互的权限
        webSettings.setJavaScriptEnabled(true);

        // 通过addJavascriptInterface()将Java对象映射到JS对象
        //参数1：Javascript对象名
        //参数2：Java对象名
        mWebView.addJavascriptInterface(new AndroidtoJs(), "test");//AndroidtoJS类对象映射到js的test对象

        // 加载JS代码
        // 格式规定为:file:///android_asset/文件名.html
        mWebView.loadUrl("file:///android_asset/javascript.html");
```

![img](https:////upload-images.jianshu.io/upload_images/944365-7fe5cb840286e867.png?imageMogr2/auto-orient/strip|imageView2/2/w/736/format/webp)

效果图

# 特点

- 优点：使用简单

> 仅将Android对象和JS对象映射即可

- 缺点：存在严重的漏洞问题，具体请看文章：[你不知道的 Android WebView 使用漏洞](https://www.jianshu.com/p/3a345d27cd42)

# 方式2：通过 `WebViewClient` 的方法`shouldOverrideUrlLoading ()`回调拦截 url

- 具体原理：

1. Android通过 `WebViewClient` 的回调方法`shouldOverrideUrlLoading ()`拦截 url
2. 解析该 url 的协议
3. 如果检测到是预先约定好的协议，就调用相应方法

> 即JS需要调用Android的方法

- 具体使用：
   **步骤1：**在JS约定所需要的Url协议
   *JS代码：javascript.html*

> 以.html格式放到src/main/assets文件夹里



```xml
<!DOCTYPE html>
<html>

   <head>
      <meta charset="utf-8">
      <title>Carson_Ho</title>
      
     <script>
         function callAndroid(){
            /*约定的url协议为：js://webview?arg1=111&arg2=222*/
            document.location = "js://webview?arg1=111&arg2=222";
         }
      </script>
</head>

<!-- 点击按钮则调用callAndroid（）方法  -->
   <body>
     <button type="button" id="button1" onclick="callAndroid()">点击调用Android代码</button>
   </body>
</html>
```

当该JS通过Android的`mWebView.loadUrl("file:///android_asset/javascript.html")`加载后，就会回调`shouldOverrideUrlLoading （）`，接下来继续看步骤2：

**步骤2：在Android通过WebViewClient复写`shouldOverrideUrlLoading （）`**

*MainActivity.java*



```dart
public class MainActivity extends AppCompatActivity {

    WebView mWebView;
//    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWebView = (WebView) findViewById(R.id.webview);

        WebSettings webSettings = mWebView.getSettings();

        // 设置与Js交互的权限
        webSettings.setJavaScriptEnabled(true);
        // 设置允许JS弹窗
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

        // 步骤1：加载JS代码
        // 格式规定为:file:///android_asset/文件名.html
        mWebView.loadUrl("file:///android_asset/javascript.html");


// 复写WebViewClient类的shouldOverrideUrlLoading方法
mWebView.setWebViewClient(new WebViewClient() {
                                      @Override
                                      public boolean shouldOverrideUrlLoading(WebView view, String url) {

                                          // 步骤2：根据协议的参数，判断是否是所需要的url
                                          // 一般根据scheme（协议格式） & authority（协议名）判断（前两个参数）
                                          //假定传入进来的 url = "js://webview?arg1=111&arg2=222"（同时也是约定好的需要拦截的）

                                          Uri uri = Uri.parse(url);                                 
                                          // 如果url的协议 = 预先约定的 js 协议
                                          // 就解析往下解析参数
                                          if ( uri.getScheme().equals("js")) {

                                              // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议
                                              // 所以拦截url,下面JS开始调用Android需要的方法
                                              if (uri.getAuthority().equals("webview")) {

                                                 //  步骤3：
                                                  // 执行JS所需要调用的逻辑
                                                  System.out.println("js调用了Android的方法");
                                                  // 可以在协议上带有参数并传递到Android上
                                                  HashMap<String, String> params = new HashMap<>();
                                                  Set<String> collection = uri.getQueryParameterNames();

                                              }

                                              return true;
                                          }
                                          return super.shouldOverrideUrlLoading(view, url);
                                      }
                                  }
        );
   }
        }
```

![img](https:////upload-images.jianshu.io/upload_images/944365-33d26e9225426020.png?imageMogr2/auto-orient/strip|imageView2/2/w/714/format/webp)

效果图

# 特点

- 优点：不存在方式1的漏洞；
- 缺点：JS获取Android方法的返回值复杂。

> 如果JS想要得到Android方法的返回值，只能通过 WebView 的 `loadUrl （）`去执行 JS 方法把返回值传递回去，相关的代码如下：



```jsx
// Android：MainActivity.java
mWebView.loadUrl("javascript:returnResult(" + result + ")");

// JS：javascript.html
function returnResult(result){
    alert("result is" + result);
}
```

# 方式3：通过 `WebChromeClient` 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt（）`方法回调拦截JS对话框`alert()`、`confirm()`、`prompt（）` 消息

在JS中，有三个常用的对话框方法：

![img](https:////upload-images.jianshu.io/upload_images/944365-1385f748618af886.png?imageMogr2/auto-orient/strip|imageView2/2/w/773/format/webp)

Paste_Image.png

方式3的原理：Android通过 `WebChromeClient` 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt（）`方法回调分别拦截JS对话框
 （即上述三个方法），得到他们的消息内容，然后解析即可。

下面的例子将用**拦截 JS的输入框（即`prompt（）`方法）**说明 ：

> 1. 常用的拦截是：拦截 JS的输入框（即`prompt（）`方法）
> 2. 因为只有`prompt（）`可以返回任意类型的值，操作最全面方便、更加灵活；而alert（）对话框没有返回值；confirm（）对话框只能返回两种状态（确定 / 取消）两个值

**步骤1：加载JS代码，如下：**
 *javascript.html*

> 以.html格式放到src/main/assets文件夹里



```xml
<!DOCTYPE html>
<html>
   <head>
      <meta charset="utf-8">
      <title>Carson_Ho</title>
      
     <script>
        
    function clickprompt(){
    // 调用prompt（）
    var result=prompt("js://demo?arg1=111&arg2=222");
    alert("demo " + result);
}

      </script>
</head>

<!-- 点击按钮则调用clickprompt()  -->
   <body>
     <button type="button" id="button1" onclick="clickprompt()">点击调用Android代码</button>
   </body>
</html>
```

当使用`mWebView.loadUrl("file:///android_asset/javascript.html")`加载了上述JS代码后，就会触发回调`onJsPrompt（）`，具体如下：

> 1. 如果是拦截警告框（即`alert()`），则触发回调`onJsAlert（）`；
> 2. 如果是拦截确认框（即`confirm()`），则触发回调`onJsConfirm（）`；

**步骤2：在Android通过`WebChromeClient`复写`onJsPrompt（）`**



```tsx
public class MainActivity extends AppCompatActivity {

    WebView mWebView;
//    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWebView = (WebView) findViewById(R.id.webview);

        WebSettings webSettings = mWebView.getSettings();

        // 设置与Js交互的权限
        webSettings.setJavaScriptEnabled(true);
        // 设置允许JS弹窗
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

// 先加载JS代码
        // 格式规定为:file:///android_asset/文件名.html
        mWebView.loadUrl("file:///android_asset/javascript.html");


        mWebView.setWebChromeClient(new WebChromeClient() {
                                        // 拦截输入框(原理同方式2)
                                        // 参数message:代表promt（）的内容（不是url）
                                        // 参数result:代表输入框的返回值
                                        @Override
                                        public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
                                            // 根据协议的参数，判断是否是所需要的url(原理同方式2)
                                            // 一般根据scheme（协议格式） & authority（协议名）判断（前两个参数）
                                            //假定传入进来的 url = "js://webview?arg1=111&arg2=222"（同时也是约定好的需要拦截的）

                                            Uri uri = Uri.parse(message);
                                            // 如果url的协议 = 预先约定的 js 协议
                                            // 就解析往下解析参数
                                            if ( uri.getScheme().equals("js")) {

                                                // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议
                                                // 所以拦截url,下面JS开始调用Android需要的方法
                                                if (uri.getAuthority().equals("webview")) {

                                                    //
                                                    // 执行JS所需要调用的逻辑
                                                    System.out.println("js调用了Android的方法");
                                                    // 可以在协议上带有参数并传递到Android上
                                                    HashMap<String, String> params = new HashMap<>();
                                                    Set<String> collection = uri.getQueryParameterNames();

                                                    //参数result:代表消息框的返回值(输入值)
                                                    result.confirm("js调用了Android的方法成功啦");
                                                }
                                                return true;
                                            }
                                            return super.onJsPrompt(view, url, message, defaultValue, result);
                                        }

// 通过alert()和confirm()拦截的原理相同，此处不作过多讲述

                                        // 拦截JS的警告框
                                        @Override
                                        public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
                                            return super.onJsAlert(view, url, message, result);
                                        }

                                        // 拦截JS的确认框
                                        @Override
                                        public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
                                            return super.onJsConfirm(view, url, message, result);
                                        }
                                    }
        );


            }

        }
```

![img](https:////upload-images.jianshu.io/upload_images/944365-018de3f2ba1828c0.png?imageMogr2/auto-orient/strip|imageView2/2/w/764/format/webp)

效果图

- Demo地址
   上述所有代码均存放在：[Carson_Ho的Github地址 ： WebView Demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FCarson-Ho%2FWebview_Cache)

## 2.2.2 三种方式的对比 & 使用场景

![img](https:////upload-images.jianshu.io/upload_images/944365-8c91481325a5253e.png?imageMogr2/auto-orient/strip|imageView2/2/w/890/format/webp)

示意图

------

# 3. 总结

- 本文主要对**Android通过WebView与JS的交互方式进行了全面介绍**

![img](https:////upload-images.jianshu.io/upload_images/944365-613b57c93dff2eb8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1189/format/webp)





# WebView的使用

在布局文件中定义一个 WebView：

```xml
<WebView
android:id="@+id/webview"
android:layout_width="match_parent"
android:layout_height="match_parent"
/>
```

然后在 Activity 中找到这个 WebView，并加载一个网页：

```kotlin
val myWebView: WebView = findViewById(R.id.webview)
myWebView.loadUrl("http://www.example.com")
```

但是这样直接加载网页到 WebView 是存在问题的。默认情况下，当你点击 WebView 中的链接时，Android 会启动设备上默认的浏览器应用，在那个浏览器中打开你的链接。为了能让链接在 WebView 中打开，你需要做一些额外的设置。

你需要给 WebView 设置一个自定义的 WebViewClient：

```kotlin
val myWebView: WebView = findViewById(R.id.webview)

myWebView.webViewClient = WebViewClient()

myWebView.loadUrl("http://www.example.com")
```

现在，所有的链接都会在 WebView 中打开了。

## 加载网页内容

WebView 能够显示两种类型的内容：从网络加载的网页和从应用本地资源加载的网页。

对于从网络加载的网页，最简单的方式是直接调用 `loadUrl()` 方法：

```kotlin
kotlin复制代码
val myWebView: WebView = findViewById(R.id.webview)

myWebView.loadUrl("http://www.example.com")
```

这样 WebView 就会开始加载指定的 URL。但是要记住，网络请求需要应用声明网络访问的权限，可以在 Manifest.xml 文件中添加如下声明：

```xml
xml复制代码
<uses-permission android:name="android.permission.INTERNET" />
```

这是最基本的网络权限，在进行任何网络请求时都需要添加。

除了从网络加载外，我们还可以从应用本地资源加载 HTML 内容。例如：

```kotlin
kotlin复制代码
val unencodedHtml = "<html><body>'%23' is the percent code for ‘#‘ </body></html>"

val encodedHtml = Base64.encodeToString(unencodedHtml.toByteArray(), Base64.NO_PADDING)

myWebView.loadData(encodedHtml, "text/html", "base64")
```

这段代码首先创建了一个包含 HTML 内容的字符串，然后将这个字符串转换为 base64 编码，最后用 'loadData()' 方法在 WebView 中显示。

## 控制 WebView设置

`WebView` 提供了一个叫做 `WebSettings` 的类，该类为 WebView 提供了一系列的设置方法。你可以通过 `getSettings()` 方法来获取这个 WebView 的设置实例。以下是一些常见的设置选项。

让我们先了解如何**启用 JavaScript**。默认情况下 WebView 是不支持 JavaScript 的，比如，如果在网页中有 JavaScript 代码，那么这些代码在 WebView 中是不会被执行的。然而，对网页来说，JavaScript 是典型的响应事件驱动，如果不开启，网页的很多功能就无法使用。如果我们想要支持 JavaScript，我们需要更改 WebView 的设置：

```kotlin
kotlin复制代码
val myWebView: WebView = findViewById(R.id.webview)

  


// Enable JavaScript

val webSettings = myWebView.settings

webSettings.javaScriptEnabled = true
```

在这段代码中，首先我们获取了 WebView 的设置，然后开启了 JavaScript。

下一个常见的设置是**选择文本缩放**。默认情况下，WebView 不支持文本缩放。但是如果你希望用户能够用手势来缩放文本，你可以开启这个设置：

```kotlin
kotlin复制代码
webSettings.textZoom = 125 // where 100 is the default text zoom
```

此外， WebView 也可以支持用户通过手势来控制 WebView 的整体缩放，只需要打开下面的设置：

```kotlin
kotlin复制代码
webSettings.setSupportZoom(true)

webSettings.builtInZoomControls = true

webSettings.displayZoomControls = false
```

- setSupportZoom(true): 设定支持缩放
- builtInZoomControls = true: 显示页面上的放大缩小控件
- displayZoomControls = false: 不显示放大缩小控件，默认为 true。

## WebView与Javascript交互

### Android -> JavaScript

使用 WebView，我们可以做到在 Android 代码中调用 JavaScript 函数，并从 WebView 的 JavaScript 代码中调用 Android 代码。这样，就可以在 Java/Kotlin 代码中使用 Web 技术，使得 WebView 成为了创建复杂用户界面或处理 Web 内容的非常有用的工具。

首先让我们看看如何在 Android 代码中调用 JavaScript 函数。这可以通过 WebView 的 `loadUrl` 方法实现：

```kotlin
myWebView.loadUrl("javascript:myFunction()")
```

这里， `"myFunction()"` 是你要调用的 JavaScript 函数的名称。

### JavaScript -> Android

反过来，如果你想让你的 WebView 的 JavaScript 代码调用你的 Android 代码，你需要使用 JavaScript 接口 (JavaScript Interface)。 下面是一个简单的例子：

`JavaScriptInterface` 是一种允许 JavaScript 代码调用你的 Android 代码（在 WebView 中）的技术。其工作原理是，你将一个实现了一个或多个用于和 JavaScript 交互的方法的对象暴露给 JavaScript。

```kotlin
class MyJavaScriptInterface(private val context: Context) {

@JavascriptInterface

fun showToast(message: String) {

Toast.makeText(context, message, Toast.LENGTH_SHORT).show()

}
}

myWebView.addJavascriptInterface(MyJavaScriptInterface(this), "AndroidFunction")
```

那么，在你的网页中，你就可以这样使用上面定义的方法了：

```javascript
javascript复制代码
AndroidFunction.showToast("Hello, WebView!");
```

当调用 `addJavascriptInterface` 方法时，你需要注意以下几点：

1. 第一个参数是你要添加进去的对象，第二个参数是这个对象在 JavaScript 代码中的名称。
2. 这个对象的所有公共方法都会被添加到 JavaScript 的全局作用域，所以要特别注意方法的命名，以免与网页自带方法重名。
3. 以安全为首要考虑，不可将含有敏感数据或可修改应用数据的方法暴露给 JavaScript。

#### Webview调试之在控制台中模拟调用 Android 方法

------

前面我们介绍了如何在 WebView 里实现 `@JavascriptInterface` ，让 JavaScript 能够调用原生 Android 的方法。那么，下面让我们在控制台模拟一下调用过程：

1. 首先，连接 Android 设备并启动应用，确保你的 WebView 正在运行。
2. 在电脑上打开 Chrome 浏览器，然后在地址栏中输入 `chrome://inspect`，然后按 Enter 键。然后将鼠标移动到你想要调试的 WebView 页面上，点击“inspect”链接。
3. DevTools 会打开一个新的调试窗口，标签栏中选择 "Console" 标签，那么现在你就可以在控制台输入 JavaScript 代码了。
4. 假设你有一个 Android 方法叫 `showToast()` ，在 Android 代码中已经注入到了 WebView，你可以在控制台模拟调用这个方法：

```js
js复制代码
window.Android.showToast("Hello, Android!");
```

如果一切正常，你应该会在 Android 设备上看到一个 Toast 弹出并显示 "Hello, Android!"。

## 处理 WebView的导航事件

WebView 提供了一些方法来处理网页导航过程中的各种事件，例如：页面开始加载，加载过程中发生错误，页面加载完成等。这些都通过 WebViewClient 来处理。

让我们来看看一些常见的 WebViewClient 的使用场景：

### 基础场景

1. **页面开始加载**： `onPageStarted()` 方法会在 WebView 开始加载一个新的 URL 时被调用。

```kotlin
kotlin复制代码
webView.webViewClient = object : WebViewClient() {

override fun onPageStarted(view: WebView, url: String, favicon: Bitmap?) {

// 页面开始加载时的处理代码

}

}
```

1. **页面加载完成**：`onPageFinished()` 方法会在 WebView 完成 URL 加载时被调用。

```kotlin
kotlin复制代码
webView.webViewClient = object : WebViewClient() {

override fun onPageFinished(view: WebView, url: String) {

// 页面加载完成时的处理代码

}

}
```

1. **加载过程中发生错误**： `onReceivedError()` 方法会在加载页面过程中发生错误时被调用。

```kotlin
kotlin复制代码
webView.webViewClient = object : WebViewClient() {

override fun onReceivedError(view: WebView, request: WebResourceRequest, error: WebResourceError) {

// 加载过程中发生错误时的处理代码

}

}
```

1. **控制新的页面加载**：`shouldOverrideUrlLoading()` 方法会在每次 WebView 尝试加载新的 URL 之前被调用。你可以决定是否中断这次加载，如果你返回 `true`，WebView 就不会加载这个 URL。

```kotlin
kotlin复制代码
webView.webViewClient = object : WebViewClient() {

override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {

// 控制新的页面加载过程的代码

return false

}

}
```

再深入一些,在 WebView 中有一些更高级的导航事件可以用来改善用户体验或者提供更多的功能。

------

### 大图片模式

当你的网页中包含了大图片，你可能希望在图片加载完成后再显示，以避免显示不完全的图片。这可以通过在 WebViewClient 中重写 `onReceivedPicture()` 方法来实现：

```kotlin
kotlin复制代码
webView.webViewClient = object : WebViewClient() {

override fun onReceivedPicture(view: WebView, picture: Picture) {

// 对包含大图片的网页进行特殊处理

}

}
```

这个方法在 WebView 用新的内容重绘自己时被调用。你可以使用这个方法来检测当大图加载完成后重绘 WebView 以提供更好的用户体验。

### 响应网页中的触摸事件

在某些情况下，你可能希望对 WebView 中的触摸事件有一些特殊的反应，比如缩放图片或者播放视频。在 WebView 中，可以通过使用 `setOnTouchListener()` 方法来设置一个监听器，监听 WebView 中的触摸事件：

```kotlin
kotlin复制代码
webView.setOnTouchListener { view, motionEvent ->

// 针对特定的动作进行处理

false

}
```

注意 `setOnTouchListener()` 方法设置的监听器会在所有其他事件处理前接收到触摸事件，如果你的监听器返回 `true`，那么这个事件就不会被 WebView 处理了。

在实践中，我们一般不直接在 WebView 中处理复杂的用户交互（比如手势）。更好的做法是使用 JavaScript 代码在网页中处理这些交互，让 WebView 专注于渲染工作。当然，如果遇到 WebView 不能很好处理的情况，我们可以使用 Android 代码来辅助。

以上只是 WebView 一些更高阶的使用方法，希望对你有所帮助！接下来我们将继续介绍一些 WebView 中的性能和安全问题，你还有其他疑问吗，或者有其他你感兴趣的话题吗？

通常，你需要使用自定义的 WebViewClient 类来控制 WebView 的导航，以处理类似的条件。这只是 WebViewClient 提供的一部分接口，还有很多其他的接口供你使用。

理解和运用 WebView 的导航控制，有助于你管理 WebView 的行为，提高用户体验。

# 处理 WebView 的安全问题

使用 WebView 时，你需要注意一些重要的安全问题。这是因为 WebView 允许你的 app 执行 JavaScript 代码，并能访问 Android 的接口，如果不加以限制，就可能会对用户数据和隐私造成威胁。以下是几点建议来帮助提高你的 WebView 的安全性：

### **限制 JavaScript 访问权限**

如果让 WebView 加载的网页内容不受你的控制，那么就应该限制 WebView 执行 JavaScript 代码。这是因为恶意的 JavaScript 代码可能会被用来制作钓鱼网站或执行恶意的操作。

对于启用了JavaScript的WebView，确实存在一些特定的安全问题。让我们来探索一些主要的安全隐患以及相应的对策。

------

**跨站脚本攻击（XSS）防护具体策略**：

- 使用内容安全策略（Content Security Policy, CSP）：CSP 是一个额外的安全层，用于帮助检测和缓解某些类型的攻击，包括跨站脚本攻击和数据注入攻击。

```html
html复制代码
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

- 对所有的输入进行合适的处理：确保 WebView 加载的内容中所有的用户生成的输入，比如 URI，Cookie，表单数据等都进行了恰当的处理，如编码或者清洁。

1. **跨窗口脚本攻击（XSW）防护具体策略**：

- 限制窗口打开：你可以在 WebViewClient 中重写 `shouldOverrideUrlLoading()`，限制除了当前页面外的其他页面跳转。

```kotlin
kotlin复制代码
webView.webViewClient = object : WebViewClient() {

override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {

// 当链接是目标网页时，返回 false，允许 WebView 加载该页面

if (request?.url.toString() == "https://target.website.com") {

return false

}

// 其他链接过滤掉，不允许 WebView 加载

return true

}

}
```

1. **启用 JavaScript 目标调用防护具体策略**：

- 使用 `@JavascriptInterface` 注解：只有被该注解标记的方法，才能被 WebView 中的 JavaScript 调用。

```kotlin
kotlin复制代码
class WebAppInterface {

@JavascriptInterface fun showToast() {...}

fun sensitiveMethod() {...}

}
```

- 在这个例子中，`showToast()` 方法可以被 JavaScript 调用，而 `sensitiveMethod()` 不能被调用。
- 关闭 JavaScript 功能

这些都是保护启用 JavaScript 的 WebView 的基本步骤。然而，网络安全是一个非常复杂的话题，尤其是涉及到 WebView 这样的复杂组件。

### **限制 WebView 加载的内容**

如果可能的话，应该限制 WebView 加载的 URL。你可以通过重写 WebViewClient 的 `shouldOverrideUrlLoading()` 方法来控制 WebView 加载哪些 URL。否则， WebView 可能会被用来加载恶意网站。

可以做一个最简单的白名单机制，只允许特定的域名加载

```kotlin
kotlin复制代码
val WHITELISTED_DOMAINS = listOf("https://www.allowed1.com", "https://www.allowed2.com", "https://www.allowed3.com")

  


webView.webViewClient = object : WebViewClient() {

  


override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {

// Parse the URL

val url = Uri.parse(request?.url.toString())

  


return WHITELISTED_DOMAINS.none { domain ->

url.host == Uri.parse(domain).host

}

}

}
```

在上面的代码中，我们首先创建了一个白名单，包含了所有允许 WebView 加载的域名。然后我们覆盖了 `shouldOverrideUrlLoading()`，在这个方法里面，我们解析了每一个请求的 URL，如果其域名不在我们的白名单中，我们便返回 `true` 阻止 WebView 加载这个 URL。

### **使用安全的 JavaScript 接口**

如果你提供的 JavaScript 接口能够访问敏感信息（如位置信息，写文件权限等），你需要确保你的 JavaScript 接口设计的足够安全，不能令外部代码访问这些信息。这可以通过使你的接口不暴露敏感操作，或者对调用进行身份验证来实现。

如何安全使用 WebView 的 JavaScript 接口?

------

以下是一些关于安全使用 WebView JavaScript 接口的策略：

1. **最小权限原则**：你应始终遵循最小权限原则，即只公开必要的接口给 WebView 的 JavaScript。这包括限制哪些方法可以被 JavaScript 调用（使用 `@JavascriptInterface` 注解）以及限制这些方法访问的数据。
2. **验证输入**：你的原生方法可能需要接受一些参数。确保你验证了所有的输入参数，这可以防止一些攻击，比如 SQL 注入。

```kotlin
kotlin复制代码
class JsBridge(private val context: Context) {

@JavascriptInterface

fun showToast(message: String) {

if (message.contains("<script>", ignoreCase = true)) {

Toast.makeText(context, "Invalid input", Toast.LENGTH_SHORT).show()

return

}

Toast.makeText(context, message, Toast.LENGTH_SHORT).show()

}

}
```

1. **限制接口使用的上下文**：如果可能，限制 JavaScript Interface 暴露的 Android 上下文。例如，不要传递整个 Activity 实例给 JavaScript Interface，而是传递一个裁剪过的上下文，该上下文只包含 WebView 所需的数据和方法。

```kotlin
kotlin复制代码
class JsBridge(private val liveData: MutableLiveData<String>) {

@JavascriptInterface

fun sendToAndroid(message: String) {

liveData.postValue(message)

}

}
```

1. **采取跨站脚本防护措施**：如前面所述，利用 CSP （内容安全策略），限制 WebView 中的 JavaScript 执行。也考虑使用 `loadData`，`loadDataWithBaseURL` 方法来加载内容，而非 `loadUrl`。
2. **注意 API 等级**：如果你的应用需要支持 Android 4.1 及以下版本，你需要对使用 `addJavascriptInterface` 方法做特别小心。因为在这些版本中，JavaScript 可以访问所有公共方法，包括 `getClass` 方法，这可能用于执行一些恶意操作。

以上都是在使用 WebView 的 JavaScript 接口时应注意的安全性问题及对策，我希望这些可以帮助你。你还有关于这个主题的问题或者其它主题的问题吗？

### **禁用第三方 cookie**

在 WebView 中，默认情况下第三方 cookie 是开启的，这可能会让你的用户暴露在跨站请求伪造（CSRF）攻击之下。你需要通过 `CookieManager` 来禁用第三方 cookie。

### **使用 HTTPS 来加载网页**

尽可能的使你的 WebView 加载 HTTPS 网站，而非 HTTP，因为 HTTPS 网站的通信是加密的，很难被截获。这是一种有效提高 WebView 安全性的方式。

## 使用WebView构建hybrid应用

混合应用程序（Hybrid applications）是原生应用程序和 Web 应用程序的结合。混合应用程序可以使用 WebView 来显示 HTML 和 JavaScript 内容，这些内容可以通过网络加载，也可以嵌入应用程序中。使用 WebView 开发混合应用程序可以带来许多好处，例如复用代码、快速迭代和更新、跨平台兼容等，这就是为什么大家热衷于用它来开发应用。

举一些例子，像 Instagram、Facebook 都使用 WebView 来展示部分内容引以减少原生开发的工作量，提高开发效率。从而可以保证应用在不同平台的一致性体验。

但是要注意 WebView 也有其局限性，它不如原生应用那么强大，特别只有一些复杂、高计算性的操作，原生应用才可以胜任。这个时候，我们在原生和 WebView 之间可能需要做权衡。

### Android TV上的Webview hybrid应用

在 Android TV 平台上创建一个含有原生视图和 WebView 的混合应用同样是可行的，但是 Android TV 平台有一些特殊的要求和限制。例如，不像手机和平板设备那样有触摸屏幕，Android TV 是通过方向键以及确定键等来控制焦点。所以，为了让你的混合应用在 Android TV 平台上良好地运行，你需要确保 WebView 中的 Web 内容以及原生 View 都能正确地响应这些方向键和确定键。

以下是一个 Android TV 用的混合应用的例子：

1. 首先，我们创建一个新的 Android TV 项目，在创建新项目时需要选择 "TV" 作为目标设备。
2. 在布局文件中，我们添加 WebView 和一个 Button。我们可以使用 "android:nextFocusUp"、"android:nextFocusDown" 等属性来控制焦点移动：

```xml
xml复制代码
<RelativeLayout

android:layout_width="match_parent"

android:layout_height="match_parent">

  


<WebView

android:id="@+id/webview"

android:layout_width="match_parent"

android:layout_height="match_parent"

android:nextFocusDown="@+id/button"/>

  


<Button

android:id="@+id/button"

android:layout_width="wrap_content"

android:layout_height="wrap_content"

android:layout_centerInParent="true"

android:text="Click Me!"

android:nextFocusUp="@+id/webview"/>

</RelativeLayout>
```

1. 在我们的 Activity 中找到 WebView，并加载一个网页：

```kotlin
kotlin复制代码
val myWebView: WebView = findViewById(R.id.webview)

val button: Button = findViewById(R.id.button)

  


myWebView.settings.javaScriptEnabled = true

myWebView.webViewClient = WebViewClient()

myWebView.loadUrl("http://example.com")

  


button.setOnClickListener {

// Do something when button is clicked

}
```

以上就是为 Android TV 创建混合应用的基础过程。要注意，这只是一个非常基础的示例，实际应用中你可能需要处理更多的焦点切换问题，还包括需要处理遥控器的其它按键的输入。

## Webview的调试方法

了解 WebView 的调试方法是非常重要的，可以帮助我们定位问题，进行性能优化。以下是一些常见的 WebView 调试方法

1. **启用 WebView 的调试模式**

启用 WebView 的调试模式可以使你方便地使用 Chrome DevTools 对 WebView 内的网页进行检查。你可以查看元素，运行 JavaScript，查看网络请求等等，和在浏览器中一样。

首先，你需要调用 WebView 的 `setWebContentsDebuggingEnabled(true)` 方法启用调试模式。注意，出于安全考虑，你只应在调试模式下启用此选项。

```kotlin
kotlin复制代码
if(BuildConfig.DEBUG){

WebView.setWebContentsDebuggingEnabled(true)

}
```

然后在 Chrome 中输入`chrome://inspect`，你就可以看到你设备上所有可调试的 WebView。

1. 
2. **使用`console.log()`**

你可以在你的网页 JavaScript 代码中使用 console.log() 输出调试信息。当 WebView 的调试模式启用后，这些信息会出现在 Chrome DevTools 的 Console 面板上。

```javascript
javascript复制代码
console.log("Hello, webview!")
```

1. **使用 Android Studio 的 Profiler**

Android Studio 的 Profiler 可以让你查看你的应用的 CPU，内存，网络使用情况，你可以使用它来查看 WebView 的性能影响。

1. **处理 WebViewClient 和 WebChromeClient 的错误和事件**

WebViewClient 和 WebChromeClient 提供了便利的方法来处理各种 WebView 事件，比如页面加载完成，发生错误等等。这些方法非常适合用来调试 WebView。

```kotlin
kotlin复制代码
myWebView.webViewClient = object :WebViewClient() {

override fun onPageStarted(view: WebView?, url: String?, favicon: Bitmap?) {

// 页面开始加载

Log.d(TAG, "页面开始加载: $url")

}

  


override fun onPageFinished(view: WebView?, url: String?) {

// 页面完成加载

Log.d(TAG, "页面完成加载: $url")

}

  


override fun onReceivedError(view: WebView?, request: WebResourceRequest?, error: WebResourceError?) {

// 页面加载错误

Log.d(TAG, "页面加载错误: ${error?.description}")

}

}
```

# Webview性能

## Webview性能度量指标

性能度量指标从广义上说，它可以帮助我们理解我们的应用或者网站在用户的角度看到的性能表现。也就是说，我们可以通过这些度量数据，了解到用户与你的网站或应用交互的速度和流畅度。同时，它也可以帮助开发者找到应用的性能瓶颈，从而进行针对性的优化。

对于Webview来说，我们通常会关注以下几个性能度量指标：

**加载性能**：测量Webview加载内容所需的时间。像第一次加载（First Contentful Paint，FCP）、以及完整内容加载（Dom Content Loaded，DCL）都是我们需要关注的度量点。

- 计算方法: 可以在 WebView 的 WebViewClient 的 onPageStarted 方法 和 onPageFinished 方法中，分别获取开始加载和完成加载的时间，然后用完成时间减去开始时间即可算出加载时间。

**运行性能**：反映出Webview运行的流畅度，包括JS的执行时间，渲染的帧率（FPS）等。这也反映了用户使用Webview时的体验。

**内存性能**：考量Webview对内存的使用情况，内存的使用多少会影响到设备的其他应用的运行情况，从而影响到用户对设备的使用体验。

**网络流量**：如果你的 WebView 需要加载大量的网络资源，那么它可能会占用大量的网络流量。你应尽可能的优化你的资源和网络请求，比如通过使用 WebP 编码的图片来代替 JPEG 或 PNG。

## Webview技术原理

Webview架构图如下

```diff
diff复制代码
+------------------+

| Android |

| WebView |

+----+------------------+-----+

| Java APIs | WebChromeClient |

+-------+----------------+----+

| WebViewCore (Bridge) |

+--------+----------------+-------+

| Layout Engine (WebCore) | JavaScript Engine (V8) |

+------------------------+---------+

| WebKit |

+-----------------------------------------------+
```

### WebView 中关键的组件和工作原理

#### WebView 控件

这是我们直接在应用中操作的部分。主要负责绘制网页内容、处理用户输入和与应用的其他部分交互。其中，一些主要的技术细节包括：

1. **事件处理**：WebView 控件负责接收并处理所有的用户输入事件，例如触摸、滑动和键盘输入。然后，这些事件会被转换为对应的 DOM 事件（例如：点击、滚动等），再由 WebViewCore 转发给 WebKit 引擎。
2. **绘制**：WebView 控件重写了 onDraw() 方法来绘制网页内容。实际的绘制工作是由底层的 WebKit 引擎完成的，然后 WebView 通过调用 native 方法来把渲染结果画到屏幕上。

------

#### WebViewCore

作为 WebView 控件和底层 WebKit 引擎之间的桥梁，WebViewCore 的主要工作是确保两者之间的通信顺畅。以下是 WebViewCore 的一些主要任务：

1. **转发事件**：WebViewCore 转发来自 WebView 控件的用户输入事件到底层的 WebKit 引擎。同时，它也负责把来自 WebKit 引擎的回调事件通知给 WebView 控件。
2. **数据交换**：WebViewCore 会与底层的 WebKit 引擎进行数据交换。例如，当 WebView 控件调用 loadUrl() 方法加载网页时，WebViewCore 会把这个请求传递给 WebKit 引擎。

------

#### WebKit 引擎

WebKit 引擎是 WebView 内部的核心组件，它负责解析和渲染 HTML、CSS 和 JavaScript 内容。WebKit 引擎分为两个部分：WebCore 和 JavaScriptCore。

1. **WebCore**：WebCore 是 WebKit 引擎解析 HTML 和 CSS 代码并进行页面布局和渲染的部分。
2. **JavaScriptCore**：JavaScriptCore 是 WebKit 引擎解析和执行 JavaScript 代码的部分。它有一个称为 JSValue 的对象，这个对象表示 JavaScript 中的一个值，这个值可以是原始类型（例如字符串、数字或者布尔值）或是一个复杂类型的对象。

### WebView的启动过程

WebView的启动可以分为以下几个阶段：

1. **实例化 WebView 控件**：这一步在你的 Activity 或者 Fragment 的 onCreate 方法中执行，通常是通过布局文件来实例化 WebView。

```kotlin
kotlin复制代码
val myWebView: WebView = findViewById(R.id.webview)
```

1. **创建 WebViewCore**：当 WebView 控件被实例化后，就会调用 WebView 的私有方法 createWebViewCore() 创建一个 WebViewCore 对象。WebViewCore 的主要任务是将 WebView 的请求传递给底层的 WebKit 引擎。
2. **实例化 WebKit**：这一步在 WebViewCore 的构造方法中完成。Webkit 是一个开源的浏览器引擎，负责解析 HTML，CSS 和 JavaScript，并渲染页面。
3. **设置 WebView 的客户端**：默认情况下，WebView 会使用内置的 WebViewClient 和 WebChromeClient，你也可以提供自己的实现版本。

```kotlin

myWebView.webViewClient = WebViewClient()

myWebView.webChromeClient = WebChromeClient()
```

1. **加载 URL**：通过调用 `loadUrl()` 方法加载一个网页。开始装载 URL 后，WebViewCore 会创建一个新的 UI 线程请求。在加载 URL 的过程中要下载文档，解析 HTML 并渲染页面，这个过程可能需要等待 IO 操作，并且需要大量的计算。

```kotlin

myWebView.loadUrl("http://www.example.com")
```

1. **渲染页面**：下载完成后，WebKit 会解析 HTML 文档生成 DOM 树，然后解析 CSS 和运行 JavaScript。最后，生成的渲染树会被绘制到屏幕。
2. **交互**：页面完成渲染后，用户可以与页面进行交互，如滚动页面、点击链接等。在这个过程中，WebView会处理用户的输入事件，如触摸事件，并可能触发 JavaScript 的事件监听器。

每一步都有其具体的工作，组合起来就形成了 WebView 的启动过程。





# Q&A





# 参考

1、https://juejin.cn/post/7322229620391510068

[Carson带你学Android：最全面、易懂的Webview使用教程](https://www.jianshu.com/p/3c94ae673e2a)

[Carson带你学Android：全面总结WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)

[Carson带你学Android：这是一份全面&详细的WebView学习攻略](https://www.jianshu.com/p/d2d4f652029d)
