> version：2022/06/22
>
> review：



目录

[TOC]



# 一、概述

`Service` 是一种可在后台执行长时间运行操作而不提供界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。此外，组件可通过绑定到服务与之进行交互，甚至是执行进程间通信 (IPC)。例如，服务可在后台处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序进行交互。



以下是三种不同的服务类型：

- 前台

  前台服务执行一些用户能注意到的操作。例如，音频应用会使用前台服务来播放音频曲目。前台服务必须显示[通知](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications?hl=zh-cn)。即使用户停止与应用的交互，前台服务仍会继续运行。

- 后台

  后台服务执行用户不会直接注意到的操作。例如，如果应用使用某个服务来压缩其存储空间，则此服务通常是后台服务。

  > 注意：如果您的应用面向 API 级别 26 或更高版本，当应用本身未在前台运行时，系统会对[运行后台服务施加限制](https://developer.android.google.cn/about/versions/oreo/background?hl=zh-cn)。在诸如此类的大多数情况下，您的应用应改为使用[计划作业](https://developer.android.google.cn/topic/performance/scheduling?hl=zh-cn)。

- 绑定

  当应用组件通过调用 `bindService()` 绑定到服务时，服务即处于*绑定*状态。绑定服务会提供客户端-服务器接口，以便组件与服务进行交互、发送请求、接收结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。仅当与另一个应用组件绑定时，绑定服务才会运行。多个组件可同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。

服务可同时以两种方式（启动服务和绑定服务）运行，它既可以是启动服务（以无限期运行），亦支持绑定。唯一的问题在于您是否实现一组回调方法：`onStartCommand()`（让组件启动服务）和 `onBind()`（实现服务绑定）。

无论服务是处于启动状态还是绑定状态（或同时处于这两种状态），任何应用组件均可像使用 Activity 那样，通过调用 `Intent` 来使用服务（即使此服务来自另一应用）。不过，可以通过清单文件将服务声明为*私有*服务，并阻止其他应用访问该服务。[使用清单文件声明服务](https://developer.android.google.cn/guide/components/services?hl=zh-cn#Declaring)部分将对此做更详尽的阐述。

> **注意：**服务在其托管进程的主线程中运行，它既**不**创建自己的线程，也**不**在单独的进程中运行（除非另行指定）。如果服务将执行任何 CPU 密集型工作或阻止性操作（例如 MP3 播放或联网），则应通过在服务内创建新线程来完成这项工作。通过使用单独的线程，可以降低发生“应用无响应”(ANR) 错误的风险，而应用的主线程仍可继续专注于运行用户与 Activity 之间的交互。



# 二、在服务和线程之间进行选择

简单地说，服务是一种即使用户未与应用交互也可在后台运行的组件，因此，只有在需要服务时才应创建服务。

如果必须在主线程之外执行操作，但只在用户与应用交互时才执行此操作，则应创建新线程。例如，如果只是想在 Activity 运行的同时播放一些音乐，则可在 `onCreate()` 中创建线程，在 `onStart()` 中启动线程运行，然后在 `onStop()` 中停止线程。还可考虑使用 `AsyncTask` 或 `HandlerThread`，而非传统的 `Thread` 类。如需了解有关线程的详细信息，请参阅[进程和线程](https://developer.android.google.cn/guide/components/processes-and-threads?hl=zh-cn#Threads)文档。

请记住，如果确实要使用服务，则默认情况下，它仍会在应用的主线程中运行，因此，如果服务执行的是密集型或阻止性操作，则仍应在服务内创建新线程。

> 总结：如果执行不需要与用户交互的操作，可以考虑服务；要与用户交互的耗时操作，考虑用线程。



# 三、基础知识

要创建服务，必须创建 `Service` 的子类（或使用它的一个现有子类）。在实现中，必须重写一些回调方法，从而处理服务生命周期的某些关键方面，并提供一种机制将组件绑定到服务（如适用）。以下是应该重写的最重要的回调方法：

- `onStartCommand()`

  当另一个组件（如 Activity）请求启动服务时，系统会通过调用 `startService()` 来调用此方法。执行此方法时，服务即会启动并可在后台无限期运行。如果实现此方法，要在服务工作完成后，通过调用 `stopSelf()` 或 `stopService()` 来停止服务。（如果只想提供绑定，则无需实现此方法。）

- `onBind()`

  当另一个组件想要与服务绑定（例如执行 RPC）时，系统会通过调用 `bindService()` 来调用此方法。在此方法的实现中，您必须通过返回 `IBinder` 提供一个接口，以供客户端用来与服务进行通信。请务必实现此方法；但是，如果您并不希望允许绑定，则应返回 null。

- `onCreate()`

  首次创建服务时，系统会（在调用 `onStartCommand()` 或 `onBind()` 之前）调用此方法来执行一次性设置程序。如果服务已在运行，则不会调用此方法。

- `onDestroy()`

  当不再使用服务且准备将其销毁时，系统会调用此方法。服务应通过实现此方法来清理任何资源，如线程、注册的侦听器、接收器等。这是服务接收的最后一个调用。

如果组件通过调用 `startService()` 启动服务（这会引起对 `onStartCommand()` 的调用），则服务会一直运行，直到其使用 `stopSelf()` 自行停止运行，或由其他组件通过调用 `stopService()` 将其停止为止。

如果组件通过调用 `bindService()` 来创建服务，且*未*调用 `onStartCommand()`，则服务只会在该组件与其绑定时运行。当该服务与其所有组件取消绑定后，系统便会将其销毁。

**销毁机制**

只有在内存过低且必须回收系统资源以供拥有用户焦点的 Activity 使用时，Android 系统才会停止服务。如果将服务绑定到拥有用户焦点的 Activity，则它不太可能会终止；如果将服务声明为[在前台运行](https://developer.android.google.cn/guide/components/services?hl=zh-cn#Foreground)，则其几乎永远不会终止。

**重启机制**

如果服务已启动并长时间运行，则系统逐渐降低其在后台任务列表中的位置，而服务被终止的概率也会大幅提升—如果服务是启动服务，则您必须将其设计为能够妥善处理系统执行的重启。如果系统终止服务，则其会在资源可用时立即重启服务，但这还取决于您从 `onStartCommand()` 返回的值。如需了解有关系统会在何时销毁服务的详细信息，请参阅[进程和线程](https://developer.android.google.cn/guide/components/processes-and-threads?hl=zh-cn)文档。

> mynote：

下文将介绍如何创建 `startService()` 和 `bindService()` 服务方法，以及如何通过其他应用组件使用这些方法。

# 四、创建服务

**使用清单文件声明服务**

如同对 Activity 及其他组件的操作一样，您必须在应用的清单文件中声明所有服务。

如要声明服务，请添加 [service](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn) 元素作为 [application](https://developer.android.google.cn/guide/topics/manifest/application-element?hl=zh-cn) 元素的子元素。下面是示例：

```xml
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
```

如需了解有关使用清单文件声明服务的详细信息，请参阅 [service](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn) 元素参考文档。

还可在 [service](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn) 元素中加入其他属性，以定义一些特性，如启动服务及其运行时所在进程需要的权限。[`android:name`](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn#nm) 属性是唯一必需的属性，用于指定服务的类名。发布应用后，请保此类名不变，以避免因依赖显式 Intent 来启动或绑定服务而破坏代码的风险（请阅读博文 [Things That Cannot Change](http://android-developers.blogspot.com/2011/06/things-that-cannot-change.html) [不能更改的内容]）。

> **注意**：为确保应用的安全性，在启动 `Service` 时，请始终使用显式 Intent，且不要为服务声明 Intent 过滤器。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务会响应 Intent，而用户也无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 `bindService()`，则系统会抛出异常。

可以通过添加 [`android:exported`](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn#exported) 属性并将其设置为 `false`，确保服务仅适用于您的应用。这可以有效阻止其他应用启动您的服务，即便在使用显式 Intent 时也如此。

> **注意**：用户可以查看其设备上正在运行的服务。如果他们发现自己无法识别或信任的服务，则可以停止该服务。为避免用户意外停止您的服务，您需要在应用清单的 [service](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn) 元素中添加 [`android:description`](https://developer.android.google.cn/guide/topics/manifest/service-element?hl=zh-cn#desc)。请在描述中用一个短句解释服务的作用及其提供的好处。



## 1、创建启动服务

启动服务由另一个组件通过调用 `startService()` 启动，这会导致调用服务的 `onStartCommand()` 方法。

服务启动后，其生命周期即独立于启动它的组件。即使系统已销毁启动服务的组件，该服务仍可在后台无限期地运行。因此，服务应在其工作完成时通过调用 `stopSelf()` 来自行停止运行，或者由另一个组件通过调用 `stopService()` 来将其停止。

应用组件（如 Activity）可通过调用 `startService()` 方法并传递 `Intent` 对象（指定服务并包含待使用服务的所有数据）来启动服务。服务会在 `onStartCommand()` 方法接收此 `Intent`。

例如，假设某 Activity 需要将一些数据保存到在线数据库中。该 Activity 可以启动一个协同服务，并通过向 `startService()` 传递一个 Intent，为该服务提供要保存的数据。服务会通过 `onStartCommand()` 接收 Intent，连接到互联网并执行数据库事务。事务完成后，服务将自行停止并销毁。

> **注意：**默认情况下，服务与服务声明所在的应用运行于同一进程，并且运行于该应用的主线程中。如果服务在与来自同一应用的 Activity 进行交互时执行密集型或阻止性（耗时）操作，则会降低 Activity 性能。为避免影响应用性能，请在服务内启动新线程。



通常，您可以扩展两个类来创建启动服务：

- `Service`

  这是适用于所有服务的基类。扩展此类时，您必须创建用于执行所有服务工作的新线程，因为服务默认使用应用的主线程，这会降低应用正在运行的任何 Activity 的性能。

- `IntentService`

  这是 `Service` 的子类，其使用工作线程（子线程）**逐一处理**所有启动请求。如果您不要求服务同时处理多个请求，此类为最佳选择。实现 `onHandleIntent()`，该方法会接收每个启动请求的 Intent，以便您执行后台工作。

下文介绍如何使用其中任一类来实现服务。

### 扩展 IntentService 类

由于大多数启动服务无需同时处理多个请求（实际上，这种多线程情况可能很危险），因此最佳选择是利用 `IntentService` 类实现服务。

`IntentService` 类会执行以下操作：

- 创建默认的工作线程，用于在应用的主线程外执行传递给 `onStartCommand()` 的所有 Intent。
- 创建工作队列，用于将 Intent 逐一传递给 `onHandleIntent()` 实现，这样您就永远不必担心多线程问题。
- 在处理完所有启动请求后停止服务，因此您永远不必调用 `stopSelf()`。
- 提供 `onBind()` 的默认实现（返回 null）。
- 提供 `onStartCommand()` 的默认实现，可将 Intent 依次发送到工作队列和 `onHandleIntent()` 实现。

如要完成客户端提供的工作，请实现 `onHandleIntent()`。不过，您还需为服务提供小型构造函数。

以下是 `IntentService` 的实现示例：

```kotlin
/**
 * A constructor is required, and must call the super [android.app.IntentService.IntentService]
 * constructor with a name for the worker thread.
 */
class HelloIntentService : IntentService("HelloIntentService") {

    /**
     * The IntentService calls this method from the default worker thread with
     * the intent that started the service. When this method returns, IntentService
     * stops the service, as appropriate.
     */
    override fun onHandleIntent(intent: Intent?) {
        // Normally we would do some work here, like download a file.
        // For our sample, we just sleep for 5 seconds.
        try {
            Thread.sleep(5000)
        } catch (e: InterruptedException) {
            // Restore interrupt status.
            Thread.currentThread().interrupt()
        }

    }
}
```

您只需要一个构造函数和一个 `onHandleIntent()` 实现即可。

如果您还决定重写其他回调方法（如 `onCreate()`、`onStartCommand()` 或 `onDestroy()`），请确保调用超类实现，以便 `IntentService` 能够妥善处理工作线程的生命周期。

例如，`onStartCommand()` 必须返回默认实现，即如何将 Intent 传递给 `onHandleIntent()`：

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()
    return super.onStartCommand(intent, flags, startId)
}
```

除 `onHandleIntent()` 之外，您无需从中调用超类的唯一方法就是 `onBind()`。只有在服务允许绑定时，您才需实现该方法。

在下一部分中，您将了解如何在扩展 `Service` 基类时实现同类服务，此类包含更多代码，但如需同时处理多个启动请求，则更适合使用该基类。

### 扩展服务类

借助 `IntentService`，您可以非常轻松地实现启动服务。但是，若要求服务执行多线程（而非通过工作队列处理启动请求），则可通过扩展 `Service` 类来处理每个 Intent。

为进行比较，以下示例代码展示了 `Service` 类的实现，该类执行的工作与上述使用 `IntentService` 的示例完全相同。换言之，对于每个启动请求，其均使用工作线程来执行作业，且每次仅处理一个请求。

```kotlin
class HelloService : Service() {

    private var serviceLooper: Looper? = null
    private var serviceHandler: ServiceHandler? = null

    // Handler that receives messages from the thread
    private inner class ServiceHandler(looper: Looper) : Handler(looper) {

        override fun handleMessage(msg: Message) {
            // Normally we would do some work here, like download a file.
            // For our sample, we just sleep for 5 seconds.
            try {
                Thread.sleep(5000)
            } catch (e: InterruptedException) {
                // Restore interrupt status.
                Thread.currentThread().interrupt()
            }

            // Stop the service using the startId, so that we don't stop
            // the service in the middle of handling another job
            stopSelf(msg.arg1)
        }
    }

    override fun onCreate() {
        // Start up the thread running the service.  Note that we create a
        // separate thread because the service normally runs in the process's
        // main thread, which we don't want to block.  We also make it
        // background priority so CPU-intensive work will not disrupt our UI.
        HandlerThread("ServiceStartArguments", Process.THREAD_PRIORITY_BACKGROUND).apply {
            start()

            // Get the HandlerThread's Looper and use it for our Handler
            serviceLooper = looper
            serviceHandler = ServiceHandler(looper)
        }
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()

        // For each start request, send a message to start a job and deliver the
        // start ID so we know which request we're stopping when we finish the job
        serviceHandler?.obtainMessage()?.also { msg ->
            msg.arg1 = startId
            serviceHandler?.sendMessage(msg)
        }

        // If we get killed, after returning from here, restart
        return START_STICKY
    }

    override fun onBind(intent: Intent): IBinder? {
        // We don't provide binding, so return null
        return null
    }

    override fun onDestroy() {
        Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show()
    }
}
```

如您所见，相较于使用 `IntentService`，此示例需要执行更多工作。

但是，由于 `onStartCommand()` 的每个调用均有您自己处理，因此您可以同时执行多个请求。此示例并未这样做，但若您希望如此，则可为每个请求创建新线程，然后立即运行这些线程（而非等待上一个请求完成）。

请注意，`onStartCommand()` 方法必须返回整型数。整型数是一个值，用于描述系统应如何在系统终止服务的情况下继续运行服务。`IntentService` 的默认实现会为您处理此情况，但您可以对其进行修改。从 `onStartCommand()` 返回的值必须是以下常量之一：

- `START_NOT_STICKY`

  如果系统在 `onStartCommand()` 返回后终止服务，则除非有待传递的挂起 Intent，否则系统*不会*重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。

- `START_STICKY`

  如果系统在 `onStartCommand()` 返回后终止服务，则其会重建服务并调用 `onStartCommand()`，但*不会*重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务，否则系统会调用包含空 Intent 的 `onStartCommand()`。在此情况下，系统会传递这些 Intent。此常量适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。

- `START_REDELIVER_INTENT`

  如果系统在 `onStartCommand()` 返回后终止服务，则其会重建服务，并通过传递给服务的最后一个 Intent 调用 `onStartCommand()`。所有挂起 Intent 均依次传递。此常量适用于主动执行应立即恢复的作业（例如下载文件）的服务。

如需详细了解这些返回值，请查阅每个常量的链接参考文档。有关此服务实现类型的扩展示例，请参阅 [GitHub 上的 MessagingService 示例](https://github.com/googlesamples/android-MessagingService)中的 `MessagingService` 类。



### 启动服务

您可以通过将 `Intent` 传递给 `startService()` 或 `startForegroundService()`，从 Activity 或其他应用组件启动服务。Android 系统会调用服务的 `onStartCommand()` 方法，并向其传递 `Intent`，从而指定要启动的服务。

>  **注意**：如果您的应用面向 API 级别 26 或更高版本，除非应用本身在前台运行，否则系统不会对使用或创建后台服务施加限制。如果应用需要创建前台服务，则其应调用 `startForegroundService()`。此方法会创建后台服务，但它会向系统发出信号，表明服务会将自行提升至前台。创建服务后，该服务必须在五秒内调用自己的 `startForeground()` 方法。

> mynote：还不是很理解这段话。

例如，Activity 可以结合使用显式 Intent 与 `startService()`，从而启动上文中的示例服务 (`HelloService`)：

```kotlin
Intent(this, HelloService::class.java).also { intent ->
    startService(intent)
}
```

`startService()` 方法会立即返回，并且 Android 系统会调用服务的 `onStartCommand()` 方法。如果服务尚未运行，则系统首先会调用 `onCreate()`，然后调用 `onStartCommand()`。

如果服务亦未提供绑定，则应用组件与服务间的唯一通信模式便是使用 `startService()` 传递的 Intent。但是，如果您希望服务返回结果，则启动服务的客户端可以为广播（通过 `getBroadcast()` 获得）创建一个 `PendingIntent`，并将其传递给启动服务的 `Intent` 中的服务。然后，服务便可使用广播传递结果。

> mynote：这段还没有实践。

多个服务启动请求会导致多次对服务的 `onStartCommand()` 进行相应的调用。但是，如要停止服务，只需一个服务停止请求（使用 `stopSelf()` 或 `stopService()`）即可。



### 停止服务

启动服务必须管理自己的生命周期。换言之，除非必须回收内存资源，否则系统不会停止或销毁服务，并且服务在 `onStartCommand()` 返回后仍会继续运行。服务必须通过调用 `stopSelf()` 自行停止运行，或由另一个组件通过调用 `stopService()` 来停止它。

一旦请求使用 `stopSelf()` 或 `stopService()` 来停止服务，系统便会尽快销毁服务。

如果服务同时处理多个对 `onStartCommand()` 的请求，则您不应在处理完一个启动请求之后停止服务，因为您可能已收到新的启动请求（在第一个请求结束时停止服务会终止第二个请求）。为避免此问题，您可以使用 `stopSelf(int)` 确保服务停止请求始终基于最近的启动请求。换言之，在调用 `stopSelf(int)` 时，您需传递与停止请求 ID 相对应的启动请求 ID（传递给 `onStartCommand()` 的 `startId`）。此外，如果服务在您能够调用 `stopSelf(int)` 之前收到新启动请求，则 ID 不匹配，服务也不会停止。

> **注意：**为避免浪费系统资源和消耗电池电量，请确保应用在工作完成之后停止其服务。如有必要，其他组件可通过调用 `stopService()` 来停止服务。即使为服务启用绑定，如果服务收到对 `onStartCommand()` 的调用，您始终仍须亲自停止服务。

如需了解有关服务生命周期的详细信息，请参阅下方[管理服务生命周期](https://developer.android.google.cn/guide/components/services?hl=zh-cn#Lifecycle)的相关部分。



## 2、创建绑定服务

绑定服务允许应用组件通过调用 `bindService()` 与其绑定，从而创建长期连接。此服务通常不允许组件通过调用 `startService()` 来*启动*它。

如需与 Activity 和其他应用组件中的服务进行交互，或需要通过进程间通信 (IPC) 向其他应用公开某些应用功能，则应创建绑定服务。

如要创建绑定服务，您需通过实现 `onBind()` 回调方法返回 `IBinder`，从而定义与服务进行通信的接口。然后，其他应用组件可通过调用 `bindService()` 来检索该接口，并开始调用与服务相关的方法。

服务只用于与其绑定的应用组件，因此若没有组件与该服务绑定，则系统会销毁该服务。您*不必*像通过 `onStartCommand()` 启动的服务那样，以相同方式停止绑定服务。

如要创建绑定服务，必须定义指定客户端如何与服务进行通信的接口。服务与客户端之间的这个接口必须是 `IBinder` 的实现，并且服务必须从 `onBind()` 回调方法返回该接口。收到 `IBinder` 后，客户端便可开始通过该接口与服务进行交互。

多个客户端可以同时绑定到服务。完成与服务的交互后，客户端会通过调用 `unbindService()` 来取消绑定。如果没有绑定到服务的客户端，则系统会销毁该服务。

实现绑定服务有多种方法，并且此实现比启动服务更为复杂。出于这些原因，请参阅另一份文档[绑定服务](https://developer.android.google.cn/guide/components/bound-services?hl=zh-cn)，了解关于绑定服务的详细介绍。



# 五、向用户发送通知

处于运行状态时，服务可以使用 [Toast 通知](https://developer.android.google.cn/guide/topics/ui/notifiers/toasts?hl=zh-cn) 或 [状态栏通知](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications?hl=zh-cn) 来通知用户所发生的事件。

Toast 通知是指仅在当前窗口界面显示片刻的消息。状态栏通知在状态栏中提供内含消息的图标，用户可通过选择该图标来执行操作（例如启动 Activity）。

通常，当某些后台工作（例如文件下载）已经完成，并且用户可对其进行操作时，状态栏通知是最佳方法。当用户从展开视图中选定通知时，该通知即可启动 Activity（例如显示已下载的文件）。

如需了解详细信息，请参阅 [Toast 通知](https://developer.android.google.cn/guide/topics/ui/notifiers/toasts?hl=zh-cn)或[状态栏通知](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications?hl=zh-cn)开发者指南。



# 六、在前台运行服务

前台服务是用户主动意识到的一种服务，因此在内存不足时，系统也不会考虑将其终止。前台服务必须为状态栏提供通知，将其放在*运行中的*标题下方。这意味着除非将服务停止或从前台移除，否则不能清除该通知。

> **注意：**请限制应用使用前台服务。
>
> 只有当应用执行的任务需供用户查看（即使该任务未直接与应用交互）时，您才应使用前台服务。因此，前台服务必须显示优先级为 `PRIORITY_LOW` 或更高的[状态栏通知](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications?hl=zh-cn)，这有助于确保用户知道应用正在执行的任务。如果某操作不是特别重要，因而您希望使用最低优先级通知，则可能不适合使用服务；相反，您可以考虑使用[计划作业](https://developer.android.google.cn/topic/performance/scheduling?hl=zh-cn)。
>
> 每个运行服务的应用都会给系统带来额外负担，从而消耗系统资源。如果应用尝试使用低优先级通知隐藏其服务，则可能会降低用户正在主动交互的应用的性能。因此，如果某个应用尝试运行拥有最低优先级通知的服务，则系统会在抽屉式通知栏的底部调用出该应用的行为。

例如，应将通过服务播放音乐的音乐播放器设置为在前台运行，因为用户会明确意识到其操作。状态栏中的通知可能表示正在播放的歌曲，并且其允许用户通过启动 Activity 与音乐播放器进行交互。同样，如果应用允许用户追踪其运行，则需通过前台服务来追踪用户的位置。

> **注意：**如果应用面向 Android 9（API 级别 28）或更高版本并使用前台服务，则其必须请求 [`FOREGROUND_SERVICE`](https://developer.android.google.cn/reference/android/Manifest.permission?hl=zh-cn#FOREGROUND_SERVICE) 权限。这是一种[普通权限](https://developer.android.google.cn/guide/topics/permissions/overview?hl=zh-cn#normal-dangerous)，因此，系统会自动为请求权限的应用授予此权限。
>
> 如果面向 API 级别 28 或更高版本的应用试图创建前台服务但未请求 `FOREGROUND_SERVICE`，则系统会抛出 [`SecurityException`](https://developer.android.google.cn/reference/java/lang/SecurityException?hl=zh-cn)。

如要请求让服务在前台运行，请调用 `startForeground()`。此方法采用两个参数：唯一标识通知的整型数和用于状态栏的 `Notification`。此通知必须拥有 `PRIORITY_LOW` 或更高的优先级。下面是示例：

```kotlin
val pendingIntent: PendingIntent =
        Intent(this, ExampleActivity::class.java).let { notificationIntent ->
            PendingIntent.getActivity(this, 0, notificationIntent, 0)
        }

val notification: Notification = Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
        .setContentTitle(getText(R.string.notification_title))
        .setContentText(getText(R.string.notification_message))
        .setSmallIcon(R.drawable.icon)
        .setContentIntent(pendingIntent)
        .setTicker(getText(R.string.ticker_text))
        .build()

startForeground(ONGOING_NOTIFICATION_ID, notification)
```

> **注意：**提供给 `startForeground()` 的整型 ID 不得为 0。

如要从前台移除服务，请调用 `stopForeground()`。此方法采用布尔值，指示是否需同时移除状态栏通知。此方法*不会*停止服务。但是，如果您在服务仍运行于前台时将其停止，则通知也会随之移除。

如需了解有关通知的详细信息，请参阅[创建状态栏通知](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications?hl=zh-cn)。



# 七、管理服务的生命周期

服务的生命周期比 Activity 的生命周期要简单得多。但是，密切关注如何创建和销毁服务反而更加重要，因为服务可以在用户未意识到的情况下运行于后台。

服务生命周期（从创建到销毁）可遵循以下任一路径：

- 启动服务

  该服务在其他组件调用 `startService()` 时创建，然后无限期运行，且必须通过调用 `stopSelf()` 来自行停止运行。此外，其他组件也可通过调用 `stopService()` 来停止此服务。服务停止后，系统会将其销毁。

- 绑定服务

  该服务在其他组件（客户端）调用 `bindService()` 时创建。然后，客户端通过 `IBinder` 接口与服务进行通信。客户端可通过调用 `unbindService()` 关闭连接。多个客户端可以绑定到相同服务，而且当所有绑定全部取消后，系统即会销毁该服务。（服务*不必*自行停止运行。）

这两条路径并非完全独立。您可以绑定到已使用 `startService()` 启动的服务。例如，您可以使用 `Intent`（标识要播放的音乐）来调用 `startService()`，从而启动后台音乐服务。随后，当用户需稍加控制播放器或获取有关当前所播放歌曲的信息时，Activity 可通过调用 `bindService()` 绑定到服务。此类情况下，在所有客户端取消绑定之前，`stopService()` 或 `stopSelf()` 实际不会停止服务。



## 实现生命周期回调

与 Activity 类似，服务也拥有生命周期回调方法，您可通过实现这些方法来监控服务状态的变化并适时执行工作。以下框架服务展示了每种生命周期方法：

```kotlin
class ExampleService : Service() {
    private var startMode: Int = 0             // indicates how to behave if the service is killed
    private var binder: IBinder? = null        // interface for clients that bind
    private var allowRebind: Boolean = false   // indicates whether onRebind should be used

    override fun onCreate() {
        // The service is being created
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // The service is starting, due to a call to startService()
        return mStartMode
    }

    override fun onBind(intent: Intent): IBinder? {
        // A client is binding to the service with bindService()
        return mBinder
    }

    override fun onUnbind(intent: Intent): Boolean {
        // All clients have unbound with unbindService()
        return mAllowRebind
    }

    override fun onRebind(intent: Intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }

    override fun onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```

> **注意：**与 Activity 生命周期回调方法不同，您*不*需要调用这些回调方法的超类实现。

![服务生命周期](images/service_lifecycle.png)

左图显示使用 `startService()` 创建的服务的生命周期，右图显示使用 `bindService()` 创建的服务的生命周期。

图 2 展示服务的典型回调方法。尽管该图分开介绍通过 `startService()` 创建的服务和通过 `bindService()` 创建的服务，但请记住，无论启动方式如何，任何服务均有可能允许客户端与其绑定。因此，最初使用 `onStartCommand()`（通过客户端调用 `startService()`）启动的服务仍可接收对 `onBind()` 的调用（当客户端调用 `bindService()` 时）。

通过实现这些方法，您可以监控服务生命周期的以下两种嵌套循环：

- 服务的**整个生命周期**贯穿调用 `onCreate()` 和返回 `onDestroy()` 之间的这段时间。与 Activity 类似，服务也在 `onCreate()` 中完成初始设置，并在 `onDestroy()` 中释放所有剩余资源。例如，音乐播放服务可以在 `onCreate()` 中创建用于播放音乐的线程，然后在 `onDestroy()` 中停止该线程。

> **注意**：无论所有服务是通过 `startService()` 还是 `bindService()` 创建，系统均会为其调用 `onCreate()` 和 `onDestroy()` 方法。

- 服务的**active 生命周期**从调用 `onStartCommand()` 或 `onBind()` 开始。每种方法均会获得 `Intent` 对象，该对象会传递至 `startService()` 或 `bindService()`。

  对于启动服务，active 生命周期与整个生命周期会同时结束（即便是在 `onStartCommand()` 返回之后，服务仍然处于活动状态）。对于绑定服务，active 生命周期会在 `onUnbind()` 返回时结束。

> **注意：**尽管您需通过调用 `stopSelf()` 或 `stopService()` 来停止绑定服务，但该服务并没有相应的回调（没有 `onStop()` 回调）。除非服务绑定到客户端，否则在服务停止时，系统会将其销毁（`onDestroy()` 是接收到的唯一回调）。

如需详细了解创建提供绑定的服务，请参阅[绑定服务](https://developer.android.google.cn/guide/components/bound-services?hl=zh-cn)文档，该文档的[管理绑定服务的生命周期](https://developer.android.google.cn/guide/components/bound-services?hl=zh-cn#Lifecycle)部分提供有关 `onRebind()` 回调方法的更多信息。







# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 脑图



# 参考

1、[官方文档-Service概述](https://developer.android.google.cn/guide/components/services?hl=zh-cn#Choosing-service-thread)