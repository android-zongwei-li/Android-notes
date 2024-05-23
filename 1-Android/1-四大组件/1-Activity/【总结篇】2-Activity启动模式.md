> 知识路径：Android > Activity
>
> version：2021/4/14
>
> review：2021/4/14
>
> 掌握程度：初学



# 前言

- `Android`基础中，`Activity`的启动模式非常重要
- 本文将全面介绍 `Activity`的启动模式

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-7bfcc8a15b5678eb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

目录

------

# 1. 定义

即Activity启动的方式

------

# 2. 启动模式的类别

Android启动提供了四种启动方式：

- 标准模式（`Standard`）
- 栈顶复用模式（`SingleTop`）
- 栈内复用模式（`SingleTask`）
- 单例模式（`SingleInstance`）

------

# 3. 知识储备

- `Activity`的管理方式 = **任务栈**
- 任务栈 采用的结构 = “后进先出” 的栈结构
- 每按一次Back键，就有一个`Activity`出栈

![img](https:////upload-images.jianshu.io/upload_images/944365-3b0c6f5564228c84.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

示意图

------

# 4. 具体介绍

- 如下图

![img](https:////upload-images.jianshu.io/upload_images/944365-8d121b25c3bf65e6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- 通俗讲解

  ![img](https:////upload-images.jianshu.io/upload_images/944365-5a53a70c672dcdbd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/793/format/webp)

  图解

------

# 5. 四种启动模式的区别

![img](https:////upload-images.jianshu.io/upload_images/944365-f8ff1efde0a29112.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 6. 启动模式的设置

启动模式有2种设置方式：在`AndroidMainifest`设置、通过`Intent`设置标志位

### 6.1 在AndroidMainifest设置

在`AndroidMainifest`的`Activity`配置进行设置



```cpp
<activity

android:launchMode="启动模式"
//属性
//standard：标准模式
//singleTop：栈顶复用模式
//singleTask：栈内复用模式
//singleInstance：单例模式
//如不设置，Activity的启动模式默认为**标准模式（standard）**
</activity>
```

### 6.2 通过`Intent`设置标志位



```java
Intent inten = new Intent (ActivityA.this,ActivityB.class);
intent,addFlags(Intent,FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

标记位属性

| 标记位属性                         |                             含义                             |
| ---------------------------------- | :----------------------------------------------------------: |
| FLAG_ACTIVITY_SINGLE_TOP           |          指定启动模式为栈顶复用模式（`SingleTop`）           |
| FLAG_ACTIVITY_NEW_TASK             |          指定启动模式为栈内复用模式（`SingleTask`）          |
| FLAG_ACTIVITY_CLEAR_TOP            | 所有位于其上层的Activity都要移除，`SingleTask`模式默认具有此标记效果 |
| FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS | 具有该标记的Activity不会出现在历史Activity的列表中，即无法通过历史列表回到该Activity上 |

### 6.3 二者设置的区别

- 优先级不同
   `Intent`设置方式的优先级 > `Manifest`设置方式，即 以前者为准
- 限定范围不同
   `Manifest`设置方式无法设定 `FLAG_ACTIVITY_CLEAR_TOP`；`Intent`设置方式 无法设置单例模式（`SingleInstance`）

------

# 7. 总结

- 本文全面介绍了 `Android`的4种 `Activity`启动模式





## 二、Activity 的启动模式

### 2.1 Android 任务栈

在说启动模式之前，先掌握一个概念：返回栈（任务栈）。

Android 中的 Activity 全都归属于 task 管理 。task 是多个 Activity 的集合， 这些 Activity 按照启动顺序排队存入一个栈（即“back stack”）。android 默认会为每个App维持一个task来存放该app的所有activity，task的默认name为该app的 packagename。

当然我们也可以在AndroidMainfest.xml中申明activity的taskAffinity属性来自定义task，但不建议使用，如果其他app也申明相同的task，它就有可能启动到你的activity，带来各种安全问题（比如拿到你的Intent）。关于taskAffinity属性和Android的安全问题，后续在探讨。

返回栈和生命周期的联系总结一下：
1、启动新活动，入栈，位于栈顶。
2、按下back键或调用finish()，栈顶活动出栈。
3、系统总是显示位于栈顶的活动。

任务栈与 Activity 的启动模式密不可分，它是用来存储 Activity 实例的一种数据结构，Activity 的跳转以及回跳都与这个任务栈有关。详情请看下面的 Activity 的启动模式。

### 2.2 Activity 的启动模式

**问题 1：Activity 为什么需要启动模式？**

我们都知道启动一个 Activity 后，这个 Activity 实例就会被放入任务栈中，当点击返回键的时候，位于任务栈顶层的 Activity 就会被清理出去，当任务栈中不存在任何 Activity 实例后，系统就回去回收这个任务栈，也就是程序退出了。

那么问题来了，既然每次启动一个 Activity 就会把对应的要启动的 Activity 的实例放入任务栈中，假如这个 Activity 会被频繁启动，那岂不是会生成很多这个 Activity 的实例吗？对内存而言这可不是什么好事，明明一个 Activity 实例就可以应付所有的启动需求，为什么要频繁生成新的 Activity 实例呢？为了杜绝这种内存浪费行为，Activity 的启动模式就被创造出来了。

**问题 2：Activity 的启动模式有哪些？特性如何**

Activity 的启动模式有 4 种，分别是：standard、singleTop、singleTask 和 singleInstance。下面一一作介绍：

1、系统默认的启动模式——Standard

默认情况下，都是Standard（标准模式）。在这种模式下，每启动一个 Activity 都会重新创建一个新的实例，不管这个实例是否存在。然后它会和启动它的Activity放到同一个任务栈中。

（拓展：有个注意的地方就是当我们用 ApplicationContext 去启动 standard 模式的 Activity 时会报错，这是因为 standard 模式的 Activity 默认会进入启动它的 Activity 所属的任务栈中，但是由于非 Activity 类型的 Context(如ApplicationContext)并没有任务栈，所以会出现错误。解决这个问题的方法就是为待启动的 Activity 指定 FLAG_ACTIVITY_NEW_TASK 标记位，这样启动的时候就会为它创建一个新的任务栈，这个时候启动 Activity 实际上以 singleTask 模式启动的。）

2、栈顶复用模式——SingleTop

在这种模式下，如果新的 Activity 已经位于任务栈的栈顶，那么此 Activity 不会被重新创建，同时它的 onNewIntent 方法被回调，通过此方法的参数我们可以取出当前请求的信息。需要注意的是，这个 Activity 的 onCreate、onStart 不会被系统调用，因为它并没有发生改变。如果新的 Activity 已经存在但不是位于栈顶，那么新的 Activity 仍然会重新创建。举个例子，假设目前栈内的情况为 ABCD，其中 ABCD 为四个 Activity，A 位于栈低，D 位于栈顶，这个时候假设要再次启动 D，如果 D 的启动模式为 singleTop，那么栈内的情况依然为 ABCD;如果 D 的启动模式为standard，那么由于 D 被重新创建，导致栈内的情况为 ABCDD。

3、栈内复用模式——SingleTask

这是一种单例实例模式，在这种模式下，只要 Activity 在一个栈中存在，那么多次启动此Activity 都不会重新创建实例，和 singleTop 一样，系统也会回调其 onNewIntent。具体一点，当一个具有 singleTask 模式的 Activity 请求启动后，比如 Activity A，系统首先寻找任务栈中是否已存在 Activity A 的实例，如果已经存在，那么系统就会把 A 调到栈顶并调用它的 onNewIntent 方法，如果 Activity A 实例不存在，就创建 A 的实例并把 A 压入栈中。举几个栗子：

- 比如目前任务栈 S1 的情况为 ABC，这个时候 Activity D 以 singleTask 模式请求启动，其所需的任务栈为 S2，由于 S2 和 D 的实例均不存在，所以系统会先创建任务栈S2，然后再创建 D 的实例并将其投入到 S2 任务栈中。
- 另外一种情况是，假设 D 所需的任务栈为 S1，其他情况如同上面的例子所示，那么由于 S1 已经存在，所以系统会直接创建 D 的实例并将其投入到 S1。 

- 如果 D 所需的任务栈为 S1，并且当前任务栈 S1 的情况为 ADBC，根据栈内复用的原则，此时 D 不会重新创建，系统会把 D 切换到栈顶并调用其 onNewIntent 方法，同时由于 singleTask 默认具有 clearTop 的效果，会导致栈内所有在 D 上面的 Activity全部出栈，于是最终 S1 中的情况为 AD。

通过以上 3 个例子，你应该能比较清晰地理解 singleTask 的含义了。

4、单实例模式——SingleInstance

这是一种加强的 singleTask 模式，它除了具有 singleTask 模式所有的特性外，还加强了一点，那就是具有此种模式的 Activity 只能单独位于一个任务栈中，换句话说，比如 Activity A 是 singleInstance 模式，当 A 启动后，系统会为它创建一个新的任务栈，然后 A 独自在这个新的任务栈中，由于栈内复用的特性，后续的请求均不会创建新的 Activity，除非这个独特的任务栈被系统销毁了。

对于 SingleInstance，面试时你要说明它的以下几个特点：

（1）以 singleInstance 模式启动的 Activity 具有全局唯一性，即整个系统中只会存在一个这样的实例。

（2）以 singleInstance 模式启动的 Activity 在整个系统中是单例的，如果在启动这样的 Activity 时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。

（3）以 singleInstance 模式启动的 Activity 具有独占性，即它会独自占用一个任务，被它开启的任何 Activity 都会运行在其他任务中。

（4）被 singleInstance 模式的 Activity 开启的其他 Activity，能够在新的任务中启动，但不一定开启新的任务，也可能在已有的一个任务中开启。

总结上面介绍了 4 种启动模式，这里需要指出一种情况，我们假设目前有 2 个任务栈，前台任务栈的情况为 AB，而后台任务栈的情况为 CD，这里假设 CD 的启动模式均为 singleTask。现在请求启动 D,那么整个后台任务栈都会被切换到前台，这个时候整个后退列表变成了 ABCD。当用户按 back 键的时候，列表中的 Activity 会一一出栈，如下图 1所示：

**注意：**

前台任务栈：就是指和用户正在交互的应用程序所在的任务栈。

后台任务栈：就是指处于后台的应用程序所在的任务栈。

![image-20210414165332360](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210414165332360.png)

如果不是请求的 D 而是请求的 C,那么情况就不一样了，如下图 2 所示：

![image-20210414165359183](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210414165359183.png)

**问题 3：如何给 Activity 选择合适的启动模式（应用场景）**

**1）standard （标准模式）**
描述：没有为Activity设置启动模式的话，默认为标准模式。每次启动一个Activity都会创建一个新的实例入栈，无论这个实例是否存在。
生命周期：每次被创建的实例Activity 的生命周期符合典型情况，它的onCreate、onStart、onResume都会被调用。
应用场景：默认都是。

**2）singleTop（栈顶复用模式）**
描述：如果当前启动的Activity在栈顶，则直接复用，否则创建一个新的实例。
生命周期：
如果栈顶的Activity被直接复用，由于它并没有发生改变，它的onCreate、onStart不会被系统调用，但是会回调一个新的方法 onNewIntent（Activity被正常创建时不会回调此方法）。
应用场景：
假设你在当前的Activity中又要启动同类型的Activity，此时建议将此类型Activity的启动模式指定为SingleTop
消息、通知页面

**3）singleTask（栈内复用模式）**
描述：如果要创建的Activity已经处于栈中时，则不会创建新的Activity，而是将存在栈中的Activity上面的其他Activity全部销毁，使它位于栈顶。
生命周期：如果在栈中，则直接复用，并回调onNewIntent方法。
应用场景：主activity一般用这个

**4）singleInstance（单实例模式）**
描述：singleInstance是全局单例模式，是一种加强的singleTask模式。它除了具有它所有特性外，还加强了一点：具有此模式的Activity只能单独位于一个任务栈中。
生命周期：如果栈存在，则复用。否则，新建一个栈，把Activity放入其中。
应用场景：经常用于系统中的应用，比如呼叫来电、闹钟、Launch、锁屏键的应用等等，整个系统中仅仅有一个。

### 2.3 启动模式的指定方式

1）在 Manifest.xml中指定Activity启动模式（静态）

```java
<activity android:name="..activity.MultiportActivity" android:launchMode="singleTask"/>
```

2）启动Activity时。在Intent中指定启动模式去创建Activity（动态）
在 new 一个 Intent 后，通过Intent的addFlags方法去动态指定一个启动模式。

```java
Intent intent = new Intent();
intent.setClass(context, MainActivity.class);
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);        
context.startActivity(intent);
```

二者的区别：
1）优先级：动态优先。若两者都存在，以addFlags为准。
2）限定范围：静态无法为Activity直接指定 FLAG_ACTIVITY_CLEAR_TOP 标识；动态无法为Activity指定 singleInstance 模式。

### 2.4 Activity 的 Flags（切勿死记，理解它）

1） FLAG_ACTIVITY_NEW_TASK
作用是为Activity指定 “SingleTask”启动模式。跟在AndroidMainfest.xml指定效果同样。
2） FLAG_ACTIVITY_SINGLE_TOP
作用是为Activity指定 “SingleTop”启动模式，跟在AndroidMainfest.xml指定效果同样。
3）FLAG_ACTIVITY_CLEAN_TOP
4）FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
具有此标记位的Activity不会出历史Activity的列表中。它等同于在xml中指定Activity的属性：android:excludeFromRecents="ture"

### 2.5 对SingleTop/SingleTask与onNewIntent的分析

我们需要考虑一个Activity跳转时携带页面參数的问题。如下：

由于当一个Activity设置了SingleTop或者SingleTask模式后，跳转此Activity出现复用原有Activity的情况时，此Activity的onCreate方法将不会再次运行。onCreate方法仅仅会在第一次创建Activity时被运行。而一般onCreate方法中会进行该页面的数据初始化、UI初始化，假设页面的展示数据与页面跳转传递的參数无关，则不必操心此问题；若页面展示的数据就是通过getIntent() 方法来获取的，那么问题就会出现：getInten()获取的一直都是老数据，根本无法接收跳转时传送的新数据！

示例：

```
<activity
            android:name=".activity.CourseDetailActivity"
            android:launchMode="singleTop"
            android:screenOrientation="portrait" />
```

```java
public class CourseDetailActivity extends BaseActivity{
  ......
  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_course_detail_layout);
        initData();
        initView();
    }
 
   //初始化数据
    private void initData() {
        Intent intent = getIntent();
        mCourseID = intent.getStringExtra(COURSE_ID);
    }
 
    //初始化UI
    private void initView() {
    ......
    }
    ......
}
```

以上代码中的CourseDetailActivity在配置文件里设置了启动模式是SingleTop模式，依据上面启动模式的介绍可得知，当CourseDetailActivity处于栈顶时。再次跳转页面到CourseDetailActivity时会直接复用原有的Activity，并且此页面须要展示的数据是从getIntent(）方法得来，可是initData()方法不会再次被调用，此时页面就无法显示新的数据。

这时我们需要另外一个回调 onNewIntent（Intent intent）方法。此方法会传入最新的intent，这样我们就能够解决上述问题。这里建议的方法是又一次去setIntent。然后又一次去初始化数据和UI。代码例如以下所看到的：

```java
    /*
    * 复用Activity时的生命周期回调
    */
    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        setIntent(intent);
        initData();
        initView();
    }
