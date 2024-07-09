# 使用 Hilt 实现依赖项注入

Hilt 是 Android 的依赖项注入库，可减少在项目中执行手动依赖项注入的样板代码。执行[手动依赖项注入](https://developer.android.google.cn/training/dependency-injection/manual?hl=zh-cn)要求你手动构造每个类及其依赖项，并借助容器重复使用和管理依赖项。

Hilt 通过为项目中的每个 Android 类提供容器并自动管理其生命周期，提供了一种在应用中使用 DI（依赖项注入）的标准方法。Hilt 在 [Dagger](https://developer.android.google.cn/training/dependency-injection/dagger-basics?hl=zh-cn) 的基础上构建而成，因而能够受益于 Dagger 的编译时正确性、运行时性能、可伸缩性和 [Android Studio 支持](https://medium.com/androiddevelopers/dagger-navigation-support-in-android-studio-49aa5d149ec9)。如需了解详情，请参阅 [Hilt 和 Dagger](https://developer.android.google.cn/training/dependency-injection/hilt-android?hl=zh-cn#hilt-and-dagger)。

本文介绍了 Hilt 的基本概念及其生成的容器，演示了如何开始在现有应用中使用 Hilt。

# 使用Hilt

## 添加依赖项

项目级 `build.gradle` 中添加插件：

```kotlin
plugins {
  ...
  id("com.google.dagger.hilt.android") version "2.44" apply false
}
```

在 `app/build.gradle` 文件中添加以下依赖项：

```kotlin
plugins {
  id 'kotlin-kapt'
  id("com.google.dagger.hilt.android")
}

android {
  ...
  compileOptions {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
  }
}

dependencies {
  implementation("com.google.dagger:hilt-android:2.44")
  kapt("com.google.dagger:hilt-android-compiler:2.44")
}

// Allow references to generated code
kapt {
  correctErrorTypes = true
}
```

## 添加注解@HiltAndroidApp

必须包含一个带有 `@HiltAndroidApp` 注解的 [`Application`](https://developer.android.google.cn/reference/android/app/Application?hl=zh-cn) 类。

`@HiltAndroidApp` 会触发 Hilt 的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器。

```kotlin
@HiltAndroidApp
class ExampleApplication : Application() { ... }
```

生成的这一 Hilt 组件会附加到 `Application` 对象的生命周期，并为其提供依赖项。此外，它也是应用的父组件，这意味着，其他组件可以访问它提供的依赖项。

## 将依赖项注入 Android 类

在 `Application` 类中设置了 Hilt 且有了应用级组件后，Hilt 可以为带有 `@AndroidEntryPoint` 注解的其他 Android 类提供依赖项：

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() { ... }
```

Hilt 目前支持以下 Android 类：

- `Application`（通过使用 `@HiltAndroidApp`）
- `ViewModel`（通过使用 `@HiltViewModel`）
- `Activity`
- `Fragment`
- `View`
- `Service`
- `BroadcastReceiver`

如果使用 `@AndroidEntryPoint` 为某个 Android 类添加注解，则还必须为依赖于该类的 Android 类添加注解。例如，如果给某个 fragment 添加了注解，则还必须为使用该 fragment 的所有 activity 添加注解。

> **注意**：在 Hilt 对 Android 类的支持方面适用以下几项例外情况：
>
> Hilt 仅支持扩展 [`ComponentActivity`](https://developer.android.google.cn/reference/kotlin/androidx/activity/ComponentActivity?hl=zh-cn) 的 activity，如 [`AppCompatActivity`](https://developer.android.google.cn/reference/kotlin/androidx/appcompat/app/AppCompatActivity?hl=zh-cn)。
>
> Hilt 仅支持扩展 `androidx.Fragment` 的 Fragment。
>
> Hilt support 中的 fragment。

`@AndroidEntryPoint` 会为项目中的每个 Android 类生成一个单独的 Hilt 组件。这些组件可以从它们各自的父类接收依赖项，在【组件层次结构】一节详细描述。

如需从组件获取依赖项，请使用 `@Inject` 注解执行字段注入：

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() {

  @Inject lateinit var analytics: AnalyticsAdapter
  ...
}
```

**注意**：由 Hilt 注入的字段不能为私有字段。尝试使用 Hilt 注入私有字段会导致编译错误。

使用 Hilt 注入的类可以有同样使用注入的基类。如果基类是抽象类，基类可以不用 `@AndroidEntryPoint` 注解。

## 定义 Hilt 绑定

为了执行字段注入，Hilt 需要知道如何从相应组件提供必要依赖项的实例。“绑定”包含将某个类型的实例作为依赖项提供所需的信息。

向 Hilt 提供绑定信息的一种方法是构造函数注入。**在某个类的构造函数中使用 `@Inject` 注解，以告知 Hilt 如何提供该类的实例**：

```kotlin
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

在一个类的代码中，带有注解的构造函数的参数即是该类的依赖项。在本例中，`AnalyticsService` 是 `AnalyticsAdapter` 的一个依赖项。因此，Hilt 还必须知道如何提供 `AnalyticsService` 的实例。

> **注意**：在构建时，Hilt 会为 Android 类生成 [Dagger](https://developer.android.google.cn/training/dependency-injection/dagger-basics?hl=zh-cn) 组件。然后，Dagger 会走查你的代码，并执行以下步骤：
>
> 构建并验证依赖关系图，确保没有未满足的依赖关系且没有依赖循环。
>
> 生成它在运行时用来创建实际对象及其依赖项的类。

## Hilt modules

有时，类型不能通过构造函数注入。发生这种情况可能有多种原因。例如，你不能通过构造函数注入接口。此外，也不能通过构造函数注入不归你所有的类型，如来自外部库的类。在这些情况下，可以使用 Hilt 模块向 Hilt 提供绑定信息。

**Hilt 模块是一个带有 `@Module` 注解的类，它会告知 Hilt 如何提供某些类型的实例**。与 Dagger 模块不同的是，必须**使用 `@InstallIn` 为 Hilt 模块添加注解，以告知 Hilt 每个模块将用在或安装在哪个 Android 类中**。

在 Hilt 模块中提供的依赖项可以在生成的所有与 Hilt 模块安装到的 Android 类关联的组件中使用。

**注意**：由于 Hilt 的代码生成操作需要访问使用 Hilt 的所有 Gradle 模块，因此编译 `Application` 类的 Gradle 模块还需要在其传递依赖项中包含你的所有 Hilt 模块和通过构造函数注入的类。

### 使用 @Binds 注入接口实例

以 `AnalyticsService` 为例。如果 `AnalyticsService` 是一个接口，则无法通过构造函数注入它，而应向 Hilt 提供绑定信息，方法是在 Hilt 模块内创建一个带有 `@Binds` 注解的抽象函数。

**`@Binds` 注解会告知 Hilt 在需要提供接口的实例时要使用哪种实现。**

带有注解的函数会向 Hilt 提供以下信息：

- 函数返回类型会告知 Hilt 该函数提供哪个接口的实例。
- 函数参数会告知 Hilt 要提供哪种实现。

```kotlin
interface AnalyticsService {
  fun analyticsMethods()
}

// Constructor-injected, because Hilt needs to know how to
// provide instances of AnalyticsServiceImpl, too.
class AnalyticsServiceImpl @Inject constructor(
  ...
) : AnalyticsService { ... }

@Module
@InstallIn(ActivityComponent::class)
abstract class AnalyticsModule {

  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}
```

Hilt modules AnalyticsModule 带有 `@InstallIn(ActivityComponent.class)` 注解，因为你希望 Hilt 将该依赖项注入 `ExampleActivity`。此注解意味着，AnalyticsModule 中的所有依赖项都可以在应用的所有 activity 中使用。

### 使用 @Provides 注入实例

接口不是无法通过构造函数注入类型的唯一一种情况。如果某个类不归你所有（因为它来自外部库，如 [Retrofit](https://square.github.io/retrofit/)、[`OkHttpClient`](https://square.github.io/okhttp/) 或 [Room 数据库](https://developer.android.google.cn/topic/libraries/architecture/room?hl=zh-cn)等类），或者必须使用[构建器模式](https://en.wikipedia.org/wiki/Builder_pattern)创建实例，也无法通过构造函数注入。

接着前面的例子来讲。如果 `AnalyticsService` 类不直接归你所有，你可以告知 Hilt 如何提供此类型的实例，方法是在 Hilt modules 中创建一个函数，并使用 `@Provides` 为该函数添加注解。

带有注解的函数会向 Hilt 提供以下信息：

- 函数返回类型会告知 Hilt 函数提供哪个类型的实例。
- 函数参数会告知 Hilt 相应类型的依赖项。
- 函数主体会告知 Hilt 如何提供相应类型的实例。每当需要提供该类型的实例时，Hilt 都会执行函数主体。

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    // Potential dependencies of this type
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

### 为同一类型提供多个绑定

如果需要让 Hilt 以依赖项的形式提供同一类型的不同实现，必须向 Hilt 提供多个绑定。可以使用限定符为同一类型定义多个绑定。

限定符是一种注解，当为某个类型定义了多个绑定时，可以使用它来标识该类型的特定绑定。

仍然接着前面的例子来讲。如果需要拦截对 `AnalyticsService` 的调用，你可以使用带有[拦截器](https://square.github.io/okhttp/interceptors/)的 `OkHttpClient`对象。对于其他服务，你可能需要以不同的方式拦截调用。在这种情况下，需要告知 Hilt 如何提供两种不同的 `OkHttpClient` 实现。

首先，定义要用于为 `@Binds` 或 `@Provides` 方法添加注解的限定符：

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class OtherInterceptorOkHttpClient
```

然后，Hilt 需要知道如何提供与每个限定符对应的类型的实例。在这种情况下，可以使用带有 `@Provides` 的 Hilt 模块。这两种方法具有相同的返回类型，但限定符将它们标记为两个不同的绑定：

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

  @AuthInterceptorOkHttpClient
  @Provides
  fun provideAuthInterceptorOkHttpClient(
    authInterceptor: AuthInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(authInterceptor)
               .build()
  }

  @OtherInterceptorOkHttpClient
  @Provides
  fun provideOtherInterceptorOkHttpClient(
    otherInterceptor: OtherInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(otherInterceptor)
               .build()
  }
}
```

你可以通过使用相应的限定符为字段或参数添加注解来注入所需的特定类型：

```kotlin
// As a dependency of another class.
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    @AuthInterceptorOkHttpClient okHttpClient: OkHttpClient
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .client(okHttpClient)
               .build()
               .create(AnalyticsService::class.java)
  }
}

