> version：2021/9/27
>
> review：

[原文地址](https://developer.android.google.cn/guide/fragments/create)

目录



# 创建Fragment

fragment 表示activity中用户界面的一部分。fragment 有自己的生命周期，接收自己的输入事件，您可以在容器activity 运行时添加或移除fragments 。

本文档描述如何创建fragment 并将其添加到 activity 中。



## 配置你的环境

Fragments 需要依赖于 [AndroidX Fragment library](https://developer.android.google.cn/jetpack/androidx/releases/fragment)。您需要将[Google Maven](https://developer.android.google.cn/studio/build/dependencies#google-maven) 存储库添加到项目的build.gradle文件中，以便包含此依赖项。

```groovy
buildscript {
    ...

    repositories {
        google()
        ...
    }
}

allprojects {
    repositories {
        google()
        ...
    }
}
```

要将AndroidX Fragment库添加到项目中，请在应用程序的build.gradle文件中添加以下依赖项：

```groovy
dependencies {
    def fragment_version = "1.3.6"

    // Java language implementation
    implementation "androidx.fragment:fragment:$fragment_version"
    // Kotlin
    implementation "androidx.fragment:fragment-ktx:$fragment_version"
}
```

## 创建fragment类

创建一个继承自AndroidX [`Fragment`](https://developer.android.google.cn/reference/androidx/fragment/app/Fragment) 的类，并覆盖它的方法来插入你自己的应用程序逻辑，类似于创建 [`Activity`](https://developer.android.google.cn/reference/android/app/Activity)类的方式。要创建定义自己布局的最小fragment ，请向基本构造函数提供fragment 的布局资源，如下所示：

```kotlin
class ExampleFragment : Fragment(R.layout.example_fragment)
```

Fragment 库还提供了更专门的fragment 基类：

- [`DialogFragment`](https://developer.android.google.cn/reference/androidx/fragment/app/DialogFragment)

  显示浮动对话框。使用这个类来创建一个对话框是一个很好的选择，而不是在[`Activity`](https://developer.android.google.cn/reference/android/app/Activity) 类中使用对话框 helper 方法，因为fragments 会自动处理对话框的创建和清理。有关详细信息，请参阅 [Displaying dialogs with `DialogFragment`](https://developer.android.google.cn/guide/fragments/dialogs)。

- [`PreferenceFragmentCompat`](https://developer.android.google.cn/reference/androidx/preference/PreferenceFragmentCompat)

  将[`Preference`](https://developer.android.google.cn/reference/androidx/preference/Preference)对象的层次结构显示为列表。您可以使用PreferenceFragmentCompat为应用程序[create a settings screen](https://developer.android.google.cn/guide/topics/ui/settings) 。

## 将fragment 添加到activity中

通常，您的fragment 必须嵌入到AndroidX  [`FragmentActivity`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentActivity)中，以作为UI的一部分添加到activity的布局中。FragmentActivity是[`AppCompatActivity`](https://developer.android.google.cn/reference/androidx/appcompat/app/AppCompatActivity)的基类，因此嵌入到AppCompatActivity（提供向后兼容性）中也可以。

您可以在activity的布局文件中定义fragment或者fragment容器——然后通过编程方式从activity中添加fragment ，从而将fragment 添加到activity的view 层次结构中。在这两种情况下，您都需要添加一个[`FragmentContainerView`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentContainerView)，它定义了fragment 应该放在activity 的视图（view ）层次结构中的位置。强烈建议始终使用FragmentContainerView作为fragment 的容器，因为相较于其他view groups(容器，如FrameLayout)，FragmentContainerView包含对fragments的特定修复（或处理，原文fixes）。

### 通过XML添加片段

To declaratively add a fragment to your activity layout's XML, use a `FragmentContainerView`element.

Here's an example activity layout containing a single `FragmentContainerView`:

以声明方式将fragment 添加到activity 的XML布局中，使用FragmentContainerView元素（element）。下面是一个包含单个FragmentContainerView的示例activity 布局：

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.example.ExampleFragment" />
```

 `android:name` 属性指定要实例化的fragment的类名。当activity的布局inflate时，将实例化指定的fragment，重新实例化fragment时会回调 [`onInflate()`](https://developer.android.google.cn/reference/androidx/fragment/app/Fragment#onInflate(android.content.Context,%20android.util.AttributeSet,%20android.os.Bundle))，并创建FragmentTransaction将该fragment添加到FragmentManager中。

**Note:** You can use the `class` attribute instead of `android:name` as an alternative way to specify which `Fragment` to instantiate.

> 注意：您可以使用`class` 属性替代android：name来指定要实例化哪个`Fragment` 。

### 以编程的方式添加fragment 

To programmatically add a fragment to your activity's layout, the layout should include a `FragmentContainerView` to serve as a fragment container, as shown in the following example:

以编程的方式将fragment 添加到activity的布局中，布局应该包括一个FragmentContainerView作为fragment 容器，如下所示：

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

与XML方法不同，这里没有在`FragmentContainerView`上使用`android：name`属性，因此不会自动实例化任何特定的fragment。相反的，[`FragmentTransaction`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction)用于实例化一个片段并将其添加到activity的布局中。

当您的 activity 正在运行时，您可以进行fragment 事务处理，例如添加、删除或替换fragment 。在`FragmentActivity`中，您可以获得 [`FragmentManager`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentManager)的一个实例，该实例可用于创建`FragmentTransaction`。然后，可以使用[`FragmentTransaction.add()`](https://developer.android.google.cn/reference/androidx/fragment/app/FragmentTransaction#add(int, java.lang.Class, android.os.Bundle))方法在活动的onCreate()方法中实例化fragment，传入布局中容器的ID（fragment_container_view）和要添加的fragment类，然后提交事务，如下所示：

```kotlin
class ExampleActivity : AppCompatActivity(R.layout.example_activity) {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (savedInstanceState == null) {
            supportFragmentManager.commit {
                setReorderingAllowed(true)
                add<ExampleFragment>(R.id.fragment_container_view)
            }
        }
    }
}
```

> 注意：在执行FragmentTransaction时，应该始终使用setReorderingAllowed(True)。有关事务重排序的详细信息，请参阅 [Fragment transactions](https://developer.android.google.cn/guide/fragments/transactions#reordering)。

在前面的示例中，请注意，只有当SavedInstanceState==null时才创建fragment事务。这是为了确保只添加一次fragment，在第一次创建activity、当发生配置更改并重新创建活动时，SavedInstanceState不再为空，并且不需要第二次添加fragment，因为该fragment会自动从SavedInstanceState还原。

If your fragment requires some initial data, arguments can be passed to your fragment by providing a `Bundle` in the call to `FragmentTransaction.add()`, as shown below:

如果您的fragment 需要一些初始数据，可以在FragmentTransaction.add()的时候提供一个Bundle将参数传递给您的fragment ，如下所示：

```kotlin
class ExampleActivity : AppCompatActivity(R.layout.example_activity) {
      override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (savedInstanceState == null) {
            val bundle = bundleOf("some_int" to 0)
            supportFragmentManager.commit {
                setReorderingAllowed(true)
                add<ExampleFragment>(R.id.fragment_container_view, args = bundle)
            }
        }
    }
}
```

然后，可以通过调用 [`requireArguments()`](https://developer.android.google.cn/reference/androidx/fragment/app/Fragment#requireArguments())从fragment 中取出`Bundle` ，并且可以使用`Bundle` 的合适的getter方法取出每个参数。

```kotlin
class ExampleFragment : Fragment(R.layout.example_fragment) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val someInt = requireArguments().getInt("some_int")
        ...
    }
}
```

## 请参阅

Fragment transactions and the `FragmentManager` are covered in more detail in the [Fragment manager guide](https://developer.android.google.cn/guide/fragments/fragmentmanager).

Fragment 事务和FragmentManager将在[Fragment manager guide](https://developer.android.google.cn/guide/fragments/fragmentmanager)中详细介绍。



# 总结

1、fragment要使用Androidx提供的，其对原生的有了很多的优化，framework中的fragment已经不维护了。

2、添加dependencies的时候，把版本号抽取出来，方便替换。

3、官方推荐使用DialogFragment来创建对话框，因为fragment会自动处理对话框的创建和清理。

4、官方强烈建立使用FragmentContainerView作为fragment的容器，而非FrameLayout。在实际的项目中，用FrameLayout来作为fragment的容器还是很普遍的。

5、fragment会自动恢复

6、参数传递和提取

7、通过构造方法传入布局也是第一次见。

## 【精益求精】我还能做（补充）些什么？

1、链接中的点还可以补充学习。

