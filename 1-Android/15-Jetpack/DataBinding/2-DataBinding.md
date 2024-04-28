在传统的开发模式中，我们实现交互页面时，需要在Activity或者fragment等UI组件所对应的XML布局文件中，除了按设计约束摆放各控件之外，还需要对这些与交互相关的控件设置id，然后在代码中进行findViewById操作，将这些控件对象进行实例化，再进行逻辑控制，如setText、setImage或者setOnClickListener等等。这样的话需要在UI组件中实现大量的逻辑处理，使得Activity或者Fragment等显得臃肿不堪，维护起来很困难，为了减轻页面的工作量，Google提出了DataBiiding。

DataBinding能够分担Activity等组件中的属于页面交互的工作。DataBinding主要有以下特点：代码更加简介，可读性高，能够在XML文件中实现UI控制，不再需要findViewById操作，能够与Model实体类直接绑定，易于维护和扩展。

要想在项目中接入DataBinding其实非常简单，只需要在module的build.gradle文件中加入以下配置即可：

```
android {
    dataBinding {
        enabled = true
    }
}
```

## DataBinding

### DataBinding简单使用

#### 1、修改布局文件

使用DataBinding的第一步，就是先改造XML文件，其实改造布局文件也特别简单，只需要在原来文件内容的基础上，最外层改为<layout>标签，其他内容不变：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="50dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

除了手动加layout标签外，也可以借助Android Studio的提示，选择Convert to data binding layout选项（Alt+Enter），自动添加layout标签。

![image-20210328222100277](C:\Users\lizw\AppData\Roaming\Typora\typora-user-images\image-20210328222100277.png)

在布局最外层加layout标签后，重新build项目，DataBinding库就会生成对应的Binding类，该类用来实现XML布局文件与Model类的绑定，如下：

![image-20210328141940085](C:\Users\lizw\AppData\Roaming\Typora\typora-user-images\image-20210328141940085.png)

```java
public class ActivityMainBindingImpl extends ActivityMainBinding  {

    @Nullable
    private static final androidx.databinding.ViewDataBinding.IncludedLayouts sIncludes;
    @Nullable
    private static final android.util.SparseIntArray sViewsWithIds;
    static {
        sIncludes = null;
        sViewsWithIds = null;
    }
    // views
    @NonNull
    private final androidx.constraintlayout.widget.ConstraintLayout mboundView0;
    // variables
    // values
    // listeners
    // Inverse Binding Event Handlers

    public ActivityMainBindingImpl(@Nullable androidx.databinding.DataBindingComponent bindingComponent, @NonNull View root) {
        this(bindingComponent, root, mapBindings(bindingComponent, root, 1, sIncludes, sViewsWithIds));
    }
    private ActivityMainBindingImpl(androidx.databinding.DataBindingComponent bindingComponent, View root, Object[] bindings) {
        super(bindingComponent, root, 0
            );
        this.mboundView0 = (androidx.constraintlayout.widget.ConstraintLayout) bindings[0];
        this.mboundView0.setTag(null);
        setRootTag(root);
        // listeners
        invalidateAll();
    }

    @Override
    public void invalidateAll() {
        synchronized(this) {
                mDirtyFlags = 0x1L;
        }
        requestRebind();
    }

    @Override
    public boolean hasPendingBindings() {
        synchronized(this) {
            if (mDirtyFlags != 0) {
                return true;
            }
        }
        return false;
    }

    @Override
    public boolean setVariable(int variableId, @Nullable Object variable)  {
        boolean variableSet = true;
            return variableSet;
    }

    @Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
        }
        return false;
    }

    @Override
    protected void executeBindings() {
        long dirtyFlags = 0;
        synchronized(this) {
            dirtyFlags = mDirtyFlags;
            mDirtyFlags = 0;
        }
        // batch finished
    }
    // Listener Stub Implementations
    // callback impls
    // dirty flag
    private  long mDirtyFlags = 0xffffffffffffffffL;
    /* flag mapping
        flag 0 (0x1L): null
    flag mapping end*/
    //end
}
```

生成Binding类的名字很特殊，它与XML布局文件的名字有对应关系，具体的联系就是，以XML布局文件为准，去掉下划线，所有单词以大驼峰的形式按顺序拼接，最后再加上Binding，如我的XML布局文件名是activity_main，生成的Binding类名就是ActivityMainBinding。

#### 2、绑定布局

