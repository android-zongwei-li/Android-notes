demo地址：



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

2、
