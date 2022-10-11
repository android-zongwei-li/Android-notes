> 知识路径：Android > 2-View体系
>
> version：2021/4/22
>
> review：2021/4/22
>
> 掌握程度：初学



> 需要注意，如果能用系统控件还是应尽量用系统控件。自定义 View 毕竟不是规范的控件，如果设计不好、不考虑性能反而会适得其反，另外适配起来可能也会产生问题。



学习自定义 View 之前需要了解 View的层次、View 的事件分发机制和 View 的工作流程。

自定义 View 可以分为三大类：

第一种是自定义 View，自定义 View 分为继承系统控件（比如 TextView）和继承 View 两种。

第二种是自定义 ViewGroup，自定义 ViewGroup 也分为继承 ViewGroup 和继承系统特定的 ViewGroup（比如 RelativeLayout）。

第三种是自定义组合控件。

接下来就分别介绍自定义 View 的使用方法。

### 1、继承系统控件的自定义 View

这种自定义 View 在系统控件的基础上进行拓展，一般是添加新的功能或者修改显示的效果，一般情况下在 onDraw()方法中进行处理。这里举一个简单的例子，写一个自定义 View，继承自 TextView：

```java
public class MyTextView extends androidx.appcompat.widget.AppCompatTextView {
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    public MyTextView(Context context) {
        this(context, null);
    }

    public MyTextView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        
        mPaint.setColor(Color.RED);
        mPaint.setStrokeWidth((float) 1.5);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float height = getHeight();
        canvas.drawLine(0, height / 2, getWidth(), height / 2, mPaint);
    }
}
```

这个自定义 View 继承 TextView，在 onDraw()方法中画了一条红色的横线（AppCompatTextView是官方提供来做适配的，即使使用的是 TextView 最终也会被转化为 AppCompatTextView，因此可以直接继承它）。接下来在布局中引用这个 MyTextView，代码如下所示：

```xml
    <com.lizw.viewsystemdemo.MyTextView
        android:layout_width="200dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:background="@color/colorPrimary" />
```

运行程序，效果如下：

![image-20210422110651128](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210422110651128.png)

### 2、继承 View 的自定义 View

和上面继承系统控件相比，继承 View 的自定义 View 实现起来要稍微复杂一些。不仅要实现 onDraw()方法，在实现过程中还要考虑到 wrap_content 属性以及 padding 属性的设置；为了方便配置自己的自定义 View，还会对外提供自定义的属性。另外，如果要改变触控的逻辑，还要重写 onTouchEvent()等触控事件的方法。按照上面的例子我们再写一个 RectView 类继承 View 来画一个正方形，代码如下所示：

#### 2.1、简单实现继承 View 的自定义 View

```java
public class RectView extends View {
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private int mColor = Color.RED;

    public RectView(Context context) {
        this(context, null);
    }

    public RectView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RectView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        mPaint.setColor(mColor);
        mPaint.setStrokeWidth((float) 1.5);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float width = getWidth();
        float height = getHeight();
        canvas.drawRect(0, 0, width, height, mPaint);
    }
}
```

在布局中引用 RectView：

```xml
    <com.lizw.viewsystemdemo.RectView
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center"
        android:layout_marginTop="50dp" />
```

运行程序，效果如下：

![image-20210422111452221](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210422111452221.png)

#### 2.2、对 padding 属性进行处理

修改布局文件，设置 padding 属性，如下所示：

```xml
    <com.lizw.viewsystemdemo.RectView
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:padding="50dp"
        android:layout_gravity="center"
        android:layout_marginTop="50dp" />
```

运行程序，发现没有任何作用。看来还得对 padding 属性进行处理。只需要在 onDraw()方法中稍加修改，在绘制正方形的时候考虑 padding 属性即可，代码如下所示：

```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float width = getWidth();
        float height = getHeight();

        int paddingLeft = getPaddingLeft();
        int paddingRight = getPaddingRight();
        int paddingTop = getPaddingTop();
        int paddingBottom = getPaddingBottom();

        canvas.drawRect(paddingLeft, paddingTop, width - paddingRight, height - paddingBottom, mPaint);
    }
```

运行程序，效果如下：



可以发现设置的 padding 属性确实生效了。

#### 2.3、对 wrap_content 属性进行处理

修改布局文件，让 RectView 的宽度分别为 wrap_content 和 match_parent 时的效果都是一样