没有用DataBinding时，为了将XML布局文件与Activity进行绑定，需要调用Activity的setContentView()方法，或者是在Fragment中调用LayoutInflate的inflate()方法，将XML在R文件中的映射传入进行绑定。如果使用了DataBinding之后，就需要使用DataBindingUtil类，如：

java：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
    }
}
```

这里binding的类型为此前生成的对应类：ActivityMainBinding。

Kotlin：

```kotlin
var binding = DataBindingUtil.setContentView<ActivityDataBindingBinding>(
            this@DataBindingActivity,
            R.layout.activity_data_binding
        )
```

使用DataBindingUtil类的setContentView()方法对Activity进行绑定，其返回值就是工具生成的Binding类。

在Fragment会稍有不同：

```kotlin
class DataBindingFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        var fragmentDataBindingBinding = FragmentDataBindingBinding.inflate(inflater)
        return fragmentDataBindingBinding.root
    }

}
```

在Fragment中不是使用DataBindingUtil类来进行绑定，而是通过生成的Binding类的inflate()方法inflate并返回一个Binding类实例，最终调用getRoot()方法返回对应的View。

#### 3、添加data标签

之前的步骤，已经使用DataBinding将XML文件与UI组件进行了绑定，然后需要在XML文件中接受Model数据，这里就用到了data标签与variable标签。

在XML文件的layout标签下，创建data标签，在data标签中再创建variable标签，variable标签主要用到的就是name属性和type属性，类似于Java语言声明变量时，需要为该变量指定类型和名称，这里的type就是该属性的类型，这里需要设置Model类的全类名，name就是属性的名称，供在XML文件的其他位置引用，表示一个Model类实例：

其中Person定义如下：

```
class Person {

    var isMen = true
    var age: Int = 0
    var name: String
    var address: String

    constructor(age: Int, name: String, address: String) {
        this.age = age
        this.name = name
        this.address = address
    }

    public fun getFullAddress(): String {
        return "China $address"
    }
}

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="person"
            type="com.jia.demo.jetpack.databinding.Person" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="50dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

data、variable标签添加后，clear project，然后重新build生成相应的Binding类。

#### 4、传递variable属性

在定义好XML布局文件之后，我们就需要在Activity或者Fragment中将Model实体类对象传递给XML文件了，这里就用到了自动生成的Binding类了，Binding类为每一个variable标签对应的type都创建了set方法用以传递数据：

```java
binding.setPerson(new Person(true, "23", "Li", "Nanjing"));
```

variable标签的type，不仅可以是我们定义的数据类型，也可以是基本数据类型，如String，Integer等等。

#### 5、使用variable

我们已经在XML文件中声明好了一个variable属性，接下来就可以使用了。

这里需要用到一个新的概念叫做布局表达式： @{ }

可以在布局表达式@{ }中，传入variable对象的各属性，也可以是方法，最终将完整的布局表达式设置给控件的对应属性即可：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <import type="com.lizw.jectpackdemo.PersonUtil" />

        <variable
            name="person"
            type="com.lizw.jectpackdemo.Person" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tv1"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{String.valueOf(person.age)}" />

        <TextView
            android:id="@+id/tv2"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{person.name}" />

        <TextView
            android:id="@+id/tv3"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{person.address}" />

        <TextView
            android:id="@+id/tv4"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{PersonUtil.getInfo(person)}" />

        <CheckBox
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:checked="@{person.man}" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onClick"
            android:text="change person" />
    </LinearLayout>
</layout>
```

上述示例中，我们将Person对象的各个属性分别设置给了三个TextView，也就是将布局表达式设置给TextView的text属性，可以看到，布局表达式中，不仅可以传入variable对象的属性，也可以调用variable对象的方法。

由于在上一步中，已经把初始化好的Person对象设置给了Binding类，所以这里直接运行代码就可以看到效果了。

Person类中的age属性是一个int类型，如果在XML中直接使用@{person.age}设置给text属性，会导致程序奔溃（**把int转为String就好了**），这里要注意，text属性传入的布局表达式中的属性或者是方法，必须是字符串类型。CheckBox的checked属性要求传入的是一个boolean类型值，而Person的isMen属性就是boolean属性，所以这里可以将checked属性设置为@{person.men}。也就是说，布局表达式中的值，需要满足控件属性的类型要求，否则会抛异常。

之前我们是将整个Person对象设置给了Binding类，我们也可以单独的修改其中的某个值，如：

```java
public void onClick(View view){
    binding.setPerson(new Person(true, 18, "Zhang", "Nanjing000"));
}
```

这样，当我们动态改变Person中的属性值时，XML布局中的内容也会自动随之改变，这正是DataBinding的一大特点，不再需要手动调用控件设置属性的方法去更新控件，实现了由Model类动态控制UI展示。

#### 6、布局表达式中使用静态方法

在开发中经常需要编写一些工具类，对Model属性进行编辑和修改，当然，工具类中的方法需要是静态方法。DataBinding也支持在XML中使用静态方法。首先我们先定义一个静态方法：

```
public class PersonUtil {