// As a dependency of a constructor-injected class.
class ExampleServiceImpl @Inject constructor(
  @AuthInterceptorOkHttpClient private val okHttpClient: OkHttpClient
) : ...

// At field injection.
@AndroidEntryPoint
class ExampleActivity: AppCompatActivity() {

  @AuthInterceptorOkHttpClient
  @Inject lateinit var okHttpClient: OkHttpClient
}
```

最佳实践是，如果你向某个类型添加限定符，应向提供该依赖项的所有可能的方式添加限定符。让基本实现或通用实现不带限定符容易出错，并且可能会导致 Hilt 注入错误的依赖项。

### Hilt 中的预定义限定符

Hilt 提供了一些预定义的限定符。例如，如果需要来自应用或 activity 的 `Context` 类，Hilt 提供了 `@ApplicationContext` 和 `@ActivityContext` 限定符。

假设本例中的 `AnalyticsAdapter` 类需要 activity 的上下文。以下代码演示了如何向 `AnalyticsAdapter` 提供 activity 上下文：

```kotlin
class AnalyticsAdapter @Inject constructor(
    @ActivityContext private val context: Context,
    private val service: AnalyticsService
) { ... }
```

## 为 Android 类生成的组件

对于你可以从中执行字段注入的每个 Android 类，都有一个关联的 Hilt 组件，你可以在 `@InstallIn` 注解中引用该组件。每个 Hilt 组件负责将其绑定注入相应的 Android 类。

前面的示例演示了如何在 Hilt 模块中使用 `ActivityComponent`。

Hilt 提供了以下组件：

| Hilt 组件                   | 注入器面向的对象                           |
| :-------------------------- | :----------------------------------------- |
| `SingletonComponent`        | `Application`                              |
| `ActivityRetainedComponent` | 不适用                                     |
| `ViewModelComponent`        | `ViewModel`                                |
| `ActivityComponent`         | `Activity`                                 |
| `FragmentComponent`         | `Fragment`                                 |
| `ViewComponent`             | `View`                                     |
| `ViewWithFragmentComponent` | 带有 `@WithFragmentBindings` 注解的 `View` |
| `ServiceComponent`          | `Service`                                  |

**注意**：Hilt 不会为广播接收器生成组件，因为 Hilt 直接从 `SingletonComponent` 注入广播接收器。

### 组件生命周期

Hilt 会按照相应 Android 类的生命周期自动创建和销毁生成的组件类的实例。

| 生成的组件                  | 创建时机                 | 销毁时机                |
| :-------------------------- | :----------------------- | :---------------------- |
| `SingletonComponent`        | `Application#onCreate()` | `Application` destroyed |
| `ActivityRetainedComponent` | `Activity#onCreate()`    | `Activity#onDestroy()`  |
| `ViewModelComponent`        | `ViewModel` created      | `ViewModel` destroyed   |
| `ActivityComponent`         | `Activity#onCreate()`    | `Activity#onDestroy()`  |
| `FragmentComponent`         | `Fragment#onAttach()`    | `Fragment#onDestroy()`  |
| `ViewComponent`             | `View#super()`           | `View` destroyed        |
| `ViewWithFragmentComponent` | `View#super()`           | `View` destroyed        |
| `ServiceComponent`          | `Service#onCreate()`     | `Service#onDestroy()`   |

