**注：对事件分发还需要深入研究。**



从onTouchEvent()方法出发，再次分析事件分发机制。



一、在自定义view的时候，经常会重写onTouchEvent()方法，如下：

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
}
```

onTouchEvent()对事件分发的影响：

![image-20210304110116529](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304110116529.png)

可以看到View#onTouchEvent()方法**恒定返回false**。



二、知道这个细节后，我们再来顺着看事件分发，从ViewGroup#dispatchTouchEvent()开始。

Step1、处理首个down事件

![image-20210304111308894](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304111308894.png)

本次分析先关注一个点：**resetTouchState()；**

![image-20210304111328012](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304111328012.png)



![image-20210304111430274](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304111430274.png)

小结：**处理首个down事件时，会把变量mFirstTouchTarget置为null**。



Step2、检查是否拦截

**![image-20210304111914670](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304111914670.png)**

我们还是把关注点放在**mFirstTouchTarget**上。

小结：当事件不为DOWN && **mFirstTouchTarget** **== null**时，**intercepted = true**;



![image-20210304143657139](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304143657139.png)



![image-20210304143716555](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210304143716555.png)



对Android源码的探索，要三番五次，每一次的角度不同，往往注意到的细节就会不同，理解也会不同，进而可以窥探其全貌。