    public static String getFullInfo(Person person) {
        return person.getName() + " " + person.getAddress();
    }
}
```

接下来需要在data标签中使用import属性，意为导入类：

```xml
    <data>

        <import type="com.lizw.jectpackdemo.PersonUtil" />

        <variable
            name="person"
            type="com.lizw.jectpackdemo.Person" />
    </data>
```

然后就可以在布局表达式中调用其静态方法了：

```xml
        <TextView
            android:id="@+id/tv4"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{PersonUtil.getInfo(person)}" />
```

#### 7、响应事件

使用DataBinding，不仅可以在布局文件中对控件某些属性进行赋值，使得Model类数据直接绑定在布局中，而且Model属性发生变化时，布局文件中的内容可以即时刷新。而且还可以使用DataBinding响应用户事件，如可以实现Button点击时的响应。

上文的介绍可以知道，布局表达式不仅可以传入对象的属性，也可以调用对象的方法。

首先创建一个工具类，在类中定义响应事件的方法，如：

```java
public class ButtonClickListener {
    public void onClick(View view) {
        Toast.makeText(view.getContext(), "click", Toast.LENGTH_SHORT).show();
    }
}
```

在布局文件中：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <import type="com.lizw.jectpackdemo.PersonUtil" />

        <variable
            name="person"
            type="com.lizw.jectpackdemo.Person" />

        <variable
            name="ButtonClickHandler"
            type="com.lizw.jectpackdemo.ButtonClickListener" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{ButtonClickHandler.onClick}"
            android:text="响应事件" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

首先在data标签中为ButtonClickListener类声明对象，在Button的onClick属性中传入布局表达式，这里内容是声明的点击处理类对象的onClick方法。

之后在Activity中传入ButtonClickListener对象即可：

```java
binding.setButtonClickHandler(new ButtonClickListener());
```

这样就点击Button的时候就可以响应点击事件了。

#### 8、include标签

为了能够让布局文件得到**复用**，在编写布局的时候，会经常用的include标签，使得相同结构与内容的布局文件可以在多处使用。但是如果一个布局文件中使用了DataBinding，同时也使用了include标签，那么在include标签引入的布局文件中如何使用一级页面的数据呢。

这时，需要在一级页面的include标签中，通过命名空间xmlns:app来引入布局变量person，将数据对象传递给二级页面，如：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>

        <variable
            name="person"
            type="com.jia.demo.jetpack.databinding.Person" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <include
            layout="@layout/layout_data_binding"
            app:persondata="@{person}" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>

```

布局表达式中直接传入页面变量person，这里的include标签属性值可以任意取名，但是要注意的是，在二级页面的variable标签中的name属性，必须与一级页面中的include标签属性名一致，如这里的“persondata”：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>

    <data>

        <variable
            name="persondata"
            type="com.lizw.jectpackdemo.Person" />
    </data>

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{persondata.address}"
            />
    </LinearLayout>

</layout>
```

这样，在include所引入的layout文件中，也可以直接使用由一级页面传过来的数据了。

### BindingAdapter

使用DataBinding库时，DataBinding会针对控件属性生成对应的XXXBindingAdapter类，如TextViewBindingAdapter类，其对TextView的每个可以使用DataBinding的属性都生成了对应的方法，而且每个方法都使用了@BindingAdapter注解，注解中的参数就是对应View的属性，如：

```java
@RestrictTo(RestrictTo.Scope.LIBRARY)
@SuppressWarnings({"WeakerAccess", "unused"})
@BindingMethods({
        @BindingMethod(type = TextView.class, attribute = "android:autoLink", method = "setAutoLinkMask"),
        @BindingMethod(type = TextView.class, attribute = "android:drawablePadding", method = "setCompoundDrawablePadding"),
        @BindingMethod(type = TextView.class, attribute = "android:editorExtras", method = "setInputExtras"),
        @BindingMethod(type = TextView.class, attribute = "android:inputType", method = "setRawInputType"),
        @BindingMethod(type = TextView.class, attribute = "android:scrollHorizontally", method = "setHorizontallyScrolling"),
        @BindingMethod(type = TextView.class, attribute = "android:textAllCaps", method = "setAllCaps"),
        @BindingMethod(type = TextView.class, attribute = "android:textColorHighlight", method = "setHighlightColor"),
        @BindingMethod(type = TextView.class, attribute = "android:textColorHint", method = "setHintTextColor"),
        @BindingMethod(type = TextView.class, attribute = "android:textColorLink", method = "setLinkTextColor"),
        @BindingMethod(type = TextView.class, attribute = "android:onEditorAction", method = "setOnEditorActionListener"),
})
public class TextViewBindingAdapter {

