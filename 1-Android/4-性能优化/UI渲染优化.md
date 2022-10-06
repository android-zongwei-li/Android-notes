> version：2022/10/6
>
> review：



目录

[TOC]



# 关键词



# 一、前置知识



# 二、UI渲染优化

理解工作中常用的UI渲染性能优化及调试方法对于我们编写高质量代码也是很有帮助的



## CPU、GPU的职责

大多数手机的屏幕刷新频率是60hz，也就是如果在1000/60=16.67ms内没有把这一帧的任务执行完毕，就会发生丢帧的现象，丢帧是造成界面卡顿的直接原因，渲染操作通常依赖于两个核心组件：CPU与GPU。CPU负责包括Measure，Layout等计算操作，GPU负责Rasterization(栅格化)操作(所谓栅格化就是将矢量图形转换为位图的过程，手机上显示是按照一个个像素来显示的，栅格化再普通一些的说法就是将一个Button,TextView等组件拆分到一个个像素上去显示)。

UI渲染优化的目的就是减轻CPU,GPU的压力，除去不必要的操作，保证每帧都在16ms以内处理完所有的CPU与GPU的计算，绘制，渲染等等操作，使UI顺滑，流畅的展示出来。



## 查找Overdraw

Overdraw(过度绘制)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在重叠的UI布局中，如果不可见的UI也在做绘制的操作或者后一个控件将前一个控件遮挡，会导致某些像素区域被绘制了多次，从而增加了CPU,GPU的压力。

那么如何找出布局中Overdraw的地方呢？

一般手机里面开发者选项都有调试GPU过度绘制的开关，打开即可。

以小米4手机为例，依次找到`设置->更多设置->开发者选项->调试GPU过度绘制开关`，打开就可以了。

打开调试GPU过度绘制开关之后，再次回到自己开发的应用发现界面怎么多了一些花花绿绿的玩意，没错，不同的颜色代表过度绘制的程度，具体如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180420104948203-194071555.png)

蓝色，淡绿，淡红，深红代表了4种不同程度的Overdraw情况，1x,2x,3x,4x分别表示同一像素上同一帧的时间内被绘制了多次，1x就表示一次(最理想情况)，4x表示4次(最差的情况)，我们要做的就是尽量减少3x,4x的情况出现。

下面以一个简单demo来进一步说明一下，如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180426174522016-785166152.jpg)

很简单的功能，功能做完了，能不能做下优化呢？打开OverDraw功能，再次查看界面，如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180426175106185-12883163.jpg)

怎么大部分都是浅绿色呢？也就是说同一像素上同一帧的时间内被绘制了2次，这是怎么回事？这时我们需要看下UI布局了，看哪些地方可以优化一下。

主界面布局如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent">

<ListView
android:id="@+id/list_view"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:divider="#F1F1F1"
android:dividerHeight="1dp"
android:background="@android:color/white"
android:scrollbars="vertical">
</ListView>

</RelativeLayout>
```

ListView每个条目布局如下：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="52dp"
android:background="@drawable/ts_account_list_selector">

<TextView
android:id="@+id/ts_item_has_login_account"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_marginLeft="10dp"
android:layout_marginTop="4dp"
android:gravity="center"
android:text="12345678999"
android:textColor="@android:color/black"
android:textSize="16sp" />

<LinearLayout
android:layout_width="wrap_content"
android:layout_height="20dp"
android:layout_alignParentBottom="true"
android:layout_marginBottom="3dp"
android:layout_marginLeft="10dp"
android:gravity="center_vertical" >

<ImageView
android:id="@+id/ts_item_time_clock_image"
android:layout_width="12dp"
android:layout_height="12dp"
android:src="@mipmap/ts_login_clock" />

<TextView
android:id="@+id/ts_item_last_login_time"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_marginLeft="5dp"
android:layout_toRightOf="@id/ts_item_time_clock_image"
android:text="上次登录"
android:textColor="@android:color/darker_gray"
android:textSize="11sp" />

<TextView
android:id="@+id/ts_item_login_time"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_marginLeft="5dp"
android:layout_toRightOf="@id/ts_item_last_login_time"
android:text="59分钟前"
android:textColor="@android:color/darker_gray"
android:textSize="11sp" />
</LinearLayout>

<TextView
android:id="@+id/ts_item_always_account_image_tips"
android:layout_width="wrap_content"
android:layout_height="13dp"
android:layout_alignParentRight="true"
android:layout_marginTop="2dp"
android:background="@mipmap/ts_always_account_bg"
android:gravity="center"
android:text="常用"
android:textColor="@android:color/white"
android:textSize="9sp" />

<ImageView
android:id="@+id/ts_item_delete_account_image"
android:layout_width="12dp"
android:layout_height="12dp"
android:layout_alignParentRight="true"
android:layout_marginTop="2dp"
android:layout_marginRight="13dp"
android:layout_centerVertical="true"
android:src="@mipmap/ts_close" />

</RelativeLayout>
```


