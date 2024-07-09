不要抱怨，积极学习，一路积累。



本文目标：自定义View入门。



1、自定义View简介

实现方式：extends View，extends ViewGroup。



2、三种构造方法的调用时机：

```java
public class TextView extends View {
    /**
     * new的时候调用。
     * <p>
     * TextView textView = new TextView(this);
     */
    public TextView(Context context) {
        super(context);
    }

    /**
     * 在布局（layout）中使用时调用。
     * <p>
     * <com.lizw.customview_01.TextView
     * android:layout_width="wrap_content"
     * android:layout_height="wrap_content"
     * android:text="Hello World" />
     */
    public TextView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    /**
     * 在布局layout中使用，并且使用了style时调用。
     * <p>
     * <com.lizw.customview_01.TextView
     * style="@style/Default"
     * android:text="Hello World" />
     */
    public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}

```



3、onMeasure()

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    // 指定布局的宽高
    // 指定控件的宽高，需要测量
    // 获取宽高的模式（wrap_content/match_parent/100dp）
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
}
```

宽高的三种模式：

```
MeasureSpec.AT_MOST: 在布局中指定了wrap_content
MeasureSpec.EXACTLY: 在布局中制定了确切的值  100dp  match_parent  fill_parent
MeasureSpec.UNSPECIFIED: 尽可能的大，很少用到。ListView+ScrollView。在测量子布局的的时候
会使用UNSPECIFIED。（会引起问题：ListView+ScrollView会显示不全，ListView只显示一行）
```



4、onDraw()

```java
/**
 * 用于绘制
 *
 * @param canvas
 */
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    canvas.drawText();
    //...
}
```



5、onTouchEvent()

```java
/**
     * 处理事件（触摸事件）、跟用户的交互
     *
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                Log.d("tag", "手指按下");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.d("tag", "手指移动");
                break;
            case MotionEvent.ACTION_UP:
                Log.d("tag", "手指抬起");
                break;
        }
        return super.onTouchEvent(event);
    }
```



6、自定义属性

android:layout_width="wrap_content"是系统的自定义属性。

6.1 在res下的values下面新建atts.xml

```java
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="TextView">
        <attr name="text" format="string"/>
        <attr name="textColor" format="color"/>
        <attr name="textSize" format="dimension"/>
        <attr name="maxLength" format="integer"/>
 <!--       <attr name="background" format="reference|color"/>    -->
    
        <attr name="inputType">
            <enum name="number" value="1"/>
            <enum name="text" value="2"/>
            <enum name="password" value="3"/>
        </attr>
    </declare-styleable>
</resources>
```

<attr name="background" format="reference|color"

不能使用background，因为background是在View中定义的，TextView继承自View，不能重复了。要使用直接android:background=""即可。

6.2 在布局中使用

声明自己的命名空间：

```
xmlns:app="http://schemas.android.com/apk/res-auto"
```

然后在自己的自定义View中使用：

```java
<com.lizw.customview_01.TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:text="Lizw"
    app:textColor="@color/colorAccent"
    app:textSize="20dp" />
```

6.3 在自定义View中获取属性

```java
private String mText;
private int mTextSize = 15;
private int mTextColor = Color.BLACK;

public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);

    // 获取自定义属性
    TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TextView);
    mText = array.getString(R.styleable.TextView_text);
    // 15  15px  15sp
    mTextSize = array.getDimensionPixelSize(R.styleable.TextView_textSize, mTextSize);
    mTextColor = array.getColor(R.styleable.TextView_textColor, mTextColor);

    array.recycle();
}
```

 

写于：2021/2/24
