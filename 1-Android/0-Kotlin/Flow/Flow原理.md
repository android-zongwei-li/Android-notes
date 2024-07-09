# Flow

采用观察者模式设计的接口。

```kotlin
// Flow 是被观察者
public interface Flow<out T> {
    // 通过 collect() 的参数，将观察者提供给被观察者
    // 被观察者Flow 将在一定情况下，通过调用 观察者FlowCollector的emit() 将数据通知给 观察者FlowCollector
	public suspend fun collect(collector: FlowCollector<T>)
}

// FlowCollector 是观察者
public fun interface FlowCollector<in T> {
    public suspend fun emit(value: T)
}
```

# flow()分析

## 使用

```kotlin
            val testFlow = flow<String> {
                emit("hello")
            }
			// 跟上面是一样的，不使用 lambda 写法
			// flow() 需要传入一个 FlowCollector 的拓展方法，在这个拓展方法中可以将数据变化通知给观察者
			// 推测：最终调用 f 方法的就是 collect 传入的 FlowCollector，因此调用emit()后可以执行到 FlowCollector的emit方法内
            val f: suspend FlowCollector<String>.() -> Unit = {
                emit("hello")
            }
            flow<String>(f)

            // 下面是接收
            testFlow.collect { value ->
                println(value)
            }

            // collect 不使用 lambda 的写法，更加直观一点
            testFlow.collect(object : FlowCollector<String> {
                override suspend fun emit(value: String) {
                    println(value)
                }
            })
```

## 源码分析

```kotlin
public fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> = SafeFlow(block)

private class SafeFlow<T>(private val block: suspend FlowCollector<T>.() -> Unit) : AbstractFlow<T>() {
    // 第 0 步：实现抽象方法 collectSafely
    override suspend fun collectSafely(collector: FlowCollector<T>) {
        // 第 4 步，调用 collector.block()，在 block() 中使用再调用 collector.emit()
        collector.block()
    }
}

public abstract class AbstractFlow<T> : Flow<T>, CancellableFlow<T> {
	// 第一步：调用 collect() 时，传入 FlowCollector
    public final override suspend fun collect(collector: FlowCollector<T>) {
        val safeCollector = SafeCollector(collector, coroutineContext)
        try {
            // 第二步：将 FlowCollector 传给具体子类
            collectSafely(safeCollector)
        } finally {
            safeCollector.releaseIntercepted()
        }
    }

    public abstract suspend fun collectSafely(collector: FlowCollector<T>)
}
```

