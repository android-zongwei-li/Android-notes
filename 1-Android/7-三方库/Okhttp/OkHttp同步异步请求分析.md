[okhttp项目地址](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsquare%2Fokhttp)

基于：3.14.9

# 同步请求

```cpp
 //  构建okHttpClient，相当于请求的客户端，Builder设计模式
 OkHttpClient okHttpClient = new OkHttpClient.Builder().readTimeout(5, TimeUnit.SECONDS).build();
        // 构建一个请求体，同样也是Builder设计模式
        Request request = new Request.Builder().url("http://www.baidu.com").build();
        //  生成一个Call对象，该对象是接口类型
        Call call = okHttpClient.newCall(request);
        try {
            //  拿到Response
            Response response = call.execute();
            Log.i("TAG",response.body().string());
        } catch (IOException e) {
            
        }
```

同步请求步骤：

1. 通过Builder模式创建OkHttpClient对象和Request对象
2. 调用OkHttpClient的newCall方法，获取一个Call对象，参数是Request
3. 调用execute方法获取一个Respone

### （一） OkHttpClient源码分析

```java
public static final class Builder {
    Dispatcher dispatcher;
    ...
    ...
    public Builder() {
      dispatcher = new Dispatcher();
      protocols = DEFAULT_PROTOCOLS;
      connectionSpecs = DEFAULT_CONNECTION_SPECS;
      eventListenerFactory = EventListener.factory(EventListener.NONE);
      proxySelector = ProxySelector.getDefault();
      if (proxySelector == null) {
        proxySelector = new NullProxySelector();
      }
      cookieJar = CookieJar.NO_COOKIES;
      socketFactory = SocketFactory.getDefault();
      hostnameVerifier = OkHostnameVerifier.INSTANCE;
      certificatePinner = CertificatePinner.DEFAULT;
      proxyAuthenticator = Authenticator.NONE;
      authenticator = Authenticator.NONE;
      connectionPool = new ConnectionPool();
      dns = Dns.SYSTEM;
      followSslRedirects = true;
      followRedirects = true;
      retryOnConnectionFailure = true;
      callTimeout = 0;
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
      pingInterval = 0;
    }
   
}

 public ConnectionPool(int maxIdleConnections, long keepAliveDuration, TimeUnit timeUnit) {
    this.maxIdleConnections = maxIdleConnections;
    this.keepAliveDurationNs = timeUnit.toNanos(keepAliveDuration);

    // Put a floor on the keep alive duration, otherwise cleanup will spin loop.
    if (keepAliveDuration <= 0) {
      throw new IllegalArgumentException("keepAliveDuration <= 0: " + keepAliveDuration);
    }
  }
```

Builder是OkHttpClient一个静态内部类，在Builder的构造函数中进行了一系列的初始化操作，其中**Dispatcher是分发器的意思，和拦截器不同的是分发器不做事件处理，只做事件流向，负责将每一次Requst进行分发，压栈到自己的线程池，并通过调用者自己不同的方式进行异步和同步处理**。ConnectionPool是一个连接池对象，它可以用来管理连接对象，从它的构造方法中可以看到连接池的默认空闲连接数为5个，keepAlive时间为5分钟。

### （二） Request源码分析

```java
Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tags = Util.immutableMap(builder.tags);
}

public static class Builder {
    @Nullable HttpUrl url;
    String method;
    Headers.Builder headers;
    @Nullable RequestBody body;

    public Builder() {
      this.method = "GET";
      this.headers = new Headers.Builder();
    }
  
    public Request build() {
      if (url == null) throw new IllegalStateException("url == null");
      return new Request(this);
    }
}
```

Request.Builder()构造函数中设置了默认的请求方法是GET方法，Request类的build()方法是用来创建一个Request对象，将当前的Builder对象传进去，并完成了对象的赋值。将Builder类的相关属性赋值给Request的相关属性，这也是Builder模式的精髓。

### （三） Call对象的创建：newCall()执行分析

```kotlin
@Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false);
  }
```

调用了RealCall的newRealCall方法，Call是一个接口，RealCall是它的实现类

```java
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
  }
```

在newRealCall方法里，完成了RealCall对象的创建，并把它返回出去。至此，Call对象已经创建完毕，**实际上创建的对象是Call的实现类RealCall 对象。**

### （四） Response对象的创建： call.execute()执行分析

Call call = okHttpClient.newCall(request) 拿到的是 RealCall 对象，所以直接看 RealCall 类的 execute()

```kotlin
@Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    timeout.enter();
    eventListener.callStart(this);
    try {
      // 分析1
      client.dispatcher().executed(this);
      // 分析2
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      e = timeoutExit(e);
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      // 分析3
      client.dispatcher().finished(this);
    }
  }
```

在同步代码块中，首先判断excuted是不是true，它的含义是否有在执行，如果是，抛出异常，如果没有执行过，将excuted置为true。eventListener.callStart(this)开启事件监听，eventListener在RealCall对象创建的时候，也一起创建了。

