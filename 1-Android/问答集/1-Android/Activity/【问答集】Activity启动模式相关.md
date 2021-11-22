### 一、启动模式

<font color='orange'>Q5：Activity 为什么需要启动模式?</font>

一般情况下，每启动一个 Activity ，就会创建一个 Activity 实例并放到任务栈中。

（当点击返回键的时候，位于任务栈顶层的 Activity 就会被清理出去，当任务栈中不存在任何 Activity 实例后，系统就回去回收这个任务栈，程序结束。）

如果一个 Activity 会被频繁启动，就会生成很多这个 Activity 的实例，占用更多的内存。那在这种场景下，如果一个 Activity 实例就可以满足所有启动需求，就没有必要生成新的 Activity 实例，这样也能节省内存。因此，就引入了 Activity 的启动模式。

<font color='orange'>Q1：Activity的启动模式有哪些及应用场景?</font>

Activity 的启动模式有 4 种，分别是：standard、singleTop、singleTask 和 singleInstance。

1、系统默认的启动模式——Standard

默认情况下，都是Standard（标准模式）。在这种模式下，每启动一个 Activity 都会重新创建一个新的实例，不管这个实例是否存在。然后它会和启动它的Activity放到同一个任务栈中。

应用场景：默认情况都是。

2、栈顶复用模式——SingleTop

如果要启动的 Activity 已经位于任务栈的栈顶，那么此 Activity 不会被重新创建。但会调用它的 onNewIntent()（通过此方法的参数我们可以取出当前请求的信息）。并且不会调用onCreate、onStart()，因为它并没有发生改变。如果要启动的 Activity 已经存在但不是位于栈顶，那么新的 Activity 仍然会重新创建。

应用场景：当前的Activity中又要启动同类型的Activity，此时建议将此类型Activity的启动模式指定为SingleTop。比如消息、通知页面。这时候返回，一般情况就到了主页面，比如微信消息、其他应用通知。

3、栈内复用模式——SingleTask

单例实例模式。在这种模式下，只要 Activity 在一个栈中存在，那么多次启动此 Activity 也不会重新创建实例，跟 singleTop 一样，也会回调 onNewIntent()。并且会移除栈中在它上面的其他 Activity。

应用场景：主Activity一般用这个

4、单实例模式——SingleInstance

是加强的 singleTask 模式，它具有 singleTask 模式所有的特性，并且单独位于一个任务栈中。

应用场景：常用于系统应用，比如呼叫来电、闹钟、Launch、锁屏键的应用等等，整个系统中仅仅有一个。

<font color='orange'>Q2：情景题，A跳B，B跳C，C跳D，再一次回A要怎么回？?</font>

可以把 A Activity 的启动模式设置为 singleTask，这时候，直接启动 A Activity 就可以回到 A 中，并把其他 Activity  移除。

方法二：我们自己管理 Activity ，比如写个方法调用后把 BCD 的 Activity 全部 finish()掉。

<font color='orange'>Q3：如何查看当前的栈（task）中有哪些Activity？</font>



<font color='orange'>Q4：栈（task）的具体实现是怎样的？</font>

