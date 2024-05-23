> version：2021/11/03
>
> review：



目录

[TOC]



# 一、前言

本文档主要介绍了各种微优化，如果将其配合使用，能够提高应用的整体性能；但是，这些更改不太可能对性能产生显著影响。选择正确的算法和数据结构应始终是您的首要任务，但此内容不在本文档的讨论范围内。您应该将本文档中的提示作为编码时的一般做法并养成习惯，从而提高综合代码效率。

编写高效代码有两个基本规则：

- **不需要做的工作就不要做。**
- **如果可以避免，就不要分配内存。**

在微优化 Android 应用时，您会遇到的最棘手的问题之一是应用肯定会在多种类型的硬件上运行。不同版本的虚拟机会在不同的处理器上以不同的速度运行。通常并不能简单地用一句话“设备 X 比设备 Y 快/慢 F 倍”概括，也不能把由一台设备产生的结果扩展到其他设备上。特别是，在模拟器上得出的测量结果很难说明在任何设备上的性能。有 JIT 的设备和没有 JIT 的设备之间也存在巨大差异：最适合有 JIT 的设备的代码并不一定最适合没有 JIT 的设备。

为了确保应用在各种设备上都能达到很好的性能，请确保代码在所有级别都很高效，并积极优化应用性能。



# 二、性能优化Tips

## 1、避免创建不必要的对象:star:

创建对象绝不是没有成本的。带有针对临时对象的线程级分配池的分代垃圾回收器可以降低分配成本，但**分配内存的成本总是要高于不分配内存**。

随着您在应用中分配越来越多的对象，您会强制进行定期垃圾回收，导致用户体验出现小“问题”。Android 2.3 中引入的并发垃圾回收器对此有所帮助，但始终应该避免不必要的工作。

因此，您应该避免创建不需要的对象实例。以下是对您有所帮助的一些措施的示例：

- 如果您有一个返回字符串的方法，并且您知道其结果无论如何都会附加到某个 `StringBuffer`，则更改签名和实现，以便函数直接进行附加，而非创建短期的临时对象。
- 从一组输入数据中提取字符串时，请尝试返回原始数据的子字符串，而非创建副本。您会创建一个新的 `String` 对象，但它会与这些数据共享 `char[]`。（需要权衡的是，如果您只使用原始输入中的一小部分，那么如果您采用这种方法，便会将原始输入全部保留在内存中。）

有一个更激进的想法是，将多维数组切片为并行的单维数组：

- 一个 `int` 数组比一个 `Integer` 对象数组好得多，但这同样可以归纳成如下原则：两个并行的 int 数组也会比一个 `(int,int)` 对象数组的效率**高得多**。原语类型的任意组合都是如此。
- 如果您需要实现存储 `(Foo,Bar)` 对象元组的容器，请尽量记住，两个并行 `Foo[]` 和 `Bar[]` 数组的效果通常比单个自定义 `(Foo,Bar)` 对象的数组好很多。（当然，如果您要设计 API 以供其他代码访问，那就另当别论了。在这些情况下，通常建议在速度方面做一点妥协，从而实现良好的 API 设计。但在您自己的内部代码中，您应该尝试让代码尽可能高效。）

**一般来说，要尽量避免创建短期临时对象。创建的对象数量越少，就意味着垃圾回收频率越低，而这会直接影响用户体验。**



## 2、静态优先于虚拟（多态）:star:

如果您不需要访问某个对象的字段，则将相应方法设为静态。**调用速度会提高大约 15%-20%。**这也是一种很好的做法，因为**根据方法签名就能确定调用此方法不会更改对象的状态。**



## 3、对常量使用 static final:star:

下面是位于类顶部的声明：

```java
    static int intVal = 42;
    static String strVal = "Hello, world!";
```

编译器会生成一个名为 `<clinit>` 的类初始化器方法，当第一次使用该类时，系统会执行此方法。此方法会将值 42 存储到 `intVal`，并从类文件字符串常量表中提取 `strVal` 的引用。以后引用这些值时，可以通过**查询字段**访问它们。

我们可以使用“final”关键字加以改进：

```java
    static final int intVal = 42;
    static final String strVal = "Hello, world!";
```