execute()的核心方法: 

- client.dispatcher().executed(this)
- Response result = getResponseWithInterceptorChain()
- client.dispatcher().finished(this)

分析1：dispatcher().executed(this)

```csharp
public Dispatcher dispatcher() {
    return dispatcher;
  }

  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }
```

首先调用了Dispatcher的dispatcher() 获取 Dispatcher 对象，紧接着调用Dispatcher的executed方法，往runningSyncCalls对象中添加了一个 call 对象，runningSyncCalls是一个存放同步请求的队列，Dispatcher类中维护了3种类型的请求队列：

```php
  /** Ready async calls in the order they'll be run. */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
```

- **readyAsyncCalls** 是异步请求的就绪队列
- **runningAsyncCalls** 是异步请求的执行队列
- **runningSyncCalls** 是同步请求的执行队列

分析2：getResponseWithInterceptorChain()

调用完Dispatcher的execute()方法后，紧接着调用了getResponseWithInterceptorChain()方法。

```csharp
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }
```

该方法返回一个Response对象。在方法中创建了 interceptors 集合，添加了数个拦截器对象，**此处使用了责任链设计模式**，创建了 Interceptor.Chain 拦截器链，并执行proceed处理请求，后续可依次在拦截器中做相应的操作。

分析3：dispatcher().finished(this)

在Response的excute方法的finally模块中，调用了 client.dispatcher().finished(this)：

```cpp
void finished(RealCall call) {
    finished(runningSyncCalls, call);
  }
```

finished方法内部调用了它的重载方法，并把**同步请求的消息队列对象和RealCall对象**传过去：

```java
private <T> void finished(Deque<T> calls, T call) {
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      idleCallback = this.idleCallback;
    }

    boolean isRunning = promoteAndExecute();

    if (!isRunning && idleCallback != null) {
      idleCallback.run();
    }
  }
```

在同步代码快中，将call对象从同步请求消息队列中移除。紧接着调用了promoteAndExecute方法：

```csharp
 private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();

        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.

        i.remove();
        asyncCall.callsPerHost().incrementAndGet();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
      isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      asyncCall.executeOn(executorService());
    }

    return isRunning;
  }
```

同步代码块有一个for循环，去迭代readyAsyncCalls，也就是待准备消息队列。但是从前面一步步分析过来，并没有往readyAsyncCalls添加过数据，所以当前的for循环并不会执行，之后的一个for循环也不会执行，isRunning返回false。

promoteAndExecute()方法返回false。

```java
private <T> void finished(Deque<T> calls, T call) {
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      idleCallback = this.idleCallback;
    }

    boolean isRunning = promoteAndExecute();

    if (!isRunning && idleCallback != null) {
      idleCallback.run();
    }
  }
```

并且idleCallback 已经完成了初始化，所以会执行 idleCallback的run()方法

## 总结

在同步请求中Dispatcher主要负责了两件事，同步请求的保存和移除。

## 异步请求

```java
 //  构建okHttpClient，相当于请求的客户端，Builder设计模式
 OkHttpClient okHttpClient = new OkHttpClient.Builder().readTimeout(5, TimeUnit.SECONDS).build();
        // 构建一个请求体，同样也是Builder设计模式
        Request request = new Request.Builder().url("http://www.baidu.com").build();
        //  生成一个Call对象，该对象是接口类型，后面会说
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
                    @Override
                    public void onFailure(Call call, IOException e) {
                        
                    }

                    @Override
                    public void onResponse(Call call, Response response) throws IOException {

                    }
                });
```

简单的看下异步请求的几个步骤：

##### 1.通过Builder模式创建OkHttpClient对象和Request对象

##### 2.调用OkHttpClient的newCall方法，获取一个Call对象，参数是Request

##### 3.调用call对象的enqueue()方法

步骤1和步骤2跟同步请求的步骤一致，主要看一下步骤3

### 异步请求源码：call.enqueue() 源码分析

异步请求和同步请求的步骤1和步骤2一致，都是在做准备工作，并没有发起请求，所以这次我们直接忽略了步骤1和步骤2的分析，直接分析步骤3的源码，我们点开RealCall的enqueue方法：



```java
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```

**我们看看它的核心代码： client.dispatcher().enqueue(new AsyncCall(responseCallback))，**
client.dispatcher()返回一个Dispatcher对象没什么可讲的，紧接着调用Dispatcher的enqueue方法，参数是AsyncCall对象，我们先看看AsyncCall是什么？



```java
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;
    private volatile AtomicInteger callsPerHost = new AtomicInteger(0);

    AsyncCall(Callback responseCallback) {
      super("OkHttp %s", redactedUrl());
      this.responseCallback = responseCallback;
    }

    AtomicInteger callsPerHost() {
      return callsPerHost;
    }
  
  }
```

只截取了部分代码，该类继承自NamedRunnable,我们看看NamedRunnable：



