# 一、GestureDetector手势检测

当用户触摸屏幕的时候会产生许多手势，如 down、up、scroll、fling 等。

View 类有 View.OnTouchListener 内部接口，通过重写它的 onTouch(View v, MotionEvent event) 方法可以处理 touch 事件，但是这个函数太过简单，如果要处理些复杂手势，使用这个接口就会很麻烦。

为此，Android SDK 提供了 GestureDetector （手势检测）类，通过这个类可以识别很多手势。在识别出手势之后，具体的事务处理则由程序员自己来实现。

GestureDetector 类对外提供了两个接口 OnGestureListener、OnDoubleTapListener 和一个外部类SimpleOnGestureListener。这个外部类其实是两个接口中所有函数的集成，它包含了这两个接口里所有必须实现的函数 ，而且都己经被重写，但所有函数体都是空的。该类是个静态类 ，程序员可以在外部继承这个类重写里面的手势函数。



## 1、GestureDetector.OnGesturelistener 接口

如果我们写个类并继承自 nGestureListener ，则会提示有几个必须重写的函数代码

```java

```



## 2、GestureDetector.OnDoubleTaplistener 接口



## 3、GestureDetector.SimpleOnGesturelistener 类



#### 4、onFling() 函数的应用一一识到是向左渭还是向石渭