此类不再需要 `<clinit>` 方法，因为常量会进入 dex 文件中的静态字段初始化器。引用 `intVal` 的代码将直接使用整数值 42，并且对 `strVal` 的访问将使用成本相对较低的**“字符串常量”指令**，而非字段查询。

> **注意**：此优化仅适用于原语类型和 `String` 常量，不适用于任意引用类型。尽管如此，最好还是尽可能声明常量 `static final`。



## 4、使用增强型 for 循环语法:star:

对于实现 `Iterable` 接口的集合以及数组，可以使用增强型 `for` 循环（有时也称为“for-each”循环）。对于集合，系统会分配迭代器以对 `hasNext()` 和 `next()` 进行接口调用。**对于 `ArrayList`，手写计数循环的速度快约 3 倍（有或没有 JIT）**，但对于其他集合，增强型 for 循环语法与使用显式迭代器完全等效。

遍历数组有以下几种替代方案：

```java
    static class Foo {
        int splat;
    }

    Foo[] array = ...

    public void zero() {
        int sum = 0;
        for (int i = 0; i < array.length; ++i) {
            sum += array[i].splat;
        }
    }

    public void one() {
        int sum = 0;
        Foo[] localArray = array;
        int len = localArray.length;

        for (int i = 0; i < len; ++i) {
            sum += localArray[i].splat;
        }
    }

    public void two() {
        int sum = 0;
        for (Foo a : array) {
            sum += a.splat;
        }
    }
```

`zero()` 速度最慢，因为 JIT 还无法消除每次循环迭代都要获取数组长度这项成本。

`one()` 速度较快。它会将所有内容都提取到局部变量中，避免查询。只有数组长度方面具有性能优势。

对于没有 JIT 的设备，`two()` 速度最快；对于具有 JIT 的设备，two() 与 **one()** 速度难以区分。two() 使用了在 1.5 版 Java 编程语言中引入的增强型 for 循环语法。

因此，您应默认使用增强型 `for` 循环，但对于性能关键型 `ArrayList` 迭代，不妨考虑使用手写计数循环。

> **提示**：另请参阅 Josh Bloch 的《Effective Java》第 46 条。



## 5、对于私有内部类，考虑使用包访问权限，而非私有访问权限

请查看以下类定义：

```java
    public class Foo {
        private class Inner {
            void stuff() {
                Foo.this.doStuff(Foo.this.mValue);
            }
        }

        private int mValue;

        public void run() {
            Inner in = new Inner();
            mValue = 27;
            in.stuff();
        }

        private void doStuff(int value) {
            System.out.println("Value is " + value);
        }
    }
```

对于上述代码，需要注意的是，我们定义了一个私有内部类 (`Foo$Inner`)，它会直接访问外部类中的私有方法和私有实例字段。这是合乎规则的，并且代码会按预期输出“Value is 27”。

问题在于，虚拟机认为从 `Foo$Inner` 直接访问 `Foo` 的私有成员不符合规则，因为 `Foo` 和 `Foo$Inner` 属于不同的类，虽然 Java 语言允许内部类访问外部类的私有成员。为了消除这种差异，编译器会生成一些合成方法：

```java
    /*package*/ static int Foo.access$100(Foo foo) {
        return foo.mValue;
    }
    /*package*/ static void Foo.access$200(Foo foo, int value) {
        foo.doStuff(value);
    }
```

每当需要访问外部类中的 `mValue` 字段或调用外部类中的 `doStuff()` 方法时，内部类代码就会调用这些静态方法。这意味着以上代码实际上可以归结为一种情况，那就是您通过访问器方法访问成员字段。之前我们讨论了访问器的速度比直接访问字段要慢，因此这是一个特定习惯用语会对性能产生“不可见”影响的示例。

如果您在性能关键位置 (hotspot) 使用这样的代码，则可以将内部类访问的字段和方法声明为拥有包访问权限（而非私有访问权限），从而避免产生相关开销。遗憾的是，这意味着同一软件包中的其他类可以直接访问这些字段，因此不应在公共 API 中使用此方法。



## 6、避免使用浮点数:star:

**一般来讲，在 Android 设备上，浮点数要比整数慢约 2 倍。**