**注意**：`ActivityRetainedComponent` 在配置更改后仍然存在，因此它在第一次调用 `Activity#onCreate()` 时创建，在最后一次调用 `Activity#onDestroy()` 时销毁。

### 组件作用域

默认情况下，Hilt 中的所有绑定都未限定作用域。这意味着，每当应用请求绑定时，Hilt 都会创建所需类型的一个新实例。

在本例中，每当 Hilt 提供 `AnalyticsAdapter` 作为其他类型的依赖项或通过字段注入提供它（如在 `ExampleActivity`中）时，Hilt 都会提供 `AnalyticsAdapter` 的一个新实例。

不过，Hilt 也允许将绑定的作用域限定为特定组件。Hilt 只为绑定作用域限定到的组件的每个实例创建一次限定作用域的绑定，对该绑定的所有请求共享同一实例。

下表列出了生成的每个组件的作用域注解：

| Android 类                                 | 生成的组件                  | 作用域                    |
| :----------------------------------------- | :-------------------------- | :------------------------ |
| `Application`                              | `SingletonComponent`        | `@Singleton`              |
| `Activity`                                 | `ActivityRetainedComponent` | `@ActivityRetainedScoped` |
| `ViewModel`                                | `ViewModelComponent`        | `@ViewModelScoped`        |
| `Activity`                                 | `ActivityComponent`         | `@ActivityScoped`         |
| `Fragment`                                 | `FragmentComponent`         | `@FragmentScoped`         |
| `View`                                     | `ViewComponent`             | `@ViewScoped`             |
| 带有 `@WithFragmentBindings` 注解的 `View` | `ViewWithFragmentComponent` | `@ViewScoped`             |
| `Service`                                  | `ServiceComponent`          | `@ServiceScoped`          |