发现哪里有问题了吗？问题在于ListView多余设置了背景：`android:background="@android:color/white"`，设置此背景对于我们这个需求根本就没有用，显示不出来并且增加GPU额外压力，去掉ListView背景之后再次观察如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180427104105612-986672707.jpg)

渲染性能提升了一个档次，在实际工作中情况会复杂很多，为了实现一个效果会不得不牺牲性能，这就需要自己团队权衡了。



## clipRect解决自定义View的OverDraw

写自定义View的时候有时会重写onDraw()，但是Android系统是无法检测onDraw里面具体会执行什么操作，从而系统无法为我们做一些优化。这样对编程人员要求就高了，如果我们自己写的View有大量重叠的地方就造成了CPU,GPU资源的浪费，但是我们可以通过`canvas.clipRect()`来帮助系统识别那些可见的区域。这个方法可以指定一块矩形区域，只有在这个区域内才会被绘制，其他的区域会被忽视，下面我们通过谷歌提供的一个小demo进一步说明。实现效果如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180427134333601-937366791.png)

主要就是卡片重叠效果，优化前代码实现如下：

DroidCard类封装要绘制的一个个卡片的信息：

```java
public class DroidCard {

public int x;//左侧绘制起点
public int width;
public int height;
public Bitmap bitmap;

public DroidCard(Resources res,int resId,int x){
this.bitmap = BitmapFactory.decodeResource(res,resId);
this.x = x;
this.width = this.bitmap.getWidth();
this.height = this.bitmap.getHeight();
}
}
```

DroidCardsView为真正的自定义View:

```java
public class DroidCardsView extends View {
//图片与图片之间的间距
private int mCardSpacing = 150;
//图片与左侧距离的记录
private int mCardLeft = 10;

private List<DroidCard> mDroidCards = new ArrayList<DroidCard>();

private Paint paint = new Paint();

public DroidCardsView(Context context) {
super(context);
initCards();
}

public DroidCardsView(Context context, AttributeSet attrs) {
super(context, attrs);
initCards();
}
/**

* 初始化卡片集合
  */
  protected void initCards(){
  Resources res = getResources();
  mDroidCards.add(new DroidCard(res,R.drawable.alex,mCardLeft));

mCardLeft+=mCardSpacing;
mDroidCards.add(new DroidCard(res,R.drawable.claire,mCardLeft));

mCardLeft+=mCardSpacing;
mDroidCards.add(new DroidCard(res,R.drawable.kathryn,mCardLeft));
}

@Override
protected void onDraw(Canvas canvas) {
super.onDraw(canvas);
for (DroidCard c : mDroidCards){
drawDroidCard(canvas, c);
}
invalidate();
}

/**

* 绘制DroidCard
  */
  private void drawDroidCard(Canvas canvas, DroidCard c) {
  canvas.drawBitmap(c.bitmap,c.x,0f,paint);
  }
  }
```

我们打开overdraw开关，效果如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180427135142525-619373111.png)

淡红色区域明显被绘制了三次（三张图片重合的地方），其实下面的图片完全没必要完全绘制，只需要绘制三分之一即可，接下来我们就需要对其优化，保证最下面两张图片只需要绘制其三分之一最上面图片完全绘制出来就可。

DroidCardsView代码优化为：