在速度方面，`float` 和 `double` 在更现代的硬件上没有区别。在空间方面，`double` 所占空间大 2 倍。对于台式机，假定空间不是问题，您应该优先使用 `double`，而非 `float`。

此外，即使对于整数，某些处理器拥有硬件乘法器，却缺少硬件除法器。在这种情况下，整数的除法和取模运算会在软件中执行；如果您要设计哈希表或要进行大量数学运算，则需要考虑这一点。



## 7、了解和使用库:star:

除了优先使用库代码（而不是自行编写）的所有常见原因之外，请注意，系统可以自由地用手动汇编替换对库方法的调用，这可能比 JIT 能够为等效 Java 生成的最佳代码效果更好。这种情况的典型示例是 `String.indexOf()` 以及相关 API，Dalvik 会使用内嵌的内建函数替换它们。同样，**在具有 JIT 的 Nexus One 上，`System.arraycopy()` 方法的速度比手动编码的循环快约 9 倍。**

> **提示**：另请参阅 Josh Bloch 的《Effective Java》第 47 条。



## 8、谨慎使用原生方法

使用 [Android NDK](https://developer.android.google.cn/tools/sdk/ndk) 利用原生代码开发应用不一定比使用 Java 语言编程更高效。首先，Java-原生转换存在一定的成本，并且 JIT 无法在这些范围外进行优化。如果您要分配原生资源（原生堆上的内存、文件描述符或任何其他元素），那么安排对这些资源进行及时回收就可能会困难得多。您还需要针对要在其中运行的每个架构编译代码（而非依赖于其有 JIT）。您可能还需要为您认为相同的架构编译多个版本：为 G1 中的 ARM 处理器编译的原生代码无法充分利用 Nexus One 中的 ARM，而为 Nexus One 中的 ARM 编译的代码也无法在 G1 中的 ARM 上运行。

原生代码主要适用于您想要将现有原生代码库移植到 Android 的情况，而不适用于对 Android 应用中使用 Java 语言编写的部分进行“加速”。

如果您确实需要使用原生代码，请参阅我们的 [JNI 提示](https://developer.android.google.cn/guide/practices/jni)。

> **提示**：另请参阅 Josh Bloch 的《Effective Java》第 54 条。



## 9、性能误区

在没有 JIT 的设备上，通过**具有确切类型的变量来调用方法的确比通过接口进行调用效率略高。**（例如，在 `HashMap map` 上调用方法比在 `Map map` 上成本更低，尽管在这两种情况下，映射都是 `HashMap`。）速度并不会慢 2 倍；实际差异只是慢一点，例如 6%。此外，JIT 也让二者在效率方面难以区分。

在没有 JIT 的设备上，缓存字段访问的速度比重复访问字段快约 20%。如果有 JIT，则字段访问的成本与本地访问大致相同，因此除非您认为这样做能够让代码更易于阅读，否则不值得进行这样的优化。（final 字段、static 字段和 static final 字段也是如此。）

> 这个和多态之间要进行平衡，多态也就意味着接口，但是效率却要低于确切的类型。



## 10、始终衡量性能:star:

在开始优化之前，请先找到需要解决的问题。**请确保您可以准确衡量现有性能，否则便无法衡量所尝试的替代方案带来的优势。**

您还可以使用 [Traceview](https://developer.android.google.cn/tools/debugging/debugging-tracing) 分析性能，但务必注意，**Traceview 目前会停用 JIT**，这可能会导致其将时间成本错误地归因于启用 JIT 后也许能够消除的代码。在实施 Traceview 数据所建议的更改后，请确保所产生的代码在没有 Traceview 的情况下运行时速度确实更快了。

如需获取更多关于分析和调试应用的帮助，请参阅以下文档：

- [使用 Traceview 和 dmtracedump 分析性能](https://developer.android.google.cn/tools/debugging/debugging-tracing)
- [系统跟踪概览](https://developer.android.google.cn/topic/performance/tracing)





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

1、新概念

JIT、《Effective Java》



# 脑图



# 参考

1、[官方文档-性能提示](https://developer.android.google.cn/training/articles/perf-tips)