    private static final String TAG = "TextViewBindingAdapters";
    @SuppressWarnings("unused")
    public static final int INTEGER = 0x01;
    public static final int SIGNED = 0x03;
    public static final int DECIMAL = 0x05;

    @BindingAdapter("android:text")
    public static void setText(TextView view, CharSequence text) {
        final CharSequence oldText = view.getText();
        if (text == oldText || (text == null && oldText.length() == 0)) {
            return;
        }
        if (text instanceof Spanned) {
            if (text.equals(oldText)) {
                return; // No change in the spans, so don't set anything.
            }
        } else if (!haveContentsChanged(text, oldText)) {
            return; // No content changes, so don't set anything.
        }
        view.setText(text);
    }

    @InverseBindingAdapter(attribute = "android:text", event = "android:textAttrChanged")
    public static String getTextString(TextView view) {
        return view.getText().toString();
    }

    @BindingAdapter({"android:autoText"})
    public static void setAutoText(TextView view, boolean autoText) {
        KeyListener listener = view.getKeyListener();

        TextKeyListener.Capitalize capitalize = TextKeyListener.Capitalize.NONE;

        int inputType = listener != null ? listener.getInputType() : 0;
        if ((inputType & InputType.TYPE_TEXT_FLAG_CAP_CHARACTERS) != 0) {
            capitalize = TextKeyListener.Capitalize.CHARACTERS;
        } else if ((inputType & InputType.TYPE_TEXT_FLAG_CAP_WORDS) != 0) {
            capitalize = TextKeyListener.Capitalize.WORDS;
        } else if ((inputType & InputType.TYPE_TEXT_FLAG_CAP_SENTENCES) != 0) {
            capitalize = TextKeyListener.Capitalize.SENTENCES;
        }
        view.setKeyListener(TextKeyListener.getInstance(autoText, capitalize));
    }
}
```

BindingAdapter类中，所有的方法都是static方法，并且每个方法都使用了@BindingAdapter注解，注解中声明所操作的View属性，当使用了DataBinding的布局文件被渲染时，属性所对应的static方法就会自动调用。

除了使用库自动生成的BindingAdapter类之外，当然也可以自定义BindingAdapter类，供开发者来实现系统没有提供的属性绑定，可以借助自定义BindingAdapter实现一些比较复杂的属性操作。

#### 自定义BindingAdapter类

我们假设来实现这样的需求，为TextView添加“fullText”属性，为该属性传入字符串内容，但是该字符串如果超过4个字符长度，超过部分直接截取，如果不够4个字符则末尾以“++”补齐。

```java
public class FullTextBindingAdapter {
    @BindingAdapter("fullText")
    public static void setFullText(TextView textView, String content) {
        if (TextUtils.isEmpty(content)) {
            content = "****";
        } else if (content.length() > 4) {
            content.substring(0, 4);
        }
        textView.setText(content);
    }
}
```

定义FullTextViewBindingAdapter类，在其中创建静态方法“setFullText()”方法，并且使用@BindingAdapter注解修饰，在注解中设置我们为TextView新定义的属性。

该方法第一个参数必须是所操作的View类型，第二个参数是字符串类型，在方法中进行逻辑处理。接下来正在布局文件中使用：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="persondata"
            type="com.lizw.jectpackdemo.Person" />
    </data>

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:fullText="@{persondata.name}" />
    </LinearLayout>

</layout>
```

可以看到，在布局文件中，还是如之前的普通属性一样，直接使用fullText属性即可。这里只是简单介绍了一个例子，我们还可以为ImageView设置网络图片，只需要传入图片地址url即可，在这里就不再详细介绍。

BindingAdapter中的静态方法是允许重载的，例如，我们需要为ImageView传入一个字符串类型的参数，在静态方法中加载网络图片并设置到ImageView上，当然，ImageView也可以设置int类型的数据，即加载本地资源图片，所以可以同时定义两个不同参数类型的重载方法，在布局中使用时传入两种类型的数据都是可以的。

