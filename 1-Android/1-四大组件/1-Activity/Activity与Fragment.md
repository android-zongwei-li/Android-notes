# 前言

- `Activity` 与 `Fragment` 的使用在`Android`开发中非常多

- 今天，我将主要讲解 

  ```
  Activity
  ```

   与 

  ```
  Fragment
  ```

   如何进行通信，实际上是要解决两个问题：

  1. `Activity` 如何传递数据到 `Fragment`？
  2. `Fragment`如何传递数据到`Activity` ？

下面，我将解答这两个问题。

> 阅读本文前，建议阅读[Android：Fragment最全面介绍 & 使用方法解析](https://www.jianshu.com/p/2bf21cefb763)

------

# 问题1： Activity 如何传递数据到 Fragment？

答：采用 `Bundle`方式。具体Demo步骤如下：

- 步骤1：`Activity`的布局文件

*activcity_2_fragment.xml*



```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/text"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20dp"
        android:text="我是Activity" />

    <FrameLayout
        android:layout_below="@+id/button"
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="500dp"/>
</LinearLayout>
```

- 步骤2：设置 `Fragment`的布局文件

*fragment.xml*



```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/colorAccent"
    >

    <TextView
        android:id="@+id/fragment"
        android:text="我是fragment"
        android:layout_gravity="center"
        android:textSize="30dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />

    <TextView
        android:id="@+id/text"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20dp"
        android:text="等待Activity发送消息" />

    <Button
        android:id="@+id/button"
        android:layout_gravity="center"
        android:text="点击接收Activity消息"
        android:layout_centerInParent="true"
        android:textSize="20dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

- 步骤3：设置`Activity`的类文件

*Activity2Fragment*



```java
public class Activity2Fragment extends AppCompatActivity {

    TextView text;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activcity_2_fragment);

        text = (TextView) findViewById(R.id.text);

        // 步骤1：获取FragmentManager
        FragmentManager fragmentManager = getFragmentManager();

        // 步骤2：获取FragmentTransaction
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

        // 步骤3：创建需要添加的Fragment 
        final mFragment fragment = new mFragment();

        // 步骤4:创建Bundle对象
        // 作用:存储数据，并传递到Fragment中
        Bundle bundle = new Bundle();

        // 步骤5:往bundle中添加数据
        bundle.putString("message", "I love Google");

        // 步骤6:把数据设置到Fragment中
        fragment.setArguments(bundle);

        // 步骤7：动态添加fragment
        // 即将创建的fragment添加到Activity布局文件中定义的占位符中（FrameLayout）
        fragmentTransaction.add(R.id.fragment_container, fragment);
        fragmentTransaction.commit();


    }
}
```

- 步骤4：设置`Fragment`的类文件

*mFragment.java*



```java
public class mFragment extends Fragment {
    Button button;
    TextView text;
    Bundle bundle;
    String message;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View contentView = inflater.inflate(R.layout.fragment, container, false);
        // 设置布局文件

        button = (Button) contentView.findViewById(R.id.button);
        text = (TextView) contentView.findViewById(R.id.text);

        // 步骤1:通过getArgments()获取从Activity传过来的全部值
        bundle = this.getArguments();

        // 步骤2:获取某一值
        message = bundle.getString("message");

        // 步骤3:设置按钮,将设置的值显示出来
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // 显示传递过来的值
                text.setText(message);

            }
        });

        return contentView;
    }
}
```

# 展示结果

![img](https:////upload-images.jianshu.io/upload_images/944365-4f2f61f4decd097b.gif?imageMogr2/auto-orient/strip|imageView2/2/w/388/format/webp)

示意图

至此，`Activity` 传递数据到 `Fragment` 讲解完毕。

------

# 问题2：Fragment 如何传递数据到 Activity

- 答：采用 **接口回调** 方式。
- 接口回调 回顾
   把实现了某一接口的类所创建的对象的引用 赋给 该接口声明的变量，通过该接口变量 调用 该实现类对象的实现的接口方法。



```cpp
// 接口声明的变量
Com com；

// 实现了Com接口的类（Com1）所创建的对象的引用 赋给 该接口声明的变量
Com com = new Com1；

// 通过该接口变量（com） 调用 该实现类对象（Com1）的实现的接口方法（carson（））
com.carson（）；
```

# 具体Demo

- 步骤1：在`Activity`的布局文件定义1占位符（`FrameLayout`）

*activity_main.xml*



```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="scut.carson_ho.fragment_2_activity.MainActivity">

    <TextView
        android:id="@+id/text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20dp"
        android:text="等待Fragment发送消息" />

    <Button
        android:id="@+id/button"
        android:layout_below="@+id/text"
        android:text="点击接收Fragment消息"
        android:layout_centerInParent="true"
        android:textSize="10dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <FrameLayout
        android:layout_below="@+id/button"
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="500dp"/>

</RelativeLayout>
```

- 步骤2：设置`Fragment`的布局文件

*fragment.xml*



```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <TextView
        android:id="@+id/fragment"
        android:text="我是fragment"
        android:gravity="center"
        android:textSize="30dp"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorAccent"/>

</LinearLayout>
```

- 步骤3：设置回调接口
   该接口用于用于`Activity`与`Fragment`通信

*ICallBack.java*



```csharp
public interface ICallBack {
    void get_message_from_Fragment(String string);

}
```

- 步骤4：设置`Fragment`的类文件

*mFragment.java*



```java
public class mFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View contentView = inflater.inflate(R.layout.fragment, container, false);
        // 设置布局文件
        return contentView;
    }

    // 设置 接口回调 方法
    public void sendMessage(ICallBack callBack){

        callBack.get_message_from_Fragment("消息:我来自Fragment");

    }
}
```

- 步骤5：设置`Acticvity`的类文件

*Main_Activity.java*



```java
public class MainActivity extends AppCompatActivity {

    Button button;
    TextView text;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button)findViewById(R.id.button);
        text = (TextView)findViewById(R.id.text);

        // 步骤1：获取FragmentManager
        FragmentManager fragmentManager = getFragmentManager();

        // 步骤2：获取FragmentTransaction
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

        // 步骤3：创建需要添加的Fragment 
        final mFragment fragment = new mFragment();

        // 步骤4：动态添加fragment
        // 即将创建的fragment添加到Activity布局文件中定义的占位符中（FrameLayout）
        fragmentTransaction.add(R.id.fragment_container, fragment);
        fragmentTransaction.commit();


        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // 通过接口回调将消息从fragment发送到Activity
                fragment.sendMessage(new ICallBack() {
                    @Override
                    public void get_message_from_Fragment(String string) {
                            text.setText(string);
                    }
                });

            }
        });
    }


}
```

# 结果展示

![img](https:////upload-images.jianshu.io/upload_images/944365-d6fabd355159dd04.gif?imageMogr2/auto-orient/strip|imageView2/2/w/392/format/webp)

示意图.gif

至此，将数据从 `Fragment` 发送到 `Activity` 讲解完毕

------

# 总结

- 看完本文，你应该非常清楚该如何实现 `Activity` 与 `Fragment` 相互通信

![img](https:////upload-images.jianshu.io/upload_images/944365-6f8f901882962ca9.png?imageMogr2/auto-orient/strip|imageView2/2/w/400/format/webp)



# 参考

[Android：手把手教你 实现Activity 与 Fragment 相互通信（含Demo）](https://www.jianshu.com/p/825eb1f98c19)