```java
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

NamedRunnable 实现了Runnable 接口。

我们直接去Dispatcher的enqueue()看看做了哪些操作。



```csharp
void enqueue(AsyncCall call) {
   synchronized (this) {
     readyAsyncCalls.add(call);

     // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
     // the same host.
     if (!call.get().forWebSocket) {
       AsyncCall existingCall = findExistingCallWithHost(call.host());
       if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
     }
   }
   promoteAndExecute();
 }
```

在同步代码块中，将当前的call请求添加到**待准备消息队列**中去，**注意这里跟同步请求的区别，同步请求的时候，并没有把当前的call添加到准备消息队列中去。**然后又调用了 promoteAndExecute()方法，同步请求的时候也调用了promoteAndExecute()方法



```csharp
private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();

        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.

        i.remove();
        asyncCall.callsPerHost().incrementAndGet();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
      isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      asyncCall.executeOn(executorService());
    }

    return isRunning;
  }
```

此时,readyAsyncCalls不为空了，我们单独的把这个for循环拎出来讲：



```csharp
for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();

        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.

        i.remove();
        asyncCall.callsPerHost().incrementAndGet();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
```

这段代码的含义是：**把符合条件的call请求从readyAsyncCalls提升为runningAsyncCalls，**我们看这段代码中的两个if语句，**第一个if语句判断当前异步请求的执行队列长度大于等于请求最大值，如果满足直接跳出for循环，maxRequests的值为64，第二个if语句判断当前执行的异步请求队列中相同主机的请求数是否大于等于maxRequestsPerHost（每个主机最大请求数，默认为5）**，如果这两个条件都不满足的情况下，把从readyAsyncCalls取出来的call请求，存到临时的
executableCalls 队列中去。

紧接着去遍历executableCalls：



```csharp
  for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      asyncCall.executeOn(executorService());
    }
```

从executableCalls获取AsyncCall对象，并且调用它的executeOn方法，executeOn()方法参数是executorService()，我们看看executorService():



```java
 public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
```

可以看出，该方法是一个同步方法，返回的是一个线程池对象，ThreadPoolExecutor()的第二个参数传入了Integer的最大值，即线程池所能容纳的最大线程数为Integer.MAX_VALUE，虽然这里设置了很大的值，但是实际情况下并非会达到最大值，因为上面enqueue()方法中有做了判断。

回到 asyncCall.executeOn(executorService())这里，executorService返回了一个线程池对象，紧接着调用线程池对象的execute方法，execute()方法传入**实现了Runable接口的AsyncCall对象**，前面在分析同步请求的时候，说了AsyncCall实现了Runable接口

![img](https:////upload-images.jianshu.io/upload_images/8744053-54d65ef22289664a.png?imageMogr2/auto-orient/strip|imageView2/2/w/969)

实现Runnable.png



ok,现在我们要看看线程池做了什么操作，直接去NamedRunnable的run方法看看做了什么操作。



```java
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

**execute()是一个抽象方法，所以我们直接去NamedRunnable的实现类AsyncCall的execute()方法看：**



```kotlin
@Override protected void execute() {
      boolean signalledCallback = false;
      timeout.enter();
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        e = timeoutExit(e);
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
```

**这段代码才是真正执行异步请求的逻辑**，getResponseWithInterceptorChain()返回Response对象，然后判断retryAndFollowUpInterceptor是否取消回调CallBack接口的onFailure()或onResponse()方法，最后finally中，和同步请求的处理一样，调用了Dispatcher对象的finished()方法。



```java
void finished(RealCall call) {
    finished(runningSyncCalls, call);
  }

  private <T> void finished(Deque<T> calls, T call) {
    Runnable idleCallback;
    synchronized (this) {
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      idleCallback = this.idleCallback;
    }

    boolean isRunning = promoteAndExecute();

    if (!isRunning && idleCallback != null) {
      idleCallback.run();
    }
  }
```

看完promoteAndExecute()方法的具体操作，我们发现，调用eneque的时候会把call请求添加到readyAsyncCalls(异步请求准备队列)中，而readyAsyncCalls队列中的请求什么时候执行呢，看完promoteAndExecute的代码就恍然大悟了。

## 总结：异步请求中，我们先去遍历**异步请求的就绪队列**，并判断异步请求的执行队列的队列大小是否小于设置的最大数的时候，如果条件满足，把该请求添加到异步请求的执行队列中去，同时把该请求添加到临时的异步请求的执行队列去中。之后，遍历这个临时的异步请求的执行队列，去执行AsyncCall的execute()方法。

![img](https:////upload-images.jianshu.io/upload_images/8744053-361d2a70b8c1b9aa.png?imageMogr2/auto-orient/strip|imageView2/2/w/1126)

# 参考

[盘它，okhttp3吊炸天源码分析(一)：同步、异步请求源码解析](https://www.jianshu.com/p/5bb1709f3729)