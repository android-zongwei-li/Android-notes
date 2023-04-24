#### 一、px、dpi、dp、sp等单位的概念

##### 1、px

1px代表屏幕上的一个物理像素点。

##### 2、dpi

dpi：dots per inch（每英寸像素数），由设备决定，是**固定的**，可以通过*context.getResources().getDisplayMetrics().densityDpi* 获取。

也可以通过以下方法算出：

​	dpi = 横向分辨率/横向英寸数 = 纵向分辨率/纵向英寸数

Google 规定的屏幕密度列表：

- *ldpi* (low) ~120dpi
- *mdpi* (medium) ~160dpi
- *hdpi* (high) ~240dpi
- *xhdpi* (extra-high) ~320dpi
- *xxhdpi* (extra-extra-high) ~480dpi
- *xxxhdpi* (extra-extra-extra-high) ~640dpi

示例：

1.5英寸\*2英寸的手机屏幕，分辨率为240*320，那么dpi = 240/1.5 = 160。

##### 3、dp

dip：device independent pixels（设备独立像素），dp与dip一样，不同的设备有不同的显示，不依赖像素。

不同的手机/平板可能具有不同的像素密度。例如同为4寸手机，有480x320分辨率的，也有800x480分辨率的，前者的像素密度就比较低。Android系统定义了几种像素密度：低（120dpi）、中（160dpi）、高（240dpi）和超高（320dpi），它们对应的dp到px的系数分别为0.75、1、1.5和2，这个系数乘以dp长度就是像素数。

即**px = dp * （dpi / 160）**。

**metrics.density = dpi/160**

例如界面上有一个长度为“80dp”的图片，那么它在240dpi的手机上实际显示为80x1.5=120px，在320dpi的手机上实际显示为80x2=160px。根据dpi的定义，也可以算出这张图片占了0.5英寸。这样就起到了适配的作用。

##### 4、sp

scale-independent pixels（缩放独立像素）。它和dp很相似，但唯一的区别在于，Android系统允许用户自定义文字尺寸大小（小，正常，大，超大等），当文字尺寸是“正常”时，1sp=1dp=0.00625inch（英寸），当文字尺寸是“大”或“超大”时，1sp>1dp=0.00625inch。

类似我们在windows里调整字体尺寸以后的效果——窗口大小不变，只有文字大小改变。

**px = sp * (dpi / 160)**

*疑问：怎么设置文字尺寸大小（小，正常，大，超大）*

示例：

```xml
<TextView 
    android:id="@+id/textView1" 
    style="@android:style/TextAppearance.Small" 
    android:layout_width="wrap_content" 
    android:layout_height="wrap_content" 
    android:text="Sample Text - Small" /> 
```

```xml
中：
style="@android:style/TextAppearance.Medium" 
大：
style="@android:style/TextAppearance.Large" 
```



其他：

##### in：

英寸，1英寸 = 2.54厘米（约）。



#### 二、相关Api

1、获取DisplayMetrics对象方法：

```java
DisplayMetrics dm = new DisplayMetrics();
//获得DisplayMetrics对象方法一
//dm = context.getResources().getDisplayMetrics();
//获得DisplayMetrics对象方法二
((Activity)context).getWindowManager().getDefaultDisplay().getMetrics(dm);
```

2、其他单位代码转px（TypedValue#applyDimension）

```java
public static float applyDimension(int unit, float value,
   DisplayMetrics metrics)
   {
 switch (unit) {
 case COMPLEX_UNIT_PX:
     return value;
 case COMPLEX_UNIT_DIP:
     return value * metrics.density;
 case COMPLEX_UNIT_SP:
     return value * metrics.scaledDensity;
 case COMPLEX_UNIT_PT:
     return value * metrics.xdpi * (1.0f/72);
 case COMPLEX_UNIT_IN:
     return value * metrics.xdpi;
 case COMPLEX_UNIT_MM:
     return value * metrics.xdpi * (1.0f/25.4f);
 }
 return 0;
    }
```

3、

```java
	/** 
     * 根据手机的分辨率从 px(像素) 的单位 转成为 dp 
     */  
    public static int px2dip(Context context, float pxValue) {  
        final float scale = context.getResources().getDisplayMetrics().density;  
        return (int) (pxValue / scale + 0.5f);  
    }
```

4、

```java
	/** 
     * 根据手机的分辨率从 dp 的单位 转成为 px(像素) 
     */  
    public static int dip2px(Context context, float dpValue) {  
        final float scale = context.getResources().getDisplayMetrics().density;  
        return (int) (dpValue * scale + 0.5f);  
    } 
```

*疑问：为什么要 +0.5f？*

已知：px = dp * scale，但由于scale是float类型，因此px可能是4.1、4.9，那么强转为int后的值都为4，显然4.9变为4差值多大。

所以为了解决这个问题，采用+0.5f的方式来处理，比如：

4.1+0.5=4.6，int为4；

4.5+0.5=5.0、4.9+0.5=4.5，int为5；

差值较小，精度较高。





写于：2021/2/25

文档归类：基础概念



复习：

2021/2/26