的，如图所示：



为何一样，请看下面这段代码：

```java
    public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```

AT_MOST：wrap_content

EXACTLY：match_parent、确切值

对于这种情况需要我们在 onMeasure 方法中指定一个默认的宽和高，在设置 wrap_content 属性时设置此默认的宽和高就可以了：

```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(600, 600);
        } else if (widthMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(600, heightSpecSize);
        } else if (heightMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, 600);
        }
    }
```

需要注意的是 setMeasuredDimension()方法接收的参数单位是 px。

运行程序，查看效果：



#### 2.4、自定义属性

Android 系统的控件以 android 开头的（比如 android：layout_width）都是系统自带的属性。为了方便配置 RectView 的属性，我们也可以自定义属性。首先在 values 目录下创建 attrs.xml：

```java
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="RectView">
        <attr name="rect_color" format="color|reference" />
    </declare-styleable>
</resources>
```

这个配置文件定义了名为 RectView 的自定义属性组合。我们定义了 rect_color 属性，它的格式为 color|reference。接下来在 RectView 的构造方法中解析自定义属性的值，如下所示：

```java
    public RectView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.RectView);
        // 提取 RectView 属性集合的 rect_color 属性。如果没设置，默认值为 mColor
        mColor = array.getColor(R.styleable.RectView_rect_color, mColor);
        // 获取资源后要及时回收
        array.recycle();

        mPaint.setColor(mColor);
        mPaint.setStrokeWidth((float) 1.5);
    }
```

用 TypedArray 来获取自定义的属性集 R.styleable.RectView，这个 RectView 就是我们在XML 中定义的 name 的值，然后通过 TypedArray 的 getColor 方法来获取自定义的属性值。最后修改布局文件，如下所示：

```java
    <com.lizw.viewsystemdemo.RectView
        android:layout_width="wrap_content"
        android:layout_height="200dp"
        android:layout_gravity="center"
        app:rect_color="@color/colorPrimary"
        android:layout_marginTop="50dp" />
```

使用自定义属性需要添加 schemas：xmlns:app="http://schemas.android.com/apk/res-auto"，其中 app 是我们自定义的名字。最后我们配置新定义的 app：rect_color 属性为 android：color/colorPrimary。运行程序发现 RectView 的颜色变成了蓝色。RectView 的完整代码，如下所示：

```java
public class RectView extends View {
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private int mColor = Color.RED;

    public RectView(Context context) {
        this(context, null);
    }

    public RectView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RectView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.RectView);
        // 提取 RectView 属性集合的 rect_color 属性。如果没设置，默认值为 mColor
        mColor = array.getColor(R.styleable.RectView_rect_color, mColor);
        // 获取资源后要及时回收
        array.recycle();

        mPaint.setColor(mColor);
        mPaint.setStrokeWidth((float) 1.5);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(600, 600);
        } else if (widthMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(600, heightSpecSize);
        } else if (heightMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, 600);
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float width = getWidth();
        float height = getHeight();

        int paddingLeft = getPaddingLeft();
        int paddingRight = getPaddingRight();
        int paddingTop = getPaddingTop();
        int paddingBottom = getPaddingBottom();

        canvas.drawRect(paddingLeft, paddingTop, width - paddingRight, height - paddingBottom, mPaint);
    }
}
```

### 3、自定义组合控件

自定义组合控件就是多个控件组合起来成为一个新的控件，其主要用于解决多次重复地使用同一类型的布局。比如我们应用的顶部标题栏以及弹出的固定样式的 Dialog，这些都是常用的，所以把它们所需要的控件组合起来重新定义成一个新的控件。本节就来自定义一个顶部的标题栏。当然，实现标题栏有很多方法，下面来看看用自定义组合控件如何去实现。首先，我们定义组合控件的布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout_title_bar"
    android:layout_width="match_parent"
    android:layout_height="50dp">

    <ImageView
        android:id="@+id/iv_title_bar_left"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:paddingLeft="20dp"
        android:src="@mipmap/ic_launcher" />

    <TextView
        android:id="@+id/tv_title_bar_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="defaultText"
        android:textAllCaps="false" />

    <ImageView
        android:id="@+id/iv_title_bar_right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:paddingLeft="20dp"
        android:src="@mipmap/ic_launcher" />