在本例中，如果你使用 `@ActivityScoped` 将 `AnalyticsAdapter` 的作用域限定为 `ActivityComponent`，Hilt 会在相应 activity 的整个生命周期内提供 `AnalyticsAdapter` 的同一实例：

```kotlin
@ActivityScoped
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

> **注意**：将绑定的作用域限定为某个组件的成本可能很高，因为提供的对象在该组件被销毁之前一直保留在内存中。请在应用中尽量少用限定作用域的绑定。如果绑定的内部状态要求在某一作用域内使用同一实例，绑定需要同步，或者绑定的创建成本很高，那么将绑定的作用域限定为某个组件是一种恰当的做法。

假设 `AnalyticsService` 的内部状态要求每次都使用同一实例 - 不只是在 `ExampleActivity` 中，而是在应用中的任何位置。在这种情况下，将 `AnalyticsService` 的作用域限定为 `SingletonComponent` 是一种恰当的做法。结果是，每当组件需要提供 `AnalyticsService` 的实例时，都会提供同一实例。

以下示例演示了如何将绑定的作用域限定为 Hilt 模块中的某个组件。绑定的作用域必须与其安装到的组件的作用域一致，因此在本例中，必须将 `AnalyticsService` 安装在 `SingletonComponent` 中，而不是安装在 `ActivityComponent`中：

```kotlin
// If AnalyticsService is an interface.
@Module
@InstallIn(SingletonComponent::class)
abstract class AnalyticsModule {

