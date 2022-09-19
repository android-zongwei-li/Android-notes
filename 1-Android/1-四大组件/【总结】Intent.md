> version：2022/09/
>
> review：



目录

[TOC]



# 一、Intent

Intent是各个系统组件之间进行数据传递的载体。



# 二、Intent 的作用

作用一：提供必要的信息，用于匹配到指定的组件。

作用二：可以携带数据，在启动新组件后，传递给新组件。



# 三、Intent 的几个重要属性

## 1、action

```java
	<activity android:name=".TargetActivity">
		<intent-filter>
			<action android:name="com.lizw.intent.action.TARGET"/>
			<category android:name="android.intent.category.DEFAULT"/>
		</intent-filter>
	</activity>
```

Activity 在添加了 action 后，其他应用就可以通过这个 action 来启动它，如下：

```java
	Intent intent = new Intent("com.lizw.intent.action.TARGET");
	startActivity(intent);
```

上面这个 action 是自定义的，除了自定义外，系统也提供了一系列 action 供我们使用：

```java
Intent.java
  public static final String ACTION_MAIN = "android.intent.action.MAIN";
	public static final String ACTION_VIEW = "android.intent.action.VIEW";
	public static final String ACTION_WEB_SEARCH = "android.intent.action.WEB_SEARCH";
	public static final String ACTION_CALL = "android.intent.action.CALL";
```

通过这些 action ，可以匹配到对应的 Activity（声明了该 action 的）。



## 2、data

```java
/**
 * 打开指定网页
 * @param view
 */
public void invokeWebBrowser() {
	Intent intent = new Intent(Intent.ACTION_VIEW);
	intent.setData(Uri.parse("http://www.baidu.com"));
	startActivity(intent);
}
```

data 是一个 Uri 类型的数据。其组成格式为scheme+host+port+path。

**android:scheme** 匹配url中的前缀，除了“http”、“https”、“tel”...之外，我们也可以定义自己的前缀

**android:host** 匹配url中的主机名部分，如“google.com”，如果定义为“*”则表示任意主机名

**android:port** 匹配url中的端口

**android:path** 匹配url中的路径



```java
	<activity android:name=".TargetActivity">
		<intent-filter>
			<action android:name="com.lizw.intent.action.TARGET"/>
			<category android:name="android.intent.category.DEFAULT"/>
			<data android:scheme="lizw" android:host="com.lizw.intent.data" android:port="7788" android:path="/target"/>
		</intent-filter>
	</activity>
```

这个时候如果只指定action就不能匹配到这个 Activity 了，我们还需要为其设置data值：

```java
public void gotoTargetActivity() {
	Intent intent = new Intent("com.lizw.intent.action.TARGET");
	intent.setData(Uri.parse("lizw://com.scott.intent.data:7788/target"));
	startActivity(intent);
}
```

url中的每个部分和TargetActivity配置信息中全部一致才能跳转成功，否则就被系统拒绝。

需要注意，像下面这两个url 也是不能匹配到的

lizw://com.scott.intent.data:7788/target**/hello**

lizw://com.scott.intent.data:7788/target**/hi**

这种情况可以将 path 改成 pathPrefix 属性，表示路径前缀。



## 3、extras

```java
/**
 * 进行关键字搜索
 * @param view
 */
public void invokeWebSearch(View view) {
	Intent intent = new Intent(Intent.ACTION_WEB_SEARCH);
	intent.putExtra(SearchManager.QUERY, "android");	//关键字
	startActivity(intent);
}
```
可以通过extras往intent中添加数据，这些数据在新的组件中可以取出来。

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val intent  = intent
    intent.extras.getInt()
    intent.getIntExtra()
}
```



## 4、category

常见的类型有：

```java
// 应用启动时，调用的 Activity
<category android:name="android.intent.category.LAUNCHER" />

// 默认的category
Intent.CATEGORY_DEFAULT（android.intent.category.DEFAULT）

// 表示该目标Activity是一个首选项界面；
Intent.CATEGORY_PREFERENCE（android.intent.category.PREFERENCE） 

// 在网页上点击图片或链接时，系统会考虑将此目标Activity列入可选列表，供用户选择以打开图片或链接。
Intent.CATEGORY_BROWSABLE（android.intent.category.BROWSABLE）
```

在为 Intent 设置category时，应使用addCategory(String category)方法向Intent中添加指定的类别信息，来匹配声明了此类别的目标Activity。



## 5、type

指定可以处理的 MIME 数据类型

比如，一个可以处理图片的Activity在其声明中包含这样的mimeType

```
<data android:mimeType="image/*" />
```


在使用Intent进行匹配时，我们可以使用setType(String type)或者setDataAndType(Uri data, String type)来设置mimeType。



## 6、component

目标组件的包或类名。

```java
    	intent.setComponent(new ComponentName(getApplicationContext(), TargetActivity.class));
    	intent.setComponent(new ComponentName(getApplicationContext(), "com.lizw.intent.TargetActivity"));
    	intent.setComponent(new ComponentName("com.lizw.other", "com.lizw.other.TargetActivity"));
```

前两种是用于匹配同一包内的目标，第三种是用于匹配其他包内的目标。

需要注意的是，如果我们在Intent中指定了component属性，系统将不会再对action、data/type、category进行匹配。













# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结



# 参考