</RelativeLayout>
```

这是很简单的布局，左右边各一个图标，中间是标题文字。接下来编写 Java 代码。因为我们的组合控件整体布局是 RelativeLayout，所以组合控件要继承 RelativeLayout，代码如下所示：

```java
public class TitleBar extends RelativeLayout {
    private ImageView iv_titlebar_left;
    private ImageView iv_titlebar_right;
    private TextView tv_titlebar_title;
    private RelativeLayout layout_titlebar_rootlayout;

    private int mBackgroundColor = Color.BLUE;
    private String mTitleName;
    private int mTextColor = Color.WHITE;

    public TitleBar(Context context) {
        this(context, null);
    }

    public TitleBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TitleBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context);
    }

    private void initView(Context context) {
        LayoutInflater.from(context).inflate(R.layout.layout_title_bar, this, true);
        iv_titlebar_left = findViewById(R.id.iv_title_bar_left);
        iv_titlebar_right = findViewById(R.id.iv_title_bar_right);
        tv_titlebar_title = findViewById(R.id.tv_title_bar_title);
        layout_titlebar_rootlayout = findViewById(R.id.layout_titlebar_rootlayout);
        
        layout_titlebar_rootlayout.setBackgroundColor(mColor);
        tv_titlebar_title.setTextColor(mTextColor);
    }

    public void setTitle(String titleName) {
        if (!TextUtils.isEmpty(titleName)) {
            tv_titlebar_title.setText(titleName);
        }
    }

    public void setLeftListener(OnClickListener onClickListener) {
        iv_titlebar_left.setOnClickListener(onClickListener);
    }

    public void setRightListener(OnClickListener onClickListener) {
        iv_titlebar_right.setOnClickListener(onClickListener);
    }
}
```

这里重写了 3 个构造方法并在构造方法中加载布局文件，对外提供了 3 个方法，分别用来设置标题的名字，以及左右按钮的点击事件。前面讲到了自定义属性，这里同样使用自定义属性，在 values 目录下创建 attrs.xml，代码如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="TitleBar">
        <attr name="titleTextColor" format="color" />
        <attr name="titleBackground" format="color" />
        <attr name="titleText" format="string" />
    </declare-styleable>
</resources>
```

我们定义了 3 个属性，分别用来设置顶部标题栏的背景颜色、标题文字颜色和标题文字。为了引入自定义属性，需要在 TitleBar 的构造方法中解析自定义属性的值，代码如下所示：

```java
    public TitleBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.TitleBar);
        mTextColor = array.getColor(R.styleable.TitleBar_titleTextColor, mTextColor);
        mTitleName = array.getString(R.styleable.TitleBar_titleText);
        mBackgroundColor = array.getColor(R.styleable.TitleBar_titleBackground, mBackgroundColor);
        array.recycle();

        initView(context);
    }
```

接下来引用组合控件的布局。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <com.lizw.viewsystemdemo.TitleBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:titleBackground="@color/colorAccent"
        app:titleText="默认标题"
        app:titleTextColor="@color/colorPrimaryDark" />

</LinearLayout>
```

最后，在主界面调用自定义的 TitleBar，并设置了左右两边按钮的点击事件：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TitleBar titleBar = findViewById(R.id.title_bar);
        titleBar.setLeftListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "left", Toast.LENGTH_SHORT).show();
            }
        });
        titleBar.setRightListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "right", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

程序运行，效果如下：

![image-20210422142656683](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210422142656683.png)

当然也可以对点击事件做下优化，将其统一到同一个接口中，先在TitleBar中新增：

```java

    private void initView(Context context) {
        LayoutInflater.from(context).inflate(R.layout.layout_title_bar, this, true);
        iv_titlebar_left = findViewById(R.id.iv_title_bar_left);
        iv_titlebar_right = findViewById(R.id.iv_title_bar_right);
        tv_titlebar_title = findViewById(R.id.tv_title_bar_title);
        layout_titlebar_rootlayout = findViewById(R.id.layout_titlebar_rootlayout);
        layout_titlebar_rootlayout.setBackgroundColor(mBackgroundColor);
        tv_titlebar_title.setTextColor(mTextColor);

        iv_titlebar_left.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                mOnTitleBarClickListener.onLeftClick();
            }
        });

        iv_titlebar_right.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                mOnTitleBarClickListener.onRightClick();
            }
        });
    }

	private OnTitleBarClickListener mOnTitleBarClickListener;

    public interface OnTitleBarClickListener {
        void onLeftClick();

        void onRightClick();
    }

    public void setOnTitleBarClickListener(OnTitleBarClickListener onTitleBarClickListener) {
        mOnTitleBarClickListener = onTitleBarClickListener;
    }