```java
public class DroidCardsView extends View {

//图片与图片之间的间距
private int mCardSpacing = 150;
//图片与左侧距离的记录
private int mCardLeft = 10;

private List<DroidCard> mDroidCards = new ArrayList<DroidCard>();

private Paint paint = new Paint();

public DroidCardsView(Context context) {
super(context);
initCards();
}

public DroidCardsView(Context context, AttributeSet attrs) {
super(context, attrs);
initCards();
}
/**

* 初始化卡片集合
  */
  protected void initCards(){
  Resources res = getResources();
  mDroidCards.add(new DroidCard(res, R.drawable.alex,mCardLeft));

mCardLeft+=mCardSpacing;
mDroidCards.add(new DroidCard(res, R.drawable.claire,mCardLeft));

mCardLeft+=mCardSpacing;
mDroidCards.add(new DroidCard(res, R.drawable.kathryn,mCardLeft));
}

@Override
protected void onDraw(Canvas canvas) {
super.onDraw(canvas);
for (int i = 0; i < mDroidCards.size() - 1; i++){
drawDroidCard(canvas, mDroidCards,i);
}
drawLastDroidCard(canvas,mDroidCards.get(mDroidCards.size()-1));
invalidate();
}

/**

* 绘制最后一个DroidCard
* @param canvas
* @param c
  */
  private void drawLastDroidCard(Canvas canvas,DroidCard c) {
  canvas.drawBitmap(c.bitmap,c.x,0f,paint);
  }

/**

* 绘制DroidCard
* @param canvas
* @param mDroidCards
* @param i
  */
  private void drawDroidCard(Canvas canvas,List<DroidCard> mDroidCards,int i) {
  DroidCard c = mDroidCards.get(i);
  canvas.save();
  canvas.clipRect((float)c.x,0f,(float)(mDroidCards.get(i+1).x),(float)c.height);
  canvas.drawBitmap(c.bitmap,c.x,0f,paint);
  canvas.restore();
  }
  }
```

主要就是使用Canvas的clipRect方法，绘制之前裁剪出一个区域，这样绘制的时候只在这区域内绘制，超出部分不会绘制出来。

重新执行程序，效果如下：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180427140830588-1561624230.png)

处理后性能就提升了一丝丝。

【还不懂】此外我们还可以使用`canvas.quickReject`方法来判断是否没和某个矩形相交，从而跳过那些非矩形区域内的绘制操作。



## Hierarchy Viewer的使用

Hierarchy Viewer可以很直观的呈现布局的层次关系。我们可以通过红，黄，绿三种不同的颜色来区分布局的Measure，Layout，Executive的相对性能表现如何

提升布局性能的关键点是尽量保持布局层级的扁平化，避免出现重复的嵌套布局。如果我们写的布局层级比较深会严重增加CPU的负担，造成性能的严重卡顿，关于Hierarchy Viewer的使用举例这里就不列举了。



## 内存抖动现象

在优化过view的树形结构和overdraw之后，可能还是感觉自己的app有卡顿和丢帧，或者滑动慢：卡顿还是存在。这时我们就要查看一下是否存在内存抖动情况了

Android有自动管理内存的机制，但是对内存的不恰当使用仍然容易引起严重的性能问题。在同一帧里面创建过多的对象是件需要特别引起注意的事情，在同一帧里创建大量对象可能引起GC的不停操作，执行GC操作的时候，所有线程的任何操作都会需要暂停，直到GC操作完成。大量不停的GC操作则会显著占用帧间隔时间。

如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了，严重时可能引起卡顿：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180427152217272-597281776.png)

导致GC频繁操作有两个主要原因：

1. 内存抖动，所谓内存抖动就是短时间产生大量对象又在短时间内马上释放。
2. 短时间产生大量对象超出阈值，内存不够，同样会触发GC操作。

观察内存抖动我们可以借助android studio中的工具，3.0以前可以使用android monitor,3.0以后被替换为android Profiler。

如果工具里面查看到短时间发生了多次内存的涨跌，这意味着很有可能发生了内存抖动，如图：

![](images/UI%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96/794139-20180427153938668-1405877845.png)

为了避免发生内存抖动，我们需要避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外，自定义View中的onDraw方法也需要引起注意，每次屏幕发生绘制以及动画执行过程中，onDraw方法都会被调用到，避免在onDraw方法里面执行复杂的操作，避免创建对象。对于那些无法避免需要创建对象的情况，我们可以考虑对象池模型，通过对象池来解决频繁创建与销毁的问题，但是这里需要注意结束使用之后，需要手动释放对象池中的对象。



# 布局常见问题与优化建议

1）没有用的父布局时指没有背景绘制或者没有大小限制的父布局，这样的布局不会对UI效果产生任何影响。我们可以把没有用的父布局，通过<merge/>标签合并来减少UI的层次；

2）使用线性布局LinearLayout排版导致UI层次变深，如果有这类问题，我们就使用相对布局RelativeLayout代替LinearLayout,减少UI的层次；

3）不常用的UI被设置成GONE,比如异常的错误页面，如果有这类问题，我们需要用<ViewStub/>标签，代替GONE提高UI性能。











# FAQ：常见问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、





# 【精益求精】我还能做（补充）些什么？

## 自我提问

Q：



# 脑图



# 参考