```

这样，在一个页面中能够反复跳转并显示不同的内容。
注：当调用到onNewIntent(intent)的时候，需要在onNewIntent() 中使用setIntent(intent)赋值给Activity的Intent。否则，后续的getIntent()都是得到老的Intent。

那么onNewIntent都会在什么情况下调用呢？
1）当ActivityA的LaunchMode为SingleTop时：如果ActivityA在栈顶,且现在要再启动ActivityA，这时会调用onNewIntent()方法 
2）当ActivityA的LaunchMode为SingleInstance,SingleTask时：如果已经ActivityA已经在堆栈中，那么此时会调用onNewIntent()方法 
3）当ActivityA的LaunchMode为Standard时：由于每次启动ActivityA都是启动新的实例，和原来启动的没关系，所以不会调用原来ActivityA的onNewIntent方法

我的理解是：只要启动的Activity被复用了，就会调用onNewIntent。因为经过我的测试，即使是standard，如果home后，点击启动按钮，还是会调用onNewIntent的。

### 2.6 Activity的状态与回收

知道了生命周期和返回栈后，就可以来探讨下活动的状态和回收了。

每个Activity在其生命周期中最多可能有四种状态。

| 活动状态    | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| 1、运行状态 | 位于栈顶。可交互。系统最不愿回收。                           |
| 2、暂停状态 | 不在栈顶，仍然可见。<br/>情形一：当活动跳转时，第一个活动会先调用onPause()，然后等待第二个活动显示onCreate() –》onStart()，在活动二从创建到显示的过程中，活动一处于暂停状态且可见。<br/>注：onPause()执行完成后，第二个活动才会onCreate(),所以onPause中执行的操作，要尽量快速，或者放到子线程中，以避免对UI线程的影响。<br/>情形二：启动一个非全屏的活动。<br/>比如启动了一个对话框形式的活动。暂停状态的活动还完全存活着，系统也不愿回收(它还可见，在内存极低的情况下，系统才会考虑回收)。 |
| 3、停止状态 | 不在栈顶，完全不可见。系统会保存相应的状态和成员变量，但不可靠。因为当其他地方需要内存时，有可能被回收。 |
| 4、销毁状态 | 移出返回栈。最倾向于回收。                                   |

注：回收，就是系统把一个对象从内存中去除，以保证内存充足。因为一个活动即使被finish()，回调了onDestory()，也**只是把活动移出管理栈，并没有置为null，还存在于内存中，等待GC**。





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





# 参考

1、[Android：四种启动模式分析](https://blog.csdn.net/elisonx/article/details/80397519)

[Carson带你学Android：最易懂的Activity启动模式详解](https://www.jianshu.com/p/399e83d02e33)