```

然后删除：

```java
    public void setLeftListener(OnClickListener onClickListener) {
        iv_titlebar_left.setOnClickListener(onClickListener);
    }

    public void setRightListener(OnClickListener onClickListener) {
        iv_titlebar_right.setOnClickListener(onClickListener);
    }
```

主界面调用自定义的 TitleBar，并设置了点击事件：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TitleBar titleBar = findViewById(R.id.title_bar);
        titleBar.setOnTitleBarClickListener(new TitleBar.OnTitleBarClickListener() {
            @Override
            public void onLeftClick() {
                Toast.makeText(MainActivity.this, "left", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onRightClick() {
                Toast.makeText(MainActivity.this, "right", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

这样代码简洁一点，也达到了同样的效果。

### 4、自定义 ViewGroup

前面介绍过自定义 ViewGroup 又分为继承 ViewGroup 和继承系统特定的 ViewGroup（比如RelativeLayout），下面主要介绍继承 ViewGroup。本节的例子是一个自定义的 ViewGroup，左右滑动切换不同的页面，类似一个特别简化的 ViewPager。其会涉及本章很多节的内容，比如 View 的工作流程、View 的滑动等。需要注意的是，要实现一个自定义的 ViewGroup 是很复杂的，这一点看 LineraLayout 等的源码就会知道，在此只实现主要的功能就好了。

#### 4.1、继承 ViewGroup

要实现自定义的 ViewGroup，首先要继承 ViewGroup 并调用父类的构造方法，实现抽象方法等：

```java
public class HorizontalView extends ViewGroup {
    public HorizontalView(Context context) {
        super(context);
    }

    public HorizontalView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public HorizontalView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        
    }
}
```

这里我们定义了名为 HorizontalView 的类并继承 ViewGroup，onLayout 这个抽象方法是必须要实现的，我们暂且什么都不做。

#### 4.2、对 wrap_content 属性进行处理

至于为什么要对 wrap_content 属性进行处理，上面也遇到过类似情况，就是measure的原因。接下来在 onMeasure 中对 wrap_content 属性进行处理：

```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        measureChildren(widthMeasureSpec, heightMeasureSpec);
        // 如果没有子元素，就设置宽和高都为 0
        if (getChildCount() == 0) {
            setMeasuredDimension(0, 0);
            return;
        }
        // 宽和高都是 AT_MOST，则宽度设置为所有子元素宽度的和，高度设置为第一个子元素的高度
        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
            View childOne = getChildAt(0);
            int childWidth = childOne.getMeasuredWidth();
            int childHeight = childOne.getMeasuredHeight();
            setMeasuredDimension(childWidth * getChildCount(), childHeight);
        } else if (widthMode == MeasureSpec.AT_MOST) {
            int childWidth = getChildAt(0).getMeasuredWidth();
            setMeasuredDimension(childWidth * getChildCount(), heightSize);
        } else if (heightMode == MeasureSpec.AT_MOST) {
            int childHeight = getChildAt(0).getMeasuredHeight();
            setMeasuredDimension(widthSize, childHeight);
        }
    }
```

这里如果没有子元素，则采用简化的写法，将宽和高直接设置为 0。正常的话，我们应该根据 LayoutParams 中的宽和高来做相应的处理。接着根据 widthMode 和 heightMode 来分别设置 HorizontalView 的宽和高。另外，我们在测量时没有考虑 HorizontalView 的 padding 和子元素的 margin。

#### 4.3、实现 onLayout

接下来实现 onLayout 来布局子元素。因为对每一种布局方式，子 View 的布局都是不同的，所以这是 ViewGroup 唯一一个抽象方法，需要我们自己去实现，代码如下所示：

```java
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childCount = getChildCount();
        int left = 0;
        View child;
        for (int i = 0; i < childCount; i++) {
            child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                int width = child.getMeasuredWidth();
                // childWidth 后续滑动需要用到
                childWidth = width;
                child.layout(left, 0, left + width, child.getMeasuredHeight());
                left += width;
            }
        }
    }