BindingAdapter中的静态方法，一般都是接受两个参数，第一个必须是所操作的View类型变量，第二个控制属性值，当然，这些方法也可以传入多个属性值，如：

```java
    @BindingAdapter(value = {"shortText", "longText"}, requireAll = false)
    public static void setFullText(TextView textView, String shortText, String longText) {
        textView.setText(shortText + longText);
    }
```

在@BindingAdapter注解中，第一个参数传入字符串数组，其中的内容就是各个新增的属性名，后面requireAll参数表示是否同时需要，这里设置为false。此方法的目的是将传入的两个字符串值拼接起来再设置给TextView。

使用是要注意赋值要使用：@{}的方式。不能直接写app:shortText="sss"这种，否则报错。

![image-20210328171555803](C:\Users\lizw\AppData\Roaming\Typora\typora-user-images\image-20210328171555803.png)

这种静态方法多参数的情况下，如果注解中并没有指定value数组，而方法参数中除了第一个View参数，后面还有两个相同类型的参数，这时第一个参数为旧参数值，最后一个为新的参数值：

```java
    @BindingAdapter("fullText1")
    public static void setFullText(TextView textView, String oldText, String newText) {
        if (TextUtils.isEmpty(oldText)) {
            textView.setText(newText);
        } else {
            textView.setText(oldText + newText);
        }
    }
```

```xml
使用：
<TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:fullText1="@{person.name}"/>
```

> 使用**@BindingAdapter**，我们**可以快速的给View添加新的属性**，并且定义属性绑定时的操作。
>
> 最爽的是它可以拓展现有view的属性，而不是传统的通过自定义View然后declare-styleable->attr添加属性。

### 双向绑定

<u>通过DataBinding可将Model数据直接绑定在布局文件中，并且当数据发生变化时，布局中的内容可以动态随之变化，我们称之为单向绑定。</u>试想这样的场景，如果布局中有一个EditText，当用户在输入框中输入内容时，我们希望对应的Model类能够实时更新，这就需要双向绑定，DataBinding同样支持这样的能力。

实现双向绑定，就需要用到ObservableField类，它能够将普通的数据对象包装成一个可观察的数据对象，数据可以是基本类型变量，可以是集合，也可以是自定义类型。我们依旧以Person类为例，有一个默认地址，直接展示到EditText中，当用户编辑输入框时，又能将输入框中的内容实时更新到对应的Person属性中。

首先定义一个ViewModel类，在其中**声明一个ObservableField对象**，其范型指定为Person，并针对Person的某个属性（如address），**实现get和set方法**，这样在DataBinding会在布局表达式中调用get和set方法，如下：

```java
public class PersonViewModel {
    ObservableField<Person> personObservableField = new ObservableField<>();

    public PersonViewModel(Person person) {
        personObservableField.set(person);
    }

    public String getAddress() {
        return address();
    }

    public void setAddress(String address) {
        Objects.requireNonNull(personObservableField.get()).setAddress(address);
    }

    public void onClick(View view) {
        Toast.makeText(view.getContext(), address(), Toast.LENGTH_SHORT).show();
    }

    private String address() {
        return Objects.requireNonNull(personObservableField.get()).getAddress();
    }
}
```

接下来在布局文件中使用DataBinding：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="personViewModel"
            type="com.lizw.jectpackdemo.PersonViewModel" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@={personViewModel.address}" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{personViewModel.onClick}"
            android:text="address"
            android:textAllCaps="false" />
    </LinearLayout>
</layout>
```

需要特别注意的是，不同于单向绑定，之前的布局表达式是@{}，实现双向绑定使用的布局表达式是@={}，多了一个等号，便可实现双向绑定。为了验证效果，摆放了一个Button，点击时将Person中的addrss信息展示出来。这样就实现了双向绑定，即根据Model数据主动更新布局，根据布局内容变化再主动通知更新Model数据。

> ObservableField与LiveData非常相似，都是将普通的数据对象封装成了可观察对象，理论上二者是可以互相替代的，但LiveData具有生命周期感知能力，并且需要调用observe()方法进行监听，而双向绑定中更推荐使用ObservableField，不需要使用observe()方法，维护起来更加简单。
>

### RecyclerView中使用DataBinding

列表布局在Android应用中非常常见，RecyclerView是使用频率很高的一个控件，DataBinding也支持在RecyclerView中实现数据绑定。

使用RcyclerView，就需要用到Adapter，在Adapter中实例化Item布局，然后将List中的数据绑定到布局中，DataBinding可以帮助开发者实例化布局并绑定数据，下面以一个简单例子进行介绍：

首先编写item布局，在item布局中使用DataBinding将person数据绑定：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="person"
            type="com.lizw.jectpackdemo.Person" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        android:background="#1abc94"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="30dp"
            android:gravity="center"
            android:text="@{person.name}" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="30dp"
            android:gravity="center"
            android:text="@{person.address}" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="30dp"
            android:gravity="center"
            android:text="@{person.getSex}" />
    </LinearLayout>
</layout>
```

