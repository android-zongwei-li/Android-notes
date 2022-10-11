本文中的示例 onCLick 的调用源码分析，另外写一篇文章详细分析：

源码分析——onClick 方法的调用.md



#### step1：使用 AS **自带虚拟机**进行源码 debug。

1. 自带虚拟机的 rom 可以保证代码没被修改过，可以跟源码的行数对得上。（真机、第三方虚拟机可能被修改过）。
2. 保证虚拟机 API 和 compileSdkVersion 版本一致。如虚拟机 API 30，那 compileSdkVersion 30

#### step2：打断点，debug 运行。

示例：

比如我们想要知道 onClick 的调用栈，可以在其方法内部打断点，如下：

![image-20210415103530923](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210415103530923.png)

此时 debug 运行后，点击按钮，就会执行到断点处。并且左下角会显示出调用栈等信息：

![image-20210415104126371](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210415104126371.png)

调用栈中的信息分析：

![image-20210415112236722](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210415112236722.png)

含义：

1. onClick 方法位于 MainActivity$1 中，MainActivity$1表示一个内部类。
2. onClick 是通过 performClick 调用的，performClick 方法位于 View 中。

![image-20210415112450730](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210415112450730.png)

含义：

run 位于 View$PerformClick 中。

run 通过 handleCallback 调用，handleCallback 位于 Handler中。

整体而言就是这样。

#### step3：查看调用栈

我们当前是想要知道 onClick 方法是如何被调用的，可以通过 **方向键上、下** 进行查看。

#### step4：如果我们需要查看 Toast 的源码呢？

可以通过 F7、F8、shift + F8 进行跟踪。

F7 ：进入

shift + F8 ：返回

F8 ：下一行



> 在实际调试中，源码行数的对应还是会有一定偏差，这里要根据实际情况进行寻找。