```

遍历所有的子元素。如果子元素不是 GONE，则调用子元素的 layout 方法将其放置到合适的位置上。这相当于默认第一个子元素占满了屏幕，后面的子元素就是在第一个屏幕后面紧挨着和屏幕一样大小的后续元素。所以 left 是一直累加的，top 保持为 0，bottom 保持为第一个元素的高度，right 就是 left+元素的宽度。同样，这里没有处理 HorizontalView 的 padding 以及子元素的 margin。

#### 4.4、处理滑动冲突

这个自定义 ViewGroup 为水平滑动，如果里面是 ListView，ListView 则为垂直滑动，这样会导致滑动的冲突。解决的方法就是，如果我们检测到的滑动方向是水平的话，就让父 View 进行拦截，确保父 View 用来进行 View 的滑动切换。

```java
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercept = false;

        // note 1、
        float x = ev.getX();
        float y = ev.getY();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = x - lastInterceptX;
                float deltaY = y - lastInterceptY;
                // note 2、
                if (Math.abs(deltaX) - Math.abs(deltaY) > 0) {
                    intercept = true;
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
        }

        lastX = x;
        lastY = y;
        lastInterceptX = x;
        lastInterceptY = y;

        return intercept;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return super.onTouchEvent(event);
    }
```

在上面代码注释 1 处，在刚进入 onInterceptTouchEvent 方法时就得到了点击事件的坐标，在 MotionEvent.ACTION_MOVE 中计算每次手指移动的距离，并在注释 2 处判断用户是水平滑动还是垂直滑动，如果是水平滑动则设置 intercept = true 来进行拦截，这样事件则由HorizontalView 的 onTouchEvent 方法来处理。

#### 4.5 弹性滑动到其他页面

接着处理 onTouchEvent 事件，在 onTouchEvent 方法里需要进行滑动切换页面，这里就需要用到 Scroller。

```java
public class HorizontalView extends ViewGroup {
    private float lastInterceptX;
    private float lastInterceptY;
    private float lastX;
    private float lastY;

    private int currentIndex = 0;
    private int childWidth = 0;
    private Scroller mScroller;
    
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = x - lastX;
                // - 往左 或 上 滚动
                // + 往右 或 下 滚动
                scrollBy((int) -deltaX, 0);
                break;
            case MotionEvent.ACTION_UP:
                int distance = getScrollX() - currentIndex * childWidth;
                // note 1、
                if (Math.abs(distance) > childWidth / 2) {
                    if (distance > 0) {
                        currentIndex++;
                    } else {
                        currentIndex--;
                    }
                }
                // note 2、
                smoothScrollTo(currentIndex * childWidth, 0);
                break;
        }
        lastX = x;
        lastY = y;
        return super.onTouchEvent(event);
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }

    private void smoothScrollTo(int destX, int destY) {
        mScroller.startScroll(getScrollX(), getScrollY(),
                destX - getScrollX(), destY - getScrollY(), 1000);
        invalidate();
    }
}    
```

同样，在刚进入 onTouchEvent 方法时就得到点击事件的坐标，在 MotionEvent.ACTION_MOVE 中用 scrollBy 方法来处理 HorizontalView 控件随手指滑动的效果。在上面代码注释 1 处判断滑动的距离是否大于宽度的 1/2，如果大于则切换到其他页面，然后调用 Scroller 来进行弹性滑动。

#### 4.6、快速滑动到其他页面

通常情况下，只在滑动超过一半时才切换到上/下一个页面是不够的。如果滑动速度很快的话，我们也可以判定为用户想要滑动到其他页面，这样的体验也是好的。需要在 onTouchEvent方法的 ACTION_UP 中对快速滑动进行处理。在这里又需要用到 VelocityTracker，它是用来测试滑动速度的。使用方法也很简单，首先在构造方法中进行初始化，代码如下所示：

```java
    private VelocityTracker mTracker;
    
	public HorizontalView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mScroller = new Scroller(context);
        mTracker = VelocityTracker.obtain();
    }