接下来编写Adapter类：

```java
public class PersonAdapter extends RecyclerView.Adapter<PersonAdapter.DataBindingViewHolder> {
    private List<Person> mPersonList;

    public PersonAdapter(List<Person> personList) {
        mPersonList = personList;
    }

    @NonNull
    @Override
    public DataBindingViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        ItemRecyclerDataBindingBinding binding = DataBindingUtil.inflate(LayoutInflater.from(parent.getContext()), R.layout.item_recycler_data_binding, parent, false);
        return new DataBindingViewHolder(binding);
    }

    @Override
    public void onBindViewHolder(@NonNull DataBindingViewHolder holder, int position) {
        holder.binding.setPerson(mPersonList.get(position));
    }

    @Override
    public int getItemCount() {
        return mPersonList.size();
    }

    public static class DataBindingViewHolder extends RecyclerView.ViewHolder {
        ItemRecyclerDataBindingBinding binding;

        public DataBindingViewHolder(@NonNull ItemRecyclerDataBindingBinding itemBinding) {
            super(itemBinding.getRoot());
            this.binding = itemBinding;
        }
    }
}
```

首先是ViewHolder类，不同于以往，之前都是需要在ViewHolder中进行findViewById对子控件进行实例化，由于我们使用了DataBinding，所以不再需要这些操作，这里只需要传入生成的Binding类就可以，在super中调用getRoot()方法返回根View。

Adapter中主要不同的是，首先通过DataBindingUtil的inflate()方法来渲染布局和创建Binding类，并将Binding对象传给ViewHolder。在onBindViewHolder中，将对应position的数据赋值给从ViewHolder的Binding类即可。

在RecyclerView中使用DataBinding就是如此简单，当List中的item数据发生变化时，列表中的内容也会随之更新。

从上面的代码中也可以发现，对RecyclerView设置LayoutManager和Adapter属于对View的一些复杂操作，我们可以通过自定义BindingAdapter的方式，定义一个新的属性，将数据List直接通过DataBinding在布局文件中绑定，将这些操作都封装到BindindAdapter中，在Activity中不再需要设置LayoutManager和Adapter操作，首先定义BindingAdapter：

```java
public class PersonRecyclerViewBindingAdapter {

    /**
     * 注意此方法要设置为static
     */
    @BindingAdapter("persons")
    public static void setPersons(RecyclerView recyclerView, List<Person> persons) {
        recyclerView.setLayoutManager(new LinearLayoutManager(recyclerView.getContext()));
        recyclerView.setAdapter(new PersonAdapter(persons));
    }
}
```

声明新的属性叫做“persons”，使用@BindingAdapter修饰静态方法，对应的需要设置List对象，在方法中对RecyclerView设置LayoutManager和Adapter。

在布局中使用DataBinding：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <import type="com.lizw.jectpackdemo.Person" />

        <import type="java.util.List" />

        <variable
            name="persons"
            type="List&lt;Person&gt;" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".recyclerview.RecyclerDataBindingActivity">

        <androidx.recyclerview.widget.RecyclerView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:persons="@{persons}" />
    </LinearLayout>
</layout>
```

这里稍不同于以往，这里所需要的数据对象是一个集合List，首先我们import导一下两个类，即Person和List，在variiable属性中设置对象名为persons，type这里要注意，由于编译器的原因，范型指定时的一对尖括号需要转义，接下来在RecyclerView的控件上将persons设置给对应属性即可。

在Activity中：

```java
public class RecyclerDataBindingActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityRecyclerDataBindingBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_recycler_data_binding);
        binding.setPersons(getPersons());
    }

    private List<Person> getPersons() {
        List<Person> people = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            people.add(new Person(true, i, "name:" + i, "address:" + i));
        }
        return people;
    }
}
```

其核心就是通过DataBindingUtil类渲染布局，将准备好的数据List设置给Binding类，即完成了RecyclerView的列表数据展示。是不是很简单。
