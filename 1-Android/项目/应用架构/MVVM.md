

需求背景：我现在想要从数据库中加载数据，然后显示出来。

我的实现方案：通过 ViewModel + Repository + Room 实现。

ViewModel 中 用到 Repository，Repository 中用到了 Room，Room 初始化需要用到 applicationContext，但是官方又不推荐通过 ViewModel 传入 context。

问题：那应该怎样更好的把 applicationContext 传递给 Room 呢？

方案一：通过 ViewModel 传入 applicationContext 

```kotlin
class HighSchoolWordsRepository(application: Application) {
    private val appDatabase: AppDatabase = AppDatabase.getInstance(application)
```

```kotlin
        fun getInstance(context: Context): AppDatabase {
            if (::appDataBase.isInitialized.not()) {
                appDataBase = databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    DATABASE_NAME
                ).build()
            }
            return appDataBase
        }
```

方案二：通过 AndroidViewModel

方案三：在 data 模块提供一个初始化类，用于注入 applicationContext，这样在 data module 就可以完成 Room 初始化了。

方案六：通过 startup 注入

可能的缺点：在大量模块都采用这种方式初始化时，可能又会引起另一个问题——影响启动速度。

方案四：依赖注入框架，Hitl。

方案五：在 Application 类中初始化 Room

```kotlin
class MyApplication : Application() {  
  
    private val database: AppDatabase by lazy {  
        Room.databaseBuilder(  
            applicationContext,  
            AppDatabase::class.java,  
            "database-name"  
        ).build()  
    }  
  
    // 提供一个访问数据库的方法  
    fun getDatabase(): AppDatabase {  
        return database  
    }  
}
```

不推荐。缺点：难维护；侵入性强；

数据库的细节不需要所有模块都了解。通常只是特定的功能页面需要用到此数据库，这样写，就把数据库暴露给整个 App 了。对其他模块也是一种干扰。我们在实际项目中，往往是多人共同开发一个 App，基于这样的前提，我们的工作尽量不要对外部模块产生影响或干扰。 



# 最佳实践

这个场景最好的实践方式是什么呢？



选择技术方案时考虑的因素：

1、复杂度

2、可维护性

我们在评估可维护性时，一个好的方法是：思考下同样的功能与实现在新增 n 个以后会是怎样的情形？

在方案六中，如果有 10 个模块都用到了数据库，都在 Application 类中这样写会怎样？

3、可拓展性

举个我遇到的例子，App 接入埋点sdk，公司其他团队提供。这时如果直接在项目中使用sdk的方法，那遇到下一个需求——替换国内的埋点sdk，换成海外的 firebase sdk 时，怎么办？如果是直接在项目中使用，完了，替换起来很麻烦，影响面很大。好的做法是，接入这类 sdk 时，做一层封装隔离，这样后面替换时，只要在业务侧封装好的类中替换即可。

4、易用性

从 App 业务层面考虑，我们开发业务时，是不是更希望在使用 sdk 时，更加方便呢？能传一个参数的，不想传两个；能不传参数的最好。

5、侵入性

考虑对 App 层的影响。一般来说，影响越小越容易维护，大家各司其职，互不影响，在各自的模块工作运行。

6、是否达成共识

在引入新的技术时，要考虑团队的情况，可能某些人员还不太熟悉某个技术点；或者对某个技术点的使用有质疑。比如现在在用 flow 处理数据流，你突然又引入了 RxJava，可能就会引起质疑。

7、保持一致

如果整个项目都统一用了某种方案，尽量不要再引入类似的方案。比如如果已经选用了 LogUtils 类来输入日志，那新开发功能时，别在使用 Log 类；已经用了 Hilt，谨慎引入其他依赖注入库，比如 Koin。



思考：怎么开发、维护好一个 Android 项目？

怎么定义好？从开发角度来说，一是bug少、或者易维护（能快速定位修复bug），二是新增需求容易。