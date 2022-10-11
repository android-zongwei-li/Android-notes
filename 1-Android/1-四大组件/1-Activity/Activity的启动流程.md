一、前言

最近在学习自定义View，要想更加深刻的理解View的相关知识（View的绘制流程、View的事件分发等），还是要从源头开始分析。那View的源头是什么呢？我注意到了Activity的顶层View是DecorView，因此现在可以先把问题缩小到：

1、**DecorView是如何加载到Activity的？是如何实例化的？**

2、要分析DecorView是如何加载到Activity的，首先还得解决一个问题：**首个Activity是如何启动（实例化）的？**现在的主要目标是分析第一个Activity（MainActivity）的实例化。以后会继续分析startActivity等情况。

3、其实照着这种思路往下，还能深入许多，比如App的启动流程。。。等等，这些知识点就等到我把前面两个问题解决后再来分析。

下面就开始分析这两个问题吧。



二、



当使用android:theme="@style/AppTheme"时的组件树：

![image-20210308145207466](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210308145207466.png)



setContentView(R.layout.activity_main)后的组件树结构：

![image-20210308141944540](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210308141944540.png)