  @Singleton
  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}

// If you don't own AnalyticsService.
@Module
@InstallIn(SingletonComponent::class)
object AnalyticsModule {

  @Singleton
  @Provides
  fun provideAnalyticsService(): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

如需详细了解 Hilt 组件作用域，请参阅 [Android 和 Hilt 中的作用域限定](https://medium.com/androiddevelopers/scoping-in-android-and-hilt-c2e5222317c0)。

> **注意**：如需详细了解使用 `@ActivityRetainedScoped` 或 `@ViewModelScoped` 限定作用域的区别，请参阅 [Hilt 和 Jetpack 集成文档](https://developer.android.google.cn/training/dependency-injection/hilt-jetpack?hl=zh-cn#viewmodelscoped)中的 `@ViewModelScoped` 部分。

### 组件层次结构

将模块安装到组件后，其绑定就可以用作该组件中其他绑定的依赖项，也可以用作组件层次结构中该组件下的任何子组件中其他绑定的依赖项：

![](images/Hilt/hilt-hierarchy.svg)**图 1.** Hilt 生成的组件的层次结构。

> **注意**：默认情况下，如果在视图中执行字段注入，`ViewComponent` 可以使用 `ActivityComponent` 中定义的绑定。如果你还需要使用 `FragmentComponent` 中定义的绑定并且视图是 fragment 的一部分，应将 `@WithFragmentBindings` 注解和 `@AndroidEntryPoint` 一起使用。

### 组件默认绑定

每个 Hilt 组件都附带一组默认绑定，Hilt 可以将其作为依赖项注入你自己的自定义绑定。请注意，这些绑定对应于常规 activity 和 fragment 类型，而不对应于任何特定子类。这是因为，Hilt 会使用单个 activity 组件定义来注入所有 activity。每个 activity 都有此组件的不同实例。

| Android 组件                | 默认绑定                                      |
| :-------------------------- | :-------------------------------------------- |
| `SingletonComponent`        | `Application`                                 |
| `ActivityRetainedComponent` | `Application`                                 |
| `ViewModelComponent`        | `SavedStateHandle`                            |
| `ActivityComponent`         | `Application`、`Activity`                     |
| `FragmentComponent`         | `Application`、`Activity` 和 `Fragment`       |
| `ViewComponent`             | `Application`、`Activity` 和 `View`           |
| `ViewWithFragmentComponent` | `Application`、`Activity`、`Fragment`、`View` |
| `ServiceComponent`          | `Application`、`Service`                      |

还可以使用 `@ApplicationContext` 获得应用上下文绑定。例如：

```kotlin
class AnalyticsServiceImpl @Inject constructor(
  @ApplicationContext context: Context
) : AnalyticsService { ... }

// The Application binding is available without qualifiers.
class AnalyticsServiceImpl @Inject constructor(
  application: Application
) : AnalyticsService { ... }
```

此外，还可以使用 `@ActivityContext` 获得 activity 上下文绑定。例如：

```kotlin
class AnalyticsAdapter @Inject constructor(
  @ActivityContext context: Context
) { ... }

// The Activity binding is available without qualifiers.
class AnalyticsAdapter @Inject constructor(
  activity: FragmentActivity
) { ... }
```

## 在 Hilt 不支持的类中注入依赖项

Hilt 支持最常见的 Android 类。不过，你可能需要在 Hilt 不支持的类中执行字段注入。

在这些情况下，你可以使用 `@EntryPoint` 注解创建入口点。入口点是由 Hilt 管理的代码与并非由 Hilt 管理的代码之间的边界。它是代码首次进入 Hilt 所管理对象的图的位置。入口点允许 Hilt 使用并非由 Hilt 管理的代码提供依赖关系图中的依赖项。

例如，Hilt 并不直接支持 [content provider](https://developer.android.google.cn/guide/topics/providers/content-providers?hl=zh-cn)。如果你希望 content provider 使用 Hilt 来获取某些依赖项，需要为所需的每个绑定类型定义一个带有 `@EntryPoint` 注解的接口并添加限定符。然后，添加 `@InstallIn` 以指定要在其中安装入口点的组件，如下所示：

```kotlin
class ExampleContentProvider : ContentProvider() {

  @EntryPoint
  @InstallIn(SingletonComponent::class)
  interface ExampleContentProviderEntryPoint {
    fun analyticsService(): AnalyticsService
  }

  ...
}
```

如需访问入口点，请使用来自 `EntryPointAccessors` 的适当静态方法。参数应该是组件实例或充当组件持有者的 `@AndroidEntryPoint` 对象。确保你以参数形式传递的组件和 `EntryPointAccessors` 静态方法都与 `@EntryPoint` 接口上的 `@InstallIn` 注解中的 Android 类匹配：

```kotlin
class ExampleContentProvider: ContentProvider() {
    ...

  override fun query(...): Cursor {
    val appContext = context?.applicationContext ?: throw IllegalStateException()
    val hiltEntryPoint =
      EntryPointAccessors.fromApplication(appContext, ExampleContentProviderEntryPoint::class.java)

    val analyticsService = hiltEntryPoint.analyticsService()
    ...
  }
}
```

在本例中，必须使用 `ApplicationContext` 检索入口点，因为入口点安装在 `SingletonComponent` 中。如果要检索的绑定位于 `ActivityComponent` 中，应改用 `ActivityContext`。

## Hilt 和 Dagger

Hilt 在依赖项注入库 [Dagger](https://dagger.dev/) 的基础上构建而成，提供了一种将 Dagger 纳入 Android 应用的标准方法。

关于 Dagger，Hilt 的目标如下：

- 简化 Android 应用的 Dagger 相关基础架构。
- 创建一组标准的组件和作用域，以简化设置、提高可读性以及在应用之间共享代码。
- 提供一种简单的方法来为各种 build 类型（如测试、调试或发布）配置不同的绑定。

由于 Android 操作系统会实例化它自己的许多框架类，因此在 Android 应用中使用 Dagger 要求你编写大量的样板。Hilt 可减少在 Android 应用中使用 Dagger 所涉及的样板代码。Hilt 会自动生成并提供以下各项：

- **用于将 Android 框架类与 Dagger 集成的组件** - 你不必手动创建。
- **作用域注解** - 与 Hilt 自动生成的组件一起使用。
- **预定义的绑定** - 表示 Android 类，如 `Application` 或 `Activity`。
- **预定义的限定符** - 表示 `@ApplicationContext` 和 `@ActivityContext`。

## 拓展资源

### Codelab

- [在 Android 应用中使用 Hilt](https://developers.google.cn/codelabs/codelabs/android-hilt/?hl=zh-cn)
- [将 Dagger 应用迁移到 Hilt](https://developers.google.cn/codelabs/codelabs/android-dagger-to-hilt/?hl=zh-cn)

### 博客

- [使用 Hilt 在 Android 上实现依赖项注入](https://medium.com/androiddevelopers/dependency-injection-on-android-with-hilt-67b6031e62d)
- [Android 和 Hilt 中的作用域限定](https://medium.com/androiddevelopers/scoping-in-android-and-hilt-c2e5222317c0)
- [向 Hilt 层次结构添加组件](https://medium.com/androiddevelopers/hilt-adding-components-to-the-hierarchy-96f207d6d92d)
- [将 Google I/O 大会应用迁移到 Hilt](https://medium.com/androiddevelopers/migrating-the-google-i-o-app-to-hilt-f3edf03affe5)



# 将 Hilt 和其他 Jetpack 库一起使用

Hilt 包含可用于从其他 Jetpack 库提供类的扩展。Hilt 目前支持以下 Jetpack 组件：

- `ViewModel`
- 导航
- Compose
- WorkManager

## 使用 Hilt 注入 ViewModel 对象

提供 [`ViewModel`](https://developer.android.google.cn/topic/libraries/architecture/viewmodel?hl=zh-cn)，方法是为其添加 `@HiltViewModel` 注解，并在 `ViewModel` 对象的构造函数中使用 `@Inject` 注解。

```kotlin
@HiltViewModel
class ExampleViewModel @Inject constructor(
  private val savedStateHandle: SavedStateHandle,
  private val repository: ExampleRepository
) : ViewModel() {
  ...
}
```

然后，带有 `@AndroidEntryPoint` 注解的 activity 或 fragment 可以使用 `ViewModelProvider` 或 `by viewModels()`[KTX 扩展](https://developer.android.google.cn/kotlin/ktx?hl=zh-cn) 获取 `ViewModel` 实例：

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() {
  private val exampleViewModel: ExampleViewModel by viewModels()
  ...
}
```

**注意**：如需将 Dagger 的辅助注入与 ViewModel 搭配使用，请参阅以下 [GitHub 问题](https://github.com/google/dagger/issues/2287)。

### @ViewModelScoped

所有 Hilt ViewModel 都由 `ViewModelComponent` 提供，后者遵循与 `ViewModel` 相同的生命周期，因此可以在配置更改后继续存在。如需将依赖项的作用域限定为 `ViewModel`，请使用 `@ViewModelScoped` 注解。

使用 `@ViewModelScoped` 类型后，系统会在注入 `ViewModel` 的所有依赖项中提供限定了作用域的类型的单个实例。请求限定了作用域的类型的 ViewModel 的其他实例会收到其他实例。

如果需要在不同 ViewModel 之间共享单个实例，则应使用 `@ActivityRetainedScoped` 或 `@Singleton` 限定其作用域。

## 与 Jetpack Navigation 库集成

请将下面这些额外的依赖项添加到 Gradle 文件中：

app/build.gradle

```kotlin
dependencies {
    ...
    implementation("androidx.hilt:hilt-navigation-fragment:1.0.0")
}
```

如果 `ViewModel` 的[作用域限定为导航图](https://developer.android.google.cn/guide/navigation/navigation-programmatic?hl=zh-cn#share_ui-related_data_between_destinations_with_viewmodel)，请使用 `hiltNavGraphViewModels` 函数，该函数可与带有 `@AndroidEntryPoint` 注解的 fragment 搭配使用。

```kotlin
val viewModel: ExampleViewModel by hiltNavGraphViewModels(R.id.my_graph)
```

## 使用 Hilt 注入 WorkManager

添加依赖到app/build.gradle

```kotlin
dependencies {
    implementation("androidx.hilt:hilt-work:1.0.0")
    // When using Kotlin.
    kapt("androidx.hilt:hilt-compiler:1.0.0")
    // When using Java.
    annotationProcessor("androidx.hilt:hilt-compiler:1.0.0")
}
```

1. 在类中使用 `@HiltWorker` 注解注入一个 [`Worker`](https://developer.android.google.cn/reference/kotlin/androidx/work/Worker?hl=zh-cn)
2. 在 `Worker` 对象的构造函数中使用 `@AssistedInject`。
3. 使用 `@Assisted`为 `Context` 和 `WorkerParameters` 依赖项添加注解
4. 只能在 `Worker` 对象中使用 `@Singleton` 或未限定作用域的绑定。

```kotlin
@HiltWorker
class ExampleWorker @AssistedInject constructor(
  @Assisted appContext: Context,
  @Assisted workerParams: WorkerParameters,
  workerDependency: WorkerDependency
) : Worker(appContext, workerParams) { ... }
```

最后，Application类实现 `Configuration.Provider` 接口，注入 `HiltWorkFactory` 的实例，并将其传入 `WorkManager` 配置，如下所示：

```kotlin
@HiltAndroidApp
class ExampleApplication : Application(), Configuration.Provider {

  @Inject lateinit var workerFactory: HiltWorkerFactory

  override fun getWorkManagerConfiguration() =
      Configuration.Builder()
            .setWorkerFactory(workerFactory)
            .build()
}
```

**注意**：由于这会自定义 `WorkManager` 配置，因此您还必须按照 [WorkManager 文档](https://developer.android.google.cn/topic/libraries/architecture/workmanager/advanced/custom-configuration?hl=zh-cn)中指定的方法，从 `AndroidManifest.xml`文件中移除默认的初始化程序。

> **警告**：`WorkManager` 版本 `2.6.0-alpha01` 或更高版本使用 `androidx.startup` 初始化程序。如需使用 Hilt 在此版本中正确配置 `WorkManager`，请查看 [`WorkManager` 版本说明](https://developer.android.google.cn/jetpack/androidx/releases/work?hl=zh-cn#2.6.0-alpha01)。

# Hilt 和 Dagger 注解备忘单

通过此备忘单，可以快速了解不同 Hilt 和 Dagger 注解的功能及其使用方法。备忘单也有[可以下载的 PDF 格式版本](https://developer.android.google.cn/static/images/training/dependency-injection/hilt-annotations.pdf?hl=zh-cn)。

| 注解                | 作用                                                         | 示例                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @HiltAndroidApp     | 使用Hilt必须包含一个带有 `@HiltAndroidApp` 注解的Application类 | class ExampleApplication : Application() { ... }             |
| @AndroidEntryPoint  | Hilt会为带有 `@AndroidEntryPoint` 注解的Android 类提供依赖项<br />  Android类包括： `ViewModel`（通过使用 `@HiltViewModel`） Activity、Fragment、 View、Service、BroadcastReceiver | @AndroidEntryPoint<br/>class ExampleActivity : AppCompatActivity() { ... } |
| @Inject             | 在方法前使用，表示需要通过Hilt提供（注入）依赖项<br />在类的构造函数中使用，告知 Hilt 如何提供该类的实例 | @AndroidEntryPoint<br/>class ExampleActivity : AppCompatActivity() {<br/><br/>  @Inject lateinit var analytics: AnalyticsAdapter<br/>  ...<br/>} |
| @Module             | 表示是一个 Hilt 模块，用于告知 Hilt 如何提供某些类型的实例。 |                                                              |
| @InstallIn          | 添加到 Hilt 模块上，告知 Hilt 此模块将用在哪个 Android 类中。 |                                                              |
| @Binds              | 用于注入接口实例。用在 Hilt 模块内的抽象函数上，告知 Hilt 在需要提供接口的实例时要使用哪种实现。 |                                                              |
| @Provides           | 用于 Hilt modules 中的函数。当依赖项是三方库中的类时，告知 Hilt 如何提供此类型的实例。 |                                                              |
| @Qualifier          | 用于提供同一类型、不同实现的依赖项                           |                                                              |
| @ApplicationContext | Hilt 提供的预定义的限定符。提供来自应用的Context。           |                                                              |
| @ActivityContext    | Hilt 提供的预定义的限定符。提供来自Activity的Context。       |                                                              |
| @EntryPoint         | 用于在content provider中使用 Hilt 来获取依赖项               |                                                              |

[![实用的 Dagger 和 Hilt 注解](images/Hilt/hilt-cheatsheet.png)](https://developer.android.google.cn/static/images/training/dependency-injection/hilt-cheatsheet.png?hl=zh-cn)







## Dagger

[官方：Dagger基础知识](https://developer.android.google.cn/training/dependency-injection/dagger-basics?hl=zh-cn)

# 参考

[官方：依赖项注入](https://developer.android.google.cn/training/dependency-injection?hl=zh-cn)