```

接着开始改写 onTouchEvent 部分，如下所示：

```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // note 7、
        mTracker.addMovement(event);
        float x = event.getX();
        float y = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = x - lastX;
                // - 往左 或 上 滚动
                // + 往右 或 下 滚动
                scrollBy((int) -deltaX, 0);
                break;
            case MotionEvent.ACTION_UP:
                int distance = getScrollX() - currentIndex * childWidth;
                // note 1、
                if (Math.abs(distance) > childWidth / 2) {
                    if (distance > 0) {
                        currentIndex++;
                    } else {
                        currentIndex--;
                    }
                } else {
                    // note 3、
                    mTracker.computeCurrentVelocity(1000);
                    float xVelocity = mTracker.getXVelocity();
                    // note 4、
                    if (Math.abs(xVelocity) > 50) {
                        // 切换到是一个页面
                        if (xVelocity > 0) {
                            currentIndex--;
                        } else {
                            //切换到下一个页面
                            currentIndex++;
                        }
                    }
                }
                // note 5、
                currentIndex = currentIndex < 0 ? 0 : Math.min(currentIndex, getChildCount() - 1);
                // note 2、
                smoothScrollTo(currentIndex * childWidth, 0);
                // note 6、
                mTracker.clear();
                break;
        }
        lastX = x;
        lastY = y;
        return super.onTouchEvent(event);
    }
```

首先要注意 note 7 不能遗漏了（注意：《Android进阶之光》这一行没贴出来）；在上面代码注释 3 处获取水平方向的速度；接着在注释 4 处，如果速度的绝对值大于 50的话，就被认为是“快速滑动”，执行切换页面。在注释 5 处对左边界和右边界做判断处理。在注释 6 处重置速度计算器。

#### 4.7、再次触摸屏幕阻止页面继续滑动

如果有如下的场景：当我们快速向左滑动切换到下一个页面时，在手指释放以后，页面会弹性滑动到下一个页面，这可能需要 1 秒才完成滑动，在这个时间内，我们再次触摸屏幕，希望能拦截这次滑动，然后再次去操作页面。 要实现在弹性滑动过程中再次触摸拦截，肯定要在onInterceptTouchEvent 的 ACTION_DOWN 中去判断。如果在 ACTION_DOWN 的时候，Scroller还没有执行完毕，说明上一次的滑动还正在进行中，则直接中断 Scroller，代码如下所示：

```java
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercept = false;

        // note 1、
        float x = ev.getX();
        float y = ev.getY();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercept = false;
                // note 3、
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = x - lastInterceptX;
                float deltaY = y - lastInterceptY;
                // note 2、
                if (Math.abs(deltaX) - Math.abs(deltaY) > 0) {
                    intercept = true;
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
        }

        // note 4、
        lastX = x;
        lastY = y;
        lastInterceptX = x;
        lastInterceptY = y;

        return intercept;
    }
```

主要逻辑在上面代码注释3处，如果Scroller没有执行完成，则调用Scroller的abortAnimation方法来打断 Scroller。因为 onInterceptTouchEvent 方法的 ACTION_DOWN 返回 false，所以在onTouchEvent 方法中无法获取 DOWN 事件，故而需要在注释 4 处设置 lastX 和 lastY 这两个参数。

#### 4.8、应用 HorizontalView

首先，我们在主布局中引用 HorizontalView，它作为父容器，里面有两个 ListView。布局文件如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <com.lizw.viewsystemdemo.HorizontalView
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ListView
            android:id="@+id/lv_one"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <ListView
            android:id="@+id/lv_two"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </com.lizw.viewsystemdemo.HorizontalView>

</LinearLayout>
```

接着，在代码中为 ListView 填加数据：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ListView listView = findViewById(R.id.lv_one);
        Integer[] data = {1, 2, 3, 4,1, 2, 3, 4,1, 2, 3, 4};
        listView.setAdapter(new ArrayAdapter<>(this, android.R.layout.simple_expandable_list_item_1, data));

        ListView listView2 = findViewById(R.id.lv_two);
        Integer[] data2 = {11, 21, 13, 41,11, 21, 13, 41,11, 21, 13, 14};
        listView2.setAdapter(new ArrayAdapter<>(this, android.R.layout.simple_expandable_list_item_1, data2));
    }
}
```

运行程序，效果如图 3-15 所示。当我们向右滑动的时候，界面会滑动到第二个 ListView，如图所示：

![image-20210422172523798](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210422172523798.png)![image-20210422172534503](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210422172534503.png)

最后贴上 HorizontalView 的整个源码：

```java
public class HorizontalView extends ViewGroup {
    private float lastInterceptX;
    private float lastInterceptY;
    private float lastX;
    private float lastY;

    private int currentIndex = 0;
    private int childWidth = 0;
    private Scroller mScroller;
    private VelocityTracker mTracker;

    public HorizontalView(Context context) {
        this(context, null);
    }

    public HorizontalView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public HorizontalView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mScroller = new Scroller(context);
        mTracker = VelocityTracker.obtain();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        measureChildren(widthMeasureSpec, heightMeasureSpec);
        // 如果没有子元素，就设置宽和高都为 0
        if (getChildCount() == 0) {
            setMeasuredDimension(0, 0);
            return;
        }
        // 宽和高都是 AT_MOST，则宽度设置为所有子元素宽度的和，高度设置为第一个子元素的高度
        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
            View childOne = getChildAt(0);
            int childWidth = childOne.getMeasuredWidth();
            int childHeight = childOne.getMeasuredHeight();
            setMeasuredDimension(childWidth * getChildCount(), childHeight);
        } else if (widthMode == MeasureSpec.AT_MOST) {
            int childWidth = getChildAt(0).getMeasuredWidth();
            setMeasuredDimension(childWidth * getChildCount(), heightSize);
        } else if (heightMode == MeasureSpec.AT_MOST) {
            int childHeight = getChildAt(0).getMeasuredHeight();
            setMeasuredDimension(widthSize, childHeight);
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childCount = getChildCount();
        int left = 0;
        View child;
        for (int i = 0; i < childCount; i++) {
            child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                int width = child.getMeasuredWidth();
                // childWidth 后续滑动需要用到
                childWidth = width;
                child.layout(left, 0, left + width, child.getMeasuredHeight());
                left += width;
            }
        }
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercept = false;

        // note 1、
        float x = ev.getX();
        float y = ev.getY();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercept = false;
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = x - lastInterceptX;
                float deltaY = y - lastInterceptY;
                // note 2、
                if (Math.abs(deltaX) - Math.abs(deltaY) > 0) {
                    intercept = true;
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
        }

        lastX = x;
        lastY = y;
        lastInterceptX = x;
        lastInterceptY = y;

        return intercept;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // note 7、
        mTracker.addMovement(event);
        float x = event.getX();
        float y = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = x - lastX;
                // - 往左 或 上 滚动
                // + 往右 或 下 滚动
                scrollBy((int) -deltaX, 0);
                break;
            case MotionEvent.ACTION_UP:
                int distance = getScrollX() - currentIndex * childWidth;
                // note 1、
                if (Math.abs(distance) > childWidth / 2) {
                    if (distance > 0) {
                        currentIndex++;
                    } else {
                        currentIndex--;
                    }
                } else {
                    // note 3、
                    mTracker.computeCurrentVelocity(1000);
                    float xVelocity = mTracker.getXVelocity();
                    Log.e("tag", "xVelocity:" + xVelocity);
                    // note 4、
                    if (Math.abs(xVelocity) > 50) {
                        // 切换到上一个页面
                        if (xVelocity > 0) {
                            currentIndex--;
                        } else {
                            //切换到下一个页面
                            currentIndex++;
                        }
                    }
                }
                // note 5、
                currentIndex = currentIndex < 0 ? 0 : Math.min(currentIndex, getChildCount() - 1);
                // note 2、
                smoothScrollTo(currentIndex * childWidth, 0);
                // note 6、
                mTracker.clear();
                break;
        }
        lastX = x;
        lastY = y;
        return super.onTouchEvent(event);
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }

    private void smoothScrollTo(int destX, int destY) {
        mScroller.startScroll(getScrollX(), getScrollY(),
                destX - getScrollX(), destY - getScrollY(), 1000);
        invalidate();
    }
}
```

#### 4.9、小结

要掌握自定义 View，需要对最基础的 View 与 ViewGroup、View 的滑动以及 View 的事件分发机制和 View 的工作流程等都熟练掌握。自定义 View 有着十分多变的处理方式，但是不管它如何多变，都遵循着一定的规则，本文对这些规则做了总结。这些基础的知识点，需要牢牢掌握，以后还会接触到其他的一些知识点，需要逐步积累，多多实践，才能更好的掌握自定义View。







参考：

1、《Android 进阶之光 第一版》

