#1．概述

　　Fragment是Activity中用户界面的一个行为或者是一部分。主要是支持在大屏幕上动态和更为灵活的去组合或是交换UI组件，通过将activity的布局分割成若干个fragment，可以在运行时编辑activity的呈现，并且那些变化会被保存在由activity管理的后台栈里面。

　　**Fragment必须总是被嵌入到一个activity之中**，并且fragment的生命周期直接受其宿主activity的生命周期的影响。你可以认为fragment是activity的一个模块零件，它有自己的生命周期，接收它自己的输入事件，并且可以在activity运行时添加或者删除。

　　应该将每一个fragment设计为模块化的和可复用化的activity组件。也就是说，你可以在多个activity中引用同一个fragment，因为fragment定义了它自己的布局，并且使用它本身生命周期回调的行为。


#２．Fragment的生命周期

先看fragment生命周期图：

![这里写图片描述](http://img.blog.csdn.net/20160429134558410) 

　　fragment所生存的activity生命周期直接影响着fragment的生命周期，由此针对activity的每一个生命周期回调都会引发一个fragment类似的回调。例如，当activity接收到onPause()时，这个activity之中的每个fragment都会接收到onPause()。
　　[这有Activity的详细说明](http://blog.csdn.net/amazing7/article/details/51244219)

　　Fragment有一些额外的生命周期回调方法（创建和销毁fragment界面）．

 - onAttach()

　　当fragment被绑定到activity时调用（Activity会被传入）。

 - onCreateView()

　　将本身的布局构建到activity中去（fragment作为activity界面的一部分）
　　

 -  onActivityCreated()

　　当activity的onCreate()函数返回时被调用。

 - onDestroyView()

　　当与fragment关联的视图体系正被移除时被调用。

 - onDetach()

　　当fragment正与activity解除关联时被调用。

当activity接收到它的onCreate()回调时，activity之中的fragment接收到onActivityCreated()回调。

　　一旦activity处于resumed状态，则可以在activity中自由的添加或者移除fragment。因此，只**有当activity处于resumed状态时**，fragment的生命周期才可以独立变化。
　　
fragment会在　activity离开恢复状态时　再一次被activity推入它的生命周期中。

**管理fragment生命周期**与管理activity生命周期很相像。像activity一样，fragment也有三种状态：

 - Resumed

　　fragment在运行中的activity可见。

 - Paused

　　另一个activity处于前台且得到焦点，但是这个fragment所在的activity仍然可见（前台activity部分透明，或者没有覆盖全屏）。

 - Stopped

　　fragment不可见。要么宿主activity已经停止，要么fragment已经从activity上移除，但已被添加到后台栈中。一个停止的fragment仍然活着（所有状态和成员信息仍然由系统保留着）。但是，它对用户来讲已经不再可见，并且如果activity被杀掉，它也将被杀掉。

　　如果activity的进程被杀掉了，在activity被重新创建时，你需要恢复fragment状态。可以执行fragment的onSaveInstanceState()来保存状态（注意在fragment是在onCreate()，onCreateView()，或onActvityCreate()中进行恢复）。

　　在生命周期方面,activity与fragment之间一个**很重要的不同**，就是在各自的后台栈中是如何存储的。
　　当activity停止时，**默认**情况下activity被安置在由系统管理的activity后台栈中；　
　　fragment仅当在一个事务被移除时，通过显式调用addToBackStack()请求保存的实例，该fragment才被置于由宿主activity管理的后台栈。　

**要创建一个fragment**，必须创建一个fragment的子类。一般情况下，我们至少需要实现以下几个fragment生命周期方法：

> onCreate()

　　在创建fragment时系统会调用此方法。在实现代码中，你可以初始化想要在fragment中保持的那些必要组件，当fragment处于暂停或者停止状态之后可重新启用它们。

>onCreateView()

　　在第一次为fragment绘制用户界面时系统会调用此方法。为fragment绘制用户界面，这个函数必须要返回所绘出的fragment的根View。如果fragment没有用户界面可以返回空。


```
@Override
		public View onCreateView(LayoutInflater inflater, ViewGroup container,Bundle savedInstanceState) {　
		
			// Inflate the layout for this fragment
return inflater.inflate(R.layout.example_fragment, container, false);
		}
```
inflate()函数需要以下三个参数：

①要inflate的布局的资源ID。　

②被inflate的布局的父ViewGroup。

③一个布尔值，表明在inflate期间被infalte的布局是否应该附上ViewGroup（第二个参数container）。（在这个例子中传入的是false，因为系统已经将被inflate的布局插入到容器中（container）——传入true会在最终的布局里创建一个多余的ViewGroup。）　

> onPause()

　　系统回调用该函数作为用户离开fragment的第一个预兆（尽管这并不总意味着fragment被销毁）。在当前用户会话结束之前，通常要在这里提交任何应该持久化的变化（因为用户可能不再返回）。


#3.将fragment添加到activity之中

　　可以通过在activity布局文件中声明fragment，用fragment标签把fragment插入到activity的布局中，或者是用应用程序源码将它添加到一个存在的ViewGroup中。　
　　
　　但fragment并不是一个定要作为activity布局的一部分，fragment也可以为activity隐身工作。



##3.1在activity的布局文件里声明fragment

　　可以像为view一样为fragment指定布局属性。例如：

```
<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:orientation="horizontal"
		android:layout_width="match_parent"
		android:layout_height="match_parent">　
		
		<fragment android:name="com.example.test.FragmentOne"
				android:id="@+id/fo"
				android:layout_width="match_parent"
				android:layout_height="match_parent" />
	</LinearLayout>

```
　　fragment标签中的android:name 属性指定了布局中实例化的Fragment类。

　　当系统创建activity布局时，它实例化了布局文件中指定的每一个fragment，并为它们调用onCreateView()函数，以获取每一个fragment的布局。系统直接在<fragment>元素的位置插入fragment返回的View。

　　注意：每个fragment都需要一个唯一的标识，如果重启activity，系统可用来恢复fragment（并且可用来捕捉fragment的事务处理，例如移除）。为fragment提供ID有三种方法：

 - 

用android:id属性提供一个唯一的标识。　

 - 用android:tag属性提供一个唯一的字符串。　

 - 如果上述两个属性都没有，系统会使用其容器视图（view）的ID。　

##3.2通过编码将fragment添加到已存在的ViewGroup中

　　在activity运行的任何时候，你都可以将fragment添加到activity布局中。
　　
　　要管理activity中的fragment，可以使用FragmentManager。可以通过在activity中调用getFragmentManager()获得。使用FragmentManager 可以做如下事情，包括：

 - 使用findFragmentById()（用于在activity布局中提供有界面的fragment）或者findFragmentByTag()获取activity中存在的fragment（用于有界面或者没有界面的fragment）。　　

 - 使用popBackStack()（模仿用户的BACK命令）从后台栈弹出fragment。　　


 - 使用addOnBackStackChangedListener()注册一个监听后台栈变化的监听器。

在Android中，对Fragment的事务操作都是通过FragmentTransaction来执行。操作大致可以分为两类：

 - 显示：add() replace() show() attach()　　

 - 隐藏：remove() hide() detach()　

> 说明：
　　调用show() & hide()方法时，Fragment的生命周期方法并不会被执行，仅仅是Fragment的View被显示或者​隐藏。

>　　执行replace()时（至少两个Fragment），会执行第二个Fragment的onAttach()方法、执行第一个Fragment的onPause()-onDetach()方法，同时containerView会detach第一个Fragment的View。

>　　add()方法执行onAttach()-onResume()的生命周期，相对的remove()就是执行完成剩下的onPause()-onDetach()周期。


可以像下面这样从Activity中取得FragmentTransaction的实例：

```
FragmentManager fragmentManager = getFragmentManager()　
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
```

可以用add()函数添加fragment，并指定要添加的fragment以及要将其插入到哪个视图（view）之中（注意commit事务）：

```
ExampleFragment fragment = new ExampleFragment();
	fragmentTransaction.add(R.id.fragment_container, fragment);
	fragmentTransaction.commit();
```

##3.3添加没有界面的fragment 

　　也可以使用fragment为activity提供后台动作，却不呈现多余的用户界面。

　　想要添加没有界面的fragment ，可以使用add(Fragment, String)（为fragment提供一个唯一的字符串“tag”，而不是视图（view）ID）。这样添加了fragment，但是，因为还没有关联到activity布局中的视图（view） ，收不到onCreateView()的调用。所以不需要实现这个方法。　
　　
　　对于无界面fragment，字符串标签是**唯一识别**它的方法。如果之后想从activity中取到fragment，需要使用findFragmentByTag()。　


#4.fragment事务后台栈

　　在调用commit()之前，可以将事务添加到fragment事务后台栈中（通过调用addToBackStatck()）。这个后台栈由activity管理，并且允许用户通过按BACK键回退到前一个fragment状态。

　　下面的代码中一个fragment代替另一个fragment，并且将之前的fragment状态保留在后台栈中：

```
 Fragment newFragment = new ExampleFragment();
 FragmentTransaction transaction = getFragmentManager().beginTransaction();
 
 transaction.replace(R.id.fragment_container, newFragment);
 transaction.addToBackStack(null);

 transaction.commit();
```

> 注意：
> 
>　　 如果添加多个变更事务（例如另一个add()或者remove()）并调用addToBackStack()，那么在调用commit()之前的所有应用的变更被作为一个单独的事务添加到后台栈中，并且BACK键可以将它们一起回退。
> 
> 　　当移除一个fragment时，如果调用了addToBackStack()，那么之后fragment会被停止，如果用户回退，它将被恢复过来。
> 
>　　调用commit()并不立刻执行事务，相反，而是采取预约方式，一旦activity的界面线程（主线程）准备好便可运行起来。然而，如果有必要的话，你可以从界面线程调用executePendingTransations()立即执行由commit()提交的事务。
> 
> 　　只能在activity保存状态（当用户离开activity时）之前用commit()提交事务。如果你尝试在那时之后提交，会抛出一个异常。这是因为如果activity需要被恢复，提交后的状态会被丢失。对于这类丢失提交的情况，可使用commitAllowingStateLoss()


#５.与Activity交互

 - Activity中已经有了该Fragment的引用，直接通过该引用进行交互。

 -如果没引用可以通过调用fragment的函数findFragmentById()或者findFragmentByTag()，从FragmentManager中获取Fragment的索引，例如： 

```
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
```

 - 在Fragment中可以通过getActivity得到当前绑定的Activity的实例。

 - 创建activity事件回调函数，在fragment内部定义一个回调接口，宿主activity来实现它。



 

# 问题解答

- 1.Android中v4包下Fragment和app包下Fragment的区别是什么？

> [点击查看答案](https://www.cnblogs.com/as3lib/p/6129313.html)

- 2.Fragment的生命周期 & 请结合Activity的生命周期再一起说说。

> [点击查看答案](https://blog.csdn.net/clandellen/article/details/79269680)

- 3.说说Fragment如何进行懒加载。

> [点击查看答案](https://www.cnblogs.com/dasusu/p/6745032.html)

- 4.ViewPager + Fragment结合使用会出现内存泄漏吗 & 如何解决？

> **原因:**一般ViewPager + Fragment结合使用出现内存泄漏的原因可能用某个集合存储了Fragment的实例,导致当用户滑动ViewPager的时候，某一个Fragment即将面临销毁的时候，由于这个集合持有的它的引用，因此不能被回收掉,如果Fragment里面有大量的数据占据内存，有可能会导致OOM。  
> **解决方法**：尽量不要使用集合来存储Fragment实例对象，除非你有良好的二次封装。再就是要做好每一页Fragment的数据缓存问题。

- 5.Fragment如何和Activity进行通信 & Fragment之间如何进行通信？
- 6.给我谈谈Fragment3种切换的方式以及区别 & 使用场景。

> [5~6题答案](https://blog.csdn.net/clandellen/article/details/79269680)

- 7.getFragmentManager,getSupportFragmentManager,getChildFragmentManager之间的区别？

> [点击查看答案](https://blog.csdn.net/Allan_Bst/article/details/64920076)

- 8.FragmentPagerAdapter和FragmentStatePagerAdapter区别？

> [点击查看答案](https://www.cnblogs.com/nbls/p/7252307.html)





# 前言

- `Fragment`在 `Android`开发中非常常用
- 今天，我将讲解关于`Fragment`的使用

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-9a6422b22416b298.png?imageMogr2/auto-orient/strip|imageView2/2/w/482/format/webp)

Fragment介绍&使用方法解析.png

------

# 1. 定义

```
Activity`界面中的一部分，可理解为模块化的`Activity
```

> 1. `Fragment`不能独立存在，必须嵌入到`Activity`中
> 2. `Fragment`具有自己的生命周期，接收它自己的事件，并可以在`Activity`运行时被添加或删除
> 3. `Fragment`的生命周期直接受所在的`Activity`的影响。如：当`Activity`暂停时，它拥有的所有`Fragment`们都暂停

------

# 2. 作用

支持动态、灵活的界面设计

> 1. `Fragment`从 `Android 3.0`后引入
> 2. 在低版本`Android 3.0`前使用 `Fragment`，需要采用`android-support-v4.jar`兼容包

------

# 3. 生命周期解析

- 先来看官方说明图

![img](https:////upload-images.jianshu.io/upload_images/944365-cc85e7626552d866.png?imageMogr2/auto-orient/strip|imageView2/2/w/317/format/webp)

示意图

### 详解每个方法的调用场景

- onAttach方法
   Fragment和Activity建立关联的时候调用（获得activity的传递的值）
- onCreateView方法
   为Fragment创建视图（加载布局）时调用（给当前的fragment绘制UI布局，可以使用线程更新UI）
- onActivityCreated方法
   当Activity中的onCreate方法执行完后调用（表示activity执行oncreate方法完成了的时候会调用此方法）
- onDestroyView方法
   Fragment中的布局被移除时调用（表示fragment销毁相关联的UI布局）
- onDetach方法
   Fragment和Activity解除关联的时候调用（脱离activity）

### Fragment生命周期解析

- 当一个fragment被创建的时候：
   onAttach()
   onCreate()
   onCreateView()
   onActivityCreated()
- 当这个fragment对用户可见的时候，它会经历以下状态。
   onStart()
   onResume()

> 1.2可以理解为从创建到显示（或切换）

- 当这个fragment进入“后台模式”的时候，它会经历以下状态。
   onPause()
   onStop()
- 当这个fragment被销毁了（或者持有它的activity被销毁了）：
   onPause()
   onStop()
   onDestroyView()
   onDestroy()
   onDetach()
- 就像Activity一样，在以下的状态中，可以使用Bundle对象保存一个fragment的对象。
   onCreate()
   onCreateView()
   onActivityCreated()

### 其他场景的调用

- 屏幕灭掉
   onPause() onSaveInstanceState() onStop()
- 屏幕解锁
   onStart() onResume()
- 切换到其他Fragment
   onPause() onStop() onDestroyView()
- 切换回本身的Fragment
   onCreateView() onActivityCreated() onStart() onResume()
- 回到桌面
   onPause() onSaveInstanceState() onStop()
- 回到应用
   onStart() onResume()
- 退出应用
   onPause() onStop() onDestroyView() onDestroy() onDetach()

### Fragment和Activity的生命周期很相似，以下是对比图

![img](https:////upload-images.jianshu.io/upload_images/944365-0f9670e55a52403c.png?imageMogr2/auto-orient/strip|imageView2/2/w/340/format/webp)

------

# 4. 具体使用

- 由于`Fragment`作为`Activity`一部分，所以`Fragment`的使用一般是添加到`Activity`中

- 将

  ```
  Fragment
  ```

  添加到

  ```
  Activity
  ```

  中一般有2种方法:

  1. 在`Activity`的`layout.xml`布局文件中静态添加
  2. 在`Activity`的`.java`文件中动态添加

### 方法1：在`Activity`的`layout.xml`布局文件中静态添加

- `Activity`的布局文件

*fragment_layout_test.xml*



```jsx
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

// 该fragment类定义在包名为"com.skywang.app"中的FragmentLayoutTest类的内部类ExampleFragment中
   <fragment android:name="com.skywang.app.FragmentLayoutTest$ExampleFragment"
        android:id="@+id/list"
        android:layout_weight="1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
  
</LinearLayout>
```

- `Fragment`的布局文件

*example_fragment.xml*



```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView
        android:text="@string/example_fragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
   
</LinearLayout>
```

- `Activity`的.java文件

*FragmentLayoutTest.java*



```java
// 在Activity使用Fragment时，需要考虑版本兼容问题
// 1. Android 3.0后，Activity可直接继承自Activity，并且在其中嵌入使用Fragment
// 2. Android 3.0前，Activity需FragmentActivity（其也继承自Activity），同时需要导入android-support-v4.jar兼容包，这样在Activity中才能嵌入Fragment

public class FragmentLayoutTest extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.fragment_layout_test);
        // 设置上述布局文件
    }

    // 继承自Fragment
    // 布局文件中的Fragment通过该FragmentLayoutTest的内部类ExampleFragment实现
    public static class ExampleFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                 Bundle savedInstanceState) {
            
            return inflater.inflate(R.layout.example_fragment, container, false);
             // 将example_fragment.xml作为该Fragment的布局文件
            // 即相当于FragmentLayoutTest直接调用example_fragment.xml来显示到Fragment中
        }
    }
}
```

至此，方法1讲解完毕。

------

# 方法2：在Activity的.java文件中动态添加

- 步骤1：在`Activity`的布局文件定义1占位符（`FrameLayout`）
   这样做的好处是：可动态在`Activity`中添加不同的 `Fragment`，更加灵活

*fragment_transaction_test.xml*



```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    
    <FrameLayout
        android:id="@+id/about_fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    
</LinearLayout>
```

- 步骤2：设置`Fragment`的布局文件

*example_fragment.xml*



```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView
        android:text="@string/example_fragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
   
</LinearLayout>
```

- 步骤3：在`Activity`的`.java`文件中动态添加`Fragment`

*FragmentTransactionTest*



```java
public class FragmentTransactionTest extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.fragment_transaction_test);
        
        // 步骤1：获取FragmentManager
        FragmentManager fragmentManager = getFragmentManager();

        // 步骤2：获取FragmentTransaction        
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        
        // 步骤3：创建需要添加的Fragment ：ExampleFragment
        ExampleFragment fragment = new ExampleFragment();

        // 步骤4：动态添加fragment
        // 即将创建的fragment添加到Activity布局文件中定义的占位符中（FrameLayout）
        fragmentTransaction.add(R.id.about_fragment_container, fragment);
        fragmentTransaction.commit();
    }
    
    // 继承与Fragment
    public static class ExampleFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                 Bundle savedInstanceState) {
            
            return inflater.inflate(R.layout.example_fragment, container, false);
            // 将example_fragment.xml作为该Fragment的布局文件
        }
    }
}
```

至此，方法2讲解完毕

------

# 5. 总结



# Carson带你学Android：3分钟全面解析Fragment生命周期

# 前言

- `Android`开发中，会经常接触 `Fragment`，所以深入了解`Fragment`生命周期非常重要
- 本文将深入讲解`Fragment`生命周期 的相关内容

------

# 目录

![img](https:////upload-images.jianshu.io/upload_images/944365-fdaa1e7f3f78771c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 1. 生命周期流程 & 方法详解

![img](https:////upload-images.jianshu.io/upload_images/944365-db402563d4da3a2c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 2. 常见场景的生命周期调用方式

![img](https:////upload-images.jianshu.io/upload_images/944365-64745b22288f10a4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 3. 与Activity生命周期对比

- `Fragment`、`Activity`的生命周期非常相似
- 具体对比如下图：

![img](https:////upload-images.jianshu.io/upload_images/944365-0f9670e55a52403c.png?imageMogr2/auto-orient/strip|imageView2/2/w/340/format/webp)

------

# 4. 总结

- 本文对 `Android` 的 `Fragment`的生命周期进行了全面介绍





# 参考

[Carson带你学Android：Fragment最全面介绍 & 使用方法解析](https://www.jianshu.com/p/2bf21cefb763)

[Carson带你学Android：3分钟全面解析Fragment生命周期](https://www.jianshu.com/p/dd37c4dca5ea)
