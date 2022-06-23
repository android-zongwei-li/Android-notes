> version：2022/06/22
>
> review：



目录

[TOC]



# 一、概述

绑定服务（bound service）是客户端-服务器接口中的服务端。借助绑定服务，组件（例如 Activity）可以绑定到服务、发送请求、接收响应，以及执行进程间通信 (IPC)。绑定服务通常只在为其他应用组件提供服务时处于活动状态，不会无限期在后台运行。

本文介绍如何创建绑定服务，包括如何绑定到来自其他应用组件的服务。如需了解有关一般服务的更多信息（例如：如何通过服务传送通知、如何将服务设置为在前台运行等），请参阅[服务](https://developer.android.google.cn/guide/components/services)文档。



# 二、基础知识

绑定服务是 `Service` 类的实现，可让其他应用与其进行绑定和交互。如需为服务提供绑定，您必须实现 `onBind()` 回调方法。此方法会返回一个 `IBinder` 对象，该对象定义的编程接口可供客户端用来与服务端进行交互。

### 绑定到已启动服务

如[服务](https://developer.android.google.cn/guide/components/services)文档中所述，您可以创建同时具有已启动和已绑定两种状态的服务。换言之，您可以通过调用 `startService()` 来启动服务，让服务无限期运行，您也可以通过调用 `bindService()` 让客户端绑定到该服务。

如果您确实允许服务同时具有已启动和已绑定状态，那么服务启动后，系统不会在所有客户端均与服务取消绑定后销毁服务，而必须由您通过调用 `stopSelf()` 或 `stopService()` 显式停止服务。

尽管您通常应实现 `onBind()` 或 `onStartCommand()`，但有时也需要同时实现这两种方法。例如，音乐播放器可能认为，让其服务无限期运行并同时提供绑定很有用处。如此一来，Activity 便可启动服务来播放音乐，并且即使用户离开应用，音乐播放也不会停止。然后，当用户返回应用时，Activity 便能绑定到服务，重新获得播放控制权。

如需详细了解为已启动服务添加绑定时的服务生命周期，请参阅[管理绑定服务的生命周期](https://developer.android.google.cn/guide/components/bound-services#Lifecycle)部分。

客户端通过调用 `bindService()` 绑定到服务。调用时，它必须提供 `ServiceConnection` 的实现，后者会监控与服务的连接。`bindService()` 的返回值指示所请求的服务是否存在，以及是否允许客户端访问该服务。Android 系统创建客户端与服务之间的连接时，会对 `ServiceConnection` 调用 `onServiceConnected()`。`onServiceConnected()` 方法包含一个 `IBinder` 参数，客户端随后会使用该参数与绑定服务通信。

您可以将多个客户端同时连接到某项服务。但是，系统会缓存 `IBinder` 服务通信通道。换言之，只有在第一个客户端绑定服务时，系统才会调用服务的 `onBind()` 方法来生成 `IBinder`。然后，系统会将该 `IBinder` 传递至绑定到同一服务的所有其他客户端，无需再次调用 `onBind()`。

当最后一个客户端取消与服务的绑定时，系统会销毁该服务（除非还通过 `startService()` 启动了该服务）。

在实现绑定服务的过程中，最重要的环节是定义 `onBind()` 回调方法所返回的接口。下一部分将为您介绍几种不同方式，以便您定义服务的 `IBinder` 接口。

# 三、创建绑定服务

创建提供绑定的服务时，您必须提供 `IBinder`，用以提供编程接口，供客户端与服务进行交互之用。您可以通过三种方式定义接口：

- [扩展 Binder 类](https://developer.android.google.cn/guide/components/bound-services#Binder)

  如果服务是供您的自有应用专用，并且在与客户端相同的进程中运行（常见情况），您应当通过扩展 `Binder` 类并从 `onBind()` 返回该类的实例来创建接口。客户端收到 `Binder` 后，可利用它直接访问 `Binder` 实现或 `Service` 中提供的公共方法。如果服务只是您自有应用的后台工作器，应优先采用这种方式。您不使用这种方式创建接口的唯一一种情况是：其他应用或不同进程占用了您的服务。

- [使用 Messenger](https://developer.android.google.cn/guide/components/bound-services#Messenger)

  如需让接口跨不同进程工作，您可以使用 `Messenger` 为服务创建接口。采用这种方式时，服务会定义一个 `Handler`，用于响应不同类型的 `Message` 对象。此 `Handler` 是 `Messenger` 的基础，后者随后可与客户端分享一个 `IBinder`，以便客户端能利用 `Message` 对象向服务发送命令。此外，客户端还可定义一个自有 `Messenger`，以便服务回传消息。这是执行进程间通信 (IPC) 最为简单的方式，因为 `Messenger` 会在单个线程中创建包含所有请求的队列，这样您就不必对服务进行线程安全设计。

- [使用 AIDL](https://developer.android.google.cn/guide/components/aidl)

  Android 接口定义语言 (AIDL) 会将对象分解成原语，操作系统可通过识别这些原语并将其编组到各进程中来执行 IPC。以前采用 `Messenger` 的方式实际上是以 AIDL 作为其底层结构。如上所述，`Messenger` 会在单个线程中创建包含所有客户端请求的队列，以便服务一次接收一个请求。不过，如果您想让服务同时处理多个请求，可以直接使用 AIDL。在此情况下，您的服务必须达到线程安全的要求，并且能够进行多线程处理。如需直接使用 AIDL，您必须创建用于定义编程接口的 `.aidl` 文件。Android SDK 工具会利用该文件生成实现接口和处理 IPC 的抽象类，您随后可在服务内对该类进行扩展。

> **注意**：大多数应用不应使用 AIDL 来创建绑定服务，因为它可能需要多线程处理能力，并可能导致更为复杂的实现。因此，AIDL 并不适合大多数应用，本文也不会阐述如何将其用于您的服务。如果您确定自己需要直接使用 AIDL，请参阅 [AIDL](https://developer.android.google.cn/guide/components/aidl) 文档。



### 扩展 Binder 类

如果您的服务仅供本地应用使用，且无需跨进程工作，您可以实现自有 `Binder` 类，让客户端通过该类直接访问服务中的公共方法。

**注意**：只有当客户端和服务处于同一应用和进程内（最常见的情况）时，此方式才有效。例如，对于需要将 Activity 绑定到在后台播放音乐的自有服务的音乐应用，此方式非常有效。

以下为设置方式：

1. 在您的服务中，创建可执行以下某种操作的 `Binder` 实例：
   - 包含客户端可调用的公共方法。
   - 返回当前的 `Service` 实例，该实例中包含客户端可调用的公共方法。
   - 返回由服务承载的其他类的实例，其中包含客户端可调用的公共方法。
2. 从 `onBind()` 回调方法返回此 `Binder` 实例。
3. 在客户端中，从 `onServiceConnected()` 回调方法接收 `Binder`，并使用提供的方法调用绑定服务。

> **注意**：服务和客户端**必须在同一应用内**，这样客户端才能转换返回的对象并正确调用其 API。服务和客户端还**必须在同一进程内**，因为此方式不执行任何跨进程编组。

例如，以下服务可让客户端通过 `Binder` 实现访问服务中的方法：

```kotlin
class LocalService : Service() {
    // Binder given to clients
    private val binder = LocalBinder()

    // Random number generator
    private val mGenerator = Random()

    /** method for clients  */
    val randomNumber: Int
        get() = mGenerator.nextInt(100)

    /**
     * Class used for the client Binder.  Because we know this service always
     * runs in the same process as its clients, we don't need to deal with IPC.
     */
    inner class LocalBinder : Binder() {
        // Return this instance of LocalService so clients can call public methods
        fun getService(): LocalService = this@LocalService
    }

    override fun onBind(intent: Intent): IBinder {
        return binder
    }
}
```

`LocalBinder` 为客户端提供 `getService()` 方法，用于检索 `LocalService` 的当前实例。这样，客户端便可调用服务中的公共方法。例如，客户端可调用服务中的 `getRandomNumber()`。

点击按钮时，以下 Activity 会绑定到 `LocalService` 并调用 `getRandomNumber()`：

```kotlin
class BindingActivity : Activity() {
    private lateinit var mService: LocalService
    private var mBound: Boolean = false

    /** Defines callbacks for service binding, passed to bindService()  */
    private val connection = object : ServiceConnection {

        override fun onServiceConnected(className: ComponentName, service: IBinder) {
            // We've bound to LocalService, cast the IBinder and get LocalService instance
            val binder = service as LocalService.LocalBinder
            mService = binder.getService()
            mBound = true
        }

        override fun onServiceDisconnected(arg0: ComponentName) {
            mBound = false
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main)
    }

    override fun onStart() {
        super.onStart()
        // Bind to LocalService
        Intent(this, LocalService::class.java).also { intent ->
            bindService(intent, connection, Context.BIND_AUTO_CREATE)
        }
    }

    override fun onStop() {
        super.onStop()
        unbindService(connection)
        mBound = false
    }

    /** Called when a button is clicked (the button in the layout file attaches to
     * this method with the android:onClick attribute)  */
    fun onButtonClick(v: View) {
        if (mBound) {
            // Call a method from the LocalService.
            // However, if this call were something that might hang, then this request should
            // occur in a separate thread to avoid slowing down the activity performance.
            val num: Int = mService.randomNumber
            Toast.makeText(this, "number: $num", Toast.LENGTH_SHORT).show()
        }
    }
}
```

上例说明了客户端如何使用 `ServiceConnection` 的实现和 `onServiceConnected()` 回调绑定到服务。下一部分将更详细地介绍绑定到服务的过程。

> **注意**：在上例中，`onStop()` 方法取消了客户端与服务的绑定。如[其他说明](https://developer.android.google.cn/guide/components/bound-services#Additional_Notes)中所述，客户端应在适当时机与服务取消绑定。

如需查看更多示例代码，请参阅 [ApiDemos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos) 中的 [`LocalService.java`](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/LocalService.java) 类和 [`LocalServiceActivities.java`](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/LocalServiceActivities.java) 类。



### 使用 Messenger

如果您需要让服务与远程进程通信，则可使用 `Messenger` 为您的服务提供接口。借助此方式，您无需使用 AIDL 便可执行进程间通信 (IPC)。

为接口使用 `Messenger` 比使用 AIDL 更简单，因为 `Messenger` 会将所有服务调用加入队列。纯 AIDL 接口会同时向服务发送多个请求，那么服务就必须执行多线程处理。

对于大多数应用，服务无需执行多线程处理，因此使用 `Messenger` 可让服务一次处理一个调用。如果您的服务必须执行多线程处理，请使用 [AIDL](https://developer.android.google.cn/guide/components/aidl) 来定义接口。

以下是对 `Messenger` 使用方式的摘要：

1. 服务实现一个 `Handler`，由其接收来自客户端的每个调用的回调。
2. 服务使用 `Handler` 来创建 `Messenger` 对象（该对象是对 `Handler` 的引用）。
3. `Messenger` 创建一个 `IBinder`，服务通过 `onBind()` 将其返回给客户端。
4. 客户端使用 `IBinder` 将 `Messenger`（它引用服务的 `Handler`）实例化，然后再用其将 `Message` 对象发送给服务。
5. 服务在其 `Handler` 中（具体而言，是在 `handleMessage()` 方法中）接收每个 `Message`。

这样，客户端便没有调用服务的方法。相反，客户端会传递服务在其 `Handler` 中接收的消息（`Message` 对象）。

下面这个简单的服务实例展示了如何使用 `Messenger` 接口：

```kotlin
/** Command to the service to display a message  */
private const val MSG_SAY_HELLO = 1

class MessengerService : Service() {

    /**
     * Target we publish for clients to send messages to IncomingHandler.
     */
    private lateinit var mMessenger: Messenger

    /**
     * Handler of incoming messages from clients.
     */
    internal class IncomingHandler(
            context: Context,
            private val applicationContext: Context = context.applicationContext
    ) : Handler() {
        override fun handleMessage(msg: Message) {
            when (msg.what) {
                MSG_SAY_HELLO ->
                    Toast.makeText(applicationContext, "hello!", Toast.LENGTH_SHORT).show()
                else -> super.handleMessage(msg)
            }
        }
    }

    /**
     * When binding to the service, we return an interface to our messenger
     * for sending messages to the service.
     */
    override fun onBind(intent: Intent): IBinder? {
        Toast.makeText(applicationContext, "binding", Toast.LENGTH_SHORT).show()
        mMessenger = Messenger(IncomingHandler(this))
        return mMessenger.binder
    }
}
```

请注意，服务会在 `Handler` 的 `handleMessage()` 方法中接收传入的 `Message`，并根据 `what` 成员决定下一步操作。

客户端只需根据服务返回的 `IBinder` 创建 `Messenger`，然后使用 `send()` 发送消息。例如，以下就是一个绑定到服务并向服务传递 `MSG_SAY_HELLO` 消息的简单 Activity：

```kotlin
class ActivityMessenger : Activity() {
    /** Messenger for communicating with the service.  */
    private var mService: Messenger? = null

    /** Flag indicating whether we have called bind on the service.  */
    private var bound: Boolean = false

    /**
     * Class for interacting with the main interface of the service.
     */
    private val mConnection = object : ServiceConnection {

        override fun onServiceConnected(className: ComponentName, service: IBinder) {
            // This is called when the connection with the service has been
            // established, giving us the object we can use to
            // interact with the service.  We are communicating with the
            // service using a Messenger, so here we get a client-side
            // representation of that from the raw IBinder object.
            mService = Messenger(service)
            bound = true
        }

        override fun onServiceDisconnected(className: ComponentName) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null
            bound = false
        }
    }

    fun sayHello(v: View) {
        if (!bound) return
        // Create and send a message to the service, using a supported 'what' value
        val msg: Message = Message.obtain(null, MSG_SAY_HELLO, 0, 0)
        try {
            mService?.send(msg)
        } catch (e: RemoteException) {
            e.printStackTrace()
        }

    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main)
    }

    override fun onStart() {
        super.onStart()
        // Bind to the service
        Intent(this, MessengerService::class.java).also { intent ->
            bindService(intent, mConnection, Context.BIND_AUTO_CREATE)
        }
    }

    override fun onStop() {
        super.onStop()
        // Unbind from the service
        if (bound) {
            unbindService(mConnection)
            bound = false
        }
    }
}
```

请注意，此示例并未展示服务如何对客户端做出响应。如果您想让服务做出响应，还需在客户端中创建一个 `Messenger`。当客户端收到 `onServiceConnected()` 回调时，会向服务发送一个 `Message`，并在其 `send()` 方法的 `replyTo` 参数中加入客户端的 `Messenger`。

如需查看如何提供双向消息传递的示例，请参阅 [`MessengerService.java`](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.java)（服务）和 [`MessengerServiceActivities.java`](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.java)（客户端）示例。



# 四、绑定到服务

应用组件（客户端）可通过调用 `bindService()` 绑定到服务。然后，Android 系统会调用服务的 `onBind()` 方法，该方法会返回用于与服务交互的 `IBinder`。

绑定是异步操作，并且 `bindService()` 可立即返回，无需将 `IBinder` 返回给客户端。如要接收 `IBinder`，客户端必须创建一个 `ServiceConnection` 实例，并将其传递给 `bindService()`。`ServiceConnection` 包含一个回调方法，系统通过调用该回调方法来传递 `IBinder`。

> **注意**：只有 Activity、服务和内容提供程序可以绑定到服务，您**无法**从广播接收器绑定到服务。

如要从您的客户端绑定到服务，请按以下步骤操作：

- 实现 `ServiceConnection` 。

  您的实现必须替换两个回调方法：

  - `onServiceConnected()`

    系统会调用该方法以传递服务的 `onBind()` 方法所返回的 `IBinder`。

  - `onServiceDisconnected()`

    当与服务的连接意外中断时，例如服务崩溃或被终止时，Android 系统会调用该方法。当客户端取消绑定时，系统不会调用该方法。

- 调用 `bindService()`，从而传递 `ServiceConnection` 实现。

  > **注意**：如果该方法返回 false，说明您的客户端与服务之间并无有效连接。不过，您的客户端仍应调用 `unbindService()`；否则，您的客户端会使服务无法在空闲时关闭。

- 当系统调用 `onServiceConnected()` 回调方法时，您可以使用接口定义的方法开始调用服务。

- 如要断开与服务的连接，请调用 `unbindService()`。

  当应用销毁客户端时，如果客户端仍与服务保持绑定状态，销毁会导致客户端取消绑定。更好的做法是在客户端与服务交互完成后，就立即取消客户端的绑定。这样做可以关闭空闲的服务。如需详细了解有关绑定和取消绑定的适当时机，请参阅[其他说明](https://developer.android.google.cn/guide/components/bound-services#Additional_Notes)。

以下示例通过[扩展 Binder 类](https://developer.android.google.cn/guide/components/bound-services#Binder)将客户端连接到上面创建的服务，因此它只需将返回的 `IBinder` 转换为 `LocalBinder` 类并请求 `LocalService` 实例：

```kotlin
var mService: LocalService

val mConnection = object : ServiceConnection {
    // Called when the connection with the service is established
    override fun onServiceConnected(className: ComponentName, service: IBinder) {
        // Because we have bound to an explicit
        // service that is running in our own process, we can
        // cast its IBinder to a concrete class and directly access it.
        val binder = service as LocalService.LocalBinder
        mService = binder.getService()
        mBound = true
    }

    // Called when the connection with the service disconnects unexpectedly
    override fun onServiceDisconnected(className: ComponentName) {
        Log.e(TAG, "onServiceDisconnected")
        mBound = false
    }
}
```

如以下示例所示，客户端可将此 `ServiceConnection` 传递至 `bindService()`，从而绑定到服务：

```kotlin
Intent(this, LocalService::class.java).also { intent ->
    bindService(intent, connection, Context.BIND_AUTO_CREATE)
}
```

- `bindService()` 的第一个参数是一个 `Intent`，用于显式命名要绑定的服务。

  > **注意**：如果您使用 intent 绑定到 `Service`，请使用[显式](https://developer.android.google.cn/guide/components/intents-filters#Types) intent 来确保应用的安全性。使用隐式 intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 intent 调用 `bindService()`，系统会抛出异常。

- 第二个参数是 `ServiceConnection` 对象。

- 第三个参数是指示绑定选项的标记。如要创建尚未处于活动状态的服务，此参数通常应为 `BIND_AUTO_CREATE`。其他可能的值为 `BIND_DEBUG_UNBIND` 和 `BIND_NOT_FOREGROUND`，或者 `0`（表示无此参数）。



# 五、其他说明

以下是一些有关绑定到服务的重要说明：

- 您应该始终捕获 `DeadObjectException` 异常，系统会在连接中断时抛出此异常。这是远程方法抛出的唯一异常。
- 对象是跨进程计数的引用。
- 如以下示例所述，在匹配客户端生命周期的引入 (bring-up) 和退出 (tear-down) 时刻期间，您通常需配对绑定和取消绑定：
  - 如果您只需在 Activity 可见时与服务交互，应在 `onStart()` 期间进行绑定，在 `onStop()` 期间取消绑定。
  - 如果您希望 Activity 即使在后台停止运行状态下仍可接收响应，那么您可以在 `onCreate()` 期间绑定，在 `onDestroy()` 期间取消绑定。请注意，这意味着您的 Activity 在其整个运行过程中（甚至包括后台运行期间）都需要使用服务，因此如果服务位于其他进程内，那么当您提高该进程的权重时，系统终止该进程的可能性会增加。

> **注意**：通常情况下，不应在 Activity 的 `onResume()` 和 `onPause()` 期间绑定和取消绑定，因为每次切换生命周期状态时都会发生这些回调，您应将这些转换期间的处理工作保持在最低水平。此外，如果您的应用内的多个 Activity 绑定到同一服务，并且其中两个 Activity 之间发生了转换，那么如果当前 Activity 先取消绑定（暂停期间），然后下一个 Activity 再进行绑定（恢复期间），系统可能就会销毁后再重新创建该服务。如需了解 Activity 如何协调其生命周期的 Activity 转换，请参阅 [Activity](https://developer.android.google.cn/guide/components/activities#CoordinatingActivities) 文档。

如需查看更多展示如何绑定到服务的示例代码，请参阅 [ApiDemos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos) 中的 [`RemoteService.java`](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/RemoteService.java) 类。



# 六、管理绑定服务的生命周期

当服务与所有客户端之间的绑定全部取消时，Android 系统会销毁该服务（除非还使用 `startService()` 调用启动了该服务）。因此，如果您的服务是纯粹的绑定服务，则无需对其生命周期进行管理，Android 系统会根据它是否绑定到任何客户端代您管理。

不过，如果您选择实现 `onStartCommand()` 回调方法，就必须显式停止服务，因为系统现在将服务视为已启动状态。在此情况下，服务将一直运行，直到其通过 `stopSelf()` 自行停止，或其他组件调用 `stopService()`（与该服务是否绑定到任何客户端无关）。

此外，如果您的服务已启动并接受绑定，那么当系统调用您的 `onUnbind()` 方法时，如果您想在客户端下一次绑定到服务时接收 `onRebind()` 调用，可以选择返回 `true`。`onRebind()` 返回空值，但客户端仍会在其 `onServiceConnected()` 回调中接收 `IBinder`。下图说明了这种生命周期的逻辑。

![已启动并且还允许绑定的服务的生命周期。](images/service_binding_tree_lifecycle.png)

如需详细了解已启动服务的生命周期，请参阅[服务](https://developer.android.google.cn/guide/components/services#Lifecycle)文档。







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

1、[绑定服务概览](https://developer.android.google.cn/guide/components/bound-services#Creating)