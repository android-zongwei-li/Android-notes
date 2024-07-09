学过一个东西后过一段时间又觉得自己什么也没学，往往是复习没到位引起的。



正文：

自定义View的**第一步**通常都是**创建构造方法**：

```java
public class TextView extends View {
    
    public TextView(Context context) {
        this(context, null);
    }

    public TextView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}
```

Step2：如果需要自定义属性，先定义，再在类中取出。

2.1 自定义属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="TextView">
        <attr name="text" format="string"/>
        <attr name="textColor" format="color|reference"/>
        <attr name="textSize" format="dimension"/>
        <attr name="maxLength" format="integer"/>

        <attr name="inputType">
            <enum name="number" value="1"/>
            <enum name="password" value="2"/>
        </attr>
    </declare-styleable>
</resources>
```

2.2 使用

通过context.obtainStyledAttributes()方法得到一个TypedArray。再通过TypedArray取到各属性的在xml中的值。如果xml中没有使用某属性，则会使用默认值。

```java
    private String mText;
    private int mTextColor = Color.BLACK;
    private int mTextSize = 29;

	public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);

    TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TextView);
    mText = array.getString(R.styleable.TextView_text);
    mTextColor = array.getColor(R.styleable.TextView_textColor, mTextColor);
    mTextSize = array.getDimensionPixelSize(R.styleable.TextView_textSize, mTextSize);

    array.recycle();
}
```

Step3：onMeasure()测量。

3.1 测量需要使用到Paint类。

```java
	private Paint mPaint;

    public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TextView);
        mText = array.getString(R.styleable.TextView_text);
        mTextColor = array.getColor(R.styleable.TextView_textColor, mTextColor);
        mTextSize = array.getDimensionPixelSize(R.styleable.TextView_textSize, mTextSize);

        array.recycle();

        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setTextSize(mTextSize);
        mPaint.setColor(mTextColor);
    }
```

3.2 测量

测量的目的是为了计算并设置此**控件的宽高**。设置方法为：setMeasuredDimension()。

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);

    int widthMode = MeasureSpec.getMode(widthMeasureSpec);

    int width = MeasureSpec.getSize(widthMeasureSpec);
    int height = MeasureSpec.getSize(heightMeasureSpec);

    if (widthMode == MeasureSpec.AT_MOST) {
    	Rect bounds = new Rect();
    	mPaint.getTextBounds(mText, 0, mText.length(), bounds);
        width = bounds.width() + getPaddingLeft() + getPaddingRight();
        height = bounds.height() + getPaddingTop() + getPaddingBottom();
    }

    setMeasuredDimension(width, height);
}
```

根据宽高Mode（模式）的不同，width的计算方式也是不同的。

1. 如果是EXACTLY（100sp/match_parent），则直接MeasureSpec.getSize()即可；
2. 如果是AT_MOST（wrap_content），测量的重点在于获取text文本的宽高，使用Paint#getTextBounds()方法进行获取。测量好的值，会被储存到传入的Rect()中。

Step4：绘制onDraw()。

通过canvas.drawText(mText, 0, baseline, mPaint);来绘制文本。

第一个参数：要绘制的text；

第二个参数：起始位置；

第三个参数：baseline，基线。这个很重要；

第四个参数：画笔Paint。

```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    Paint.FontMetricsInt fontMetrics = mPaint.getFontMetricsInt();
    int dy = (fontMetrics.bottom - fontMetrics.top) / 2 - fontMetrics.bottom;
    int baseline = getHeight() / 2 + dy;
    canvas.drawText(mText, 0, baseline, mPaint);
}
```

关于top，bottom，dy之间的关系如下图：

![image-20210225100031217](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210225100031217.png)

重点就在于计算dy，dy是文本中间线到baseline的距离。

(fontMetrics.bottom - fontMetrics.top)即文本占用的整个高度；

(fontMetrics.bottom - fontMetrics.top) / 2 就是高度的一半。

所以dy = 高度的一半 - fontMetrics.bottom。



到此，一个可以显示文本的简陋TextView就完成了。



附：

完整代码如下：

```java
public class TextView extends View {
    private String mText;
    private int mTextColor = Color.BLACK;
    private int mTextSize = 29;

    private Paint mPaint;

    public TextView(Context context) {
        this(context, null);
    }

    public TextView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TextView);
        mText = array.getString(R.styleable.TextView_text);
        mTextColor = array.getColor(R.styleable.TextView_textColor, mTextColor);
        mTextSize = array.getDimensionPixelSize(R.styleable.TextView_textSize, mTextSize);

        array.recycle();

        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setTextSize(mTextSize);
        mPaint.setColor(mTextColor);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = MeasureSpec.getSize(heightMeasureSpec);

        if (widthMode == MeasureSpec.AT_MOST) {
            Rect bounds = new Rect();
            mPaint.getTextBounds(mText, 0, mText.length(), bounds);
            width = bounds.width() + getPaddingLeft() + getPaddingRight();
            height = bounds.height() + getPaddingTop() + getPaddingBottom();
        }

        setMeasuredDimension(width, height);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        Paint.FontMetricsInt fontMetrics = mPaint.getFontMetricsInt();
        int dy = (fontMetrics.bottom - fontMetrics.top) / 2 - fontMetrics.bottom;
        int baseline = getHeight() / 2 + dy;
        canvas.drawText(mText, 0, baseline, mPaint);
    }
}
```



写于：2021/2/25

文档归类：自定义View-实践-TextView。