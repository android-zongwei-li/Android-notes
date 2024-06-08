# 一、简介

官网：https://square.github.io/okhttp/

Github：https://github.com/square/okhttp

OkHttp 是一个网络请求框架。它有以下默认特性：

允许连接到同一个主机地址的所有请求,提高请求效率
共享Socket,减少对服务器的请求次数
通过连接池,减少了请求延迟
缓存响应数据来减少重复的网络请求
减少了对数据流量的消耗
自动处理GZip压缩

功能：

get,post请求
文件的上传和下载
加载图片
支持请求回调，直接返回对象、对象集合
支持session的保持

# OkHttp基本使用

## 引入依赖

```kotlin
implementation("com.squareup.okhttp3:okhttp:3.14.9")
// 下面这个库是用Kotlin实现的，还没试过
implementation("com.squareup.okhttp3:okhttp:4.12.0")
```

添加网络权限

```kotlin
<uses-permission android:name="android.permission.INTERNET" />
```

[一个请求测试服务器](https://httpbin.org)

## GET

### 异步GET请求

使用步骤：
创建OkHttpClient对象
构造Request对象
通过OkHttpClient和Request对象来构建Call对象
通过Call对象的enqueue(Callback)方法来执行异步请求

```kotlin
    fun asyncGet() {
        val url = "http://wwww.baidu.com"
        val okHttpClient = OkHttpClient()
        val request: Request = Request.Builder().url(url).get() //默认就是GET请求，可以省略
                .build()
        val call: Call = okHttpClient.newCall(request)
        call.enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                Log.d(TAG, "onFailure: ")
            }
            
            @Throws(IOException::class)
            override fun onResponse(call: Call, response: Response) {
                //response.body().string() 获得服务器返回的数据
                Log.d(TAG, "onResponse: " + response.body()?.string())
            }
        })
    }
```

### 同步GET请求

前面几个步骤和异步方式一样，只是最后一步是通过 Call.execute() 来提交请求，注意这种方式会阻塞调用线程，所以在Android中应放在子线程中执行，否则有可能引起ANR异常，Android3.0 以后已经不允许在主线程访问网络。

```kotlin
    fun syncGet() {
        val url = "http://wwww.baidu.com"
        val okHttpClient = OkHttpClient()
        val request: Request = Request.Builder().url(url).build()
        val call = okHttpClient.newCall(request)
        lifecycleScope.launch {
            withContext(Dispatchers.IO) {
                try {
                    val response = call.execute()
                    Log.d(TAG, "run: " + response.body()!!.string())
                } catch (e: IOException) {
                    e.printStackTrace()
                }
            }
        }
    }
```

## POST

### POST请求（提交String）

这种方式与前面的区别就是在构造Request对象时，需要多构造一个RequestBody对象，用它来携带我们要提交的数据。在构造 RequestBody 需要指定MediaType，用于描述请求/响应 body 的内容类型。
MediaType指的是要传递的数据的MIME类型，MediaType对象包含了三种信息：type 、subtype以及charset，一般我们会将这些信息传入parse()方法中，这样就可以解析出MediaType对象。如果不知道某种类型数据的MIME类型，可以参见MIME参考手册 较详细的列出了所有数据的MIME类型。

使用步骤：
创建OkHttpClient对象
构造mediaType对象
构造RequestBody对象
通过RequestBody构造Request对象
通过OkHttpClient和Request对象来构建Call对象
通过Call对象的enqueue(Callback)方法来执行异步请求

```kotlin
    fun postString() {
        val url = "http://wwww.baidu.com"
        val client = OkHttpClient()
        val mediaType = MediaType.parse("text/x-markdown; charset=utf-8")
        val requestBody = RequestBody.create(mediaType, "RequestBody")
        val request: Request = Request.Builder()
                .url(url)
                .post(requestBody)
                .build()
        val call = client.newCall(request)
        call.enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                Log.d(TAG, "onFailure: ")
            }
            
            @Throws(IOException::class)
            override fun onResponse(call: Call, response: Response) {
                Log.d(TAG, "onResponse: " + response.body()!!.string())
            }
        })
    }
```

### POST请求（提交表单 key-value参数）

向服务器提交表单时，使用 RequestBody 的实现类FormBody来描述请求体，它可以携带一些经过编码的 key-value 请求体，FromBody用于提交表单键值对(key-value),其作用类似于HTML中的< form >标记。

```kotlin
    fun postKeyValue(userName: String, password: String) {
        val url = "http://wwww.baidu.com"
        val client = OkHttpClient()
        val requestBody: RequestBody = FormBody.Builder()
                .add("username", userName)
                .add("password", password)
                .build()
        val request: Request = Request.Builder()
                .url(url)
                .post(requestBody)
                .build()
        val call = client.newCall(request)
        call.enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                Log.d(TAG, "onFailure: ")
            }
            
            @Throws(IOException::class)
            override fun onResponse(call: Call, response: Response) {
                Log.d(TAG, "onResponse: " + response.body()!!.string())
            }
        })
    }
```

### POST请求（提交json数据）

使用步骤与POST提交String相同，唯一的区别就是mediaType对象要解析的MIME类型为（“application/json;charset=utf-8”）。

```kotlin
    fun postJson(jsonData: String) {
        val url = "http://wwww.baidu.com"
        val client = OkHttpClient()
        val mediaType = MediaType.parse("application/json;charset=utf-8")
        val requestBody: RequestBody = RequestBody.create(mediaType, jsonData)
        val request: Request = Request.Builder()
                .url(url)
                .post(requestBody)
                .build()
        val call = client.newCall(request)
        call.enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                Log.d(TAG, "onFailure: ")
            }
            
            @Throws(IOException::class)
            override fun onResponse(call: Call, response: Response) {
                Log.d(TAG, "onResponse: " + response.body()!!.string())
            }
        })
    }
```

# 自定义拦截器

```kotlin
class ParamsInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
	      Log.d("custom intercept", request.toString())
        // 业务逻辑，对 request 作处理
        
        val response = chain.proceed(request)
     		Log.d("custom intercept", response.toString())
        // 业务逻辑，对 response 作处理
        return response
    }
}
```

## 使用自定义拦截器

```kotlin
val okHttpClient = OkHttpClient.Builder()
                .addInterceptor(ParamsInterceptor())
                .build()
```

## 使用场景

- 打印日志
- 添加公共请求参数

# 核心Api

## Request



## Response

# 参考

[OkHttp基本使用教程](https://blog.csdn.net/weixin_45648614/article/details/104381825)
