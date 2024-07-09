# 什么是DataStore

DataStore是一种数据存储解决方案，允许使用协议缓存区来存储 key-value 值或者序列化对象。DataStore 使用 Kotlin 协程和 Flow 以异步、一致的事务方式存储数据。谷歌建议`DataStore` 替代 `SharedPreferences`。

> **注意**：如果您需要支持大型或复杂数据集、部分更新或参照完整性，请考虑使用 [Room](https://developer.android.google.cn/training/data-storage/room?hl=zh-cn)，而不是 DataStore。DataStore 非常适合简单的小型数据集，但不支持部分更新或引用完整性。

## 分类

`DataStore`一共有两种类型：`Preferences DataStore` 和 `Proto DataStore`。

- `Preferences DataStore`：使用键存储和访问数据。此实现不需要预定义的架构，不确保类型安全。
- `Proto DataStore`：将数据作为自定义数据类型的实例进行存储。要求使用协议缓冲区来定义架构，可以确保类型安全。

# 正确使用 DataStore

为了正确使用 DataStore，请始终谨记以下规则：

1. **请勿在同一进程中为给定文件创建多个 `DataStore` 实例**，否则会破坏所有 DataStore 功能。如果给定文件在同一进程中有多个有效的 DataStore 实例，DataStore 在读取或更新数据时将抛出 `IllegalStateException`。
2. **DataStore 的通用类型必须不可变**。更改 DataStore 中使用的类型会导致 DataStore 提供的所有保证都失效，并且可能会造成严重的、难以发现的 bug。强烈建议您使用可保证不可变性、具有简单的 API 和高效序列化的协议缓冲区。
3. **切勿对同一个文件混用 `SingleProcessDataStore` 和 `MultiProcessDataStore`**。如果打算从多个进程访问 `DataStore`，请始终使用 [`MultiProcessDataStore`](https://developer.android.google.cn/topic/libraries/architecture/datastore?hl=zh-cn#multiprocess)。

# Preferences DataStore

Preferences DataStore 使用 [`DataStore`](https://developer.android.google.cn/reference/kotlin/androidx/datastore/core/DataStore?hl=zh-cn) 和 [`Preferences`](https://developer.android.google.cn/reference/kotlin/androidx/datastore/preferences/core/Preferences?hl=zh-cn) 类将简单的键值对保留在磁盘上。

## 1、添加依赖

```kotlin
    // Preferences DataStore (SharedPreferences like APIs)
    dependencies {
        implementation "androidx.datastore:datastore-preferences:1.1.1"

        // 可选 - RxJava2 support
        implementation "androidx.datastore:datastore-preferences-rxjava2:1.1.1"

        // 可选 - RxJava3 support
        implementation "androidx.datastore:datastore-preferences-rxjava3:1.1.1"
    }
```

## 2、创建Preferences DataStore

使用 [`preferencesDataStore`](https://developer.android.google.cn/reference/kotlin/androidx/datastore/preferences/package-summary?hl=zh-cn#dataStore) 创建的属性委托来创建 `Datastore<Preferences>` 实例。在 Kotlin 文件顶层调用该实例一次，便可在应用的所有其余部分通过此属性访问该实例。这样可以更轻松地将 `DataStore` 保留为单例。必需的 `name` 参数是 Preferences DataStore 的名称。

```Kotlin
// 注意 Preferences 是下面这个类，不是 java.util.prefs.Preferences，导入错了会报错
import androidx.datastore.preferences.core.Preferences

// At the top level of your kotlin file:
// 命名：biz + DataStore
val Context.appDataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
```

如果使用 RxJava，通过 [`RxPreferenceDataStoreBuilder`](https://developer.android.google.cn/reference/kotlin/androidx/datastore/rxjava2/RxDataStoreBuilder?hl=zh-cn) 创建。

## 3、将内容写入Preferences DataSotre

Preferences DataStore 提供了一个 [`edit()`](https://developer.android.google.cn/reference/kotlin/androidx/datastore/preferences/core/package-summary?hl=zh-cn#edit) 函数，用于以事务方式更新 `DataStore` 中的数据，该函数的 `transform` 参数接受代码块，可以在其中根据需要更新值，块中的所有代码均被视为单个事务。

```Kotlin
class DatastoreDemoActivity : AppCompatActivity() {
    companion object {
        const val DATA_STORE_KEY_TEXT = "name"
    }

    // store data
    fun saveText(dataStore: DataStore<Preferences>, content: String) {
        lifecycleScope.launch(Dispatchers.IO) {
            val textKey = stringPreferencesKey(DATA_STORE_KEY_TEXT)
            dataStore.edit { settings ->
                settings[textKey] = content
            }
        }
    }
}
```

## 4、读取内容

为了读取内容，我们需要根据内容的属性使用特定的Key。下列代码是当内容为String的情况下，需要调用`stringPreferencesKey`方法。

```Kotlin
val textKey = stringPreferencesKey(MainActivity.DATA_STORE_TEXT_KEY)
```

然后把Key传入给DataStore的实例就可以读到数据内容了。

```Kotlin
class DatastoreDemoActivity : AppCompatActivity() {
    companion object {
        const val DATA_STORE_KEY_TEXT = "name"
    }

    // read data
    fun getText(dataStore: DataStore<Preferences>) {
        lifecycleScope.launch(Dispatchers.IO) {
            val textKey = stringPreferencesKey(DATA_STORE_KEY_TEXT)
            dataStore.edit { settings ->
                val text = settings[textKey]
                println(text)
            }
        }
    }
}
```







# Proto DataStore

## 添加依赖

```kotlin
    // Typed DataStore (Typed API surface, such as Proto)
    dependencies {
        implementation "androidx.datastore:datastore:1.1.1"

        // optional - RxJava2 support
        implementation "androidx.datastore:datastore-rxjava2:1.1.1"

        // optional - RxJava3 support
        implementation "androidx.datastore:datastore-rxjava3:1.1.1"
    }
```

`Proto DataStore`实现使用`DataStore`和协议缓冲区将类型化的对象保留在磁盘上。换句话来说，就是可以存储自定义类。

## 3.1 定义架构

首先，我们需要在路径为`app/src/main/proto`的目录下一个proto文件中创建预定义架构。关于具体的protobuf语言的使用方法，可以查看[这里](https://link.juejin.cn?target=https%3A%2F%2Fdevelopers.google.com%2Fprotocol-buffers%2Fdocs%2Fproto3)。

我写的例子中的预定义架构的代码如下。

```proto
proto复制代码syntax = "proto3";

option java_package = "com.example.datastoredemo";
option java_multiple_files = true;

message DataModelPreference {
  string name = 1;
  int32 age = 2;
}
```

简而言之，你需要修改`java_package`为你的项目路径，还需要写入你想要自定义的数据结构。

## 3.2 创建Data Model

我们需要创建一个与预定义结构中的数据结构相同的数据模型。如果想要让Proto DataStore中有默认的值，可以在Data Model中设置默认值即可。

```kotlin
kotlin复制代码data class DataModel(
    val name: String? = "1",
    val age: Int? = 1
)
```

## 3.3 创建Serializer

下一步，我们需要创建一个Serializer。我们要继承自`Serializer`同时需要重写一些方法。 具体的代码如下。 需要注意的是，Serializer中的`DataModelPreference`应该与预定义架构中定义的数据结构相同。

```kotlin
kotlin复制代码object DataModelSerializer : Serializer<DataModelPreference> {
    override fun readFrom(input: InputStream): DataModelPreference {
        try {
            return DataModelPreference.parseFrom(input)
        } catch (exception: InvalidProtocolBufferException) {
            throw CorruptionException("Cannot read proto.", exception)
        }
    }

    override fun writeTo(t: DataModelPreference, output: OutputStream) {
        t.writeTo(output)
    }

    override val defaultValue: DataModelPreference
        get() = DataModelPreference.getDefaultInstance()
}
```

## 3.4 创建DataStore的实例

利用`Context.createDataStore()`扩展函数去创建`DataStore<T>`的实例。
 当然，T是预定义架构中定义的那个数据类型`DataModelPreference`。
 往参数`fileName`中传入要保存的数据文件的名称，需要注意的是文件类型是`pb`.

```kotlin
private val datastore: DataStore<DataModelPreference> = createDataStore(
    fileName = PROTO_DATA_FILE_NAME,
    serializer = DataModelSerializer
)
```

## 3.5 从Proto DataStore中读取数据

利用`DataSotre.data`从存储的object中读取数据，以`Flow`形式返回。这里需要注意的是，一定要使用IO线程，不然会造成UI卡顿。

```kotlin
fun getText(dataStore: DataStore<DataModelPreference>) {
        viewModelScope.launch(Dispatchers.IO) {
            dataModelFlow = dataStore.data.map { pref ->
                Log.d("AAAA", "name: ${pref.name} age: ${pref.age}")
                DataModel(pref.name, pref.age)
            }
        }
    }
```

## 3.6 保存数据到Proto DataStore

利用`DataStore.update`方法来保存或者更新Proto DataStore中的数据。

```kotlin
fun getText(dataStore: DataStore<DataModelPreference>) {
        viewModelScope.launch(Dispatchers.IO) {
            dataModelFlow = dataStore.data.map { pref ->
                DataModel(pref.name, pref.age)
            }
        }
    }
```

#### 3.7 Screenshot

![1_ndGMoIE4A7p6wmCQzqUcmQ.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4db7d7d9333745a68defff200116ff3f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)



# 参考

[官方文档：DataStore](https://developer.android.google.cn/topic/libraries/architecture/datastore?hl=zh-cn)

[让你易上手的Jetpack DataStore教程](https://juejin.cn/post/6949561129630859272)