性能优化相关
共用的资源只加载一次。
自定义View中默认的颜色，图片等，都只要加载一次；需要注意，size如果是用dp2px的，不可以缓存，在切换分辨率或者字体大小后，dp2px会计算出新的值；
gc前后，内存对象对比。


UI相关
设置底部导航栏透明，看得到导航栏后面的内容
<item name="android:navigationBarColor">@android:color/transparent</item>

// 这两个是要在 v29/style 才能使用。
<item name="android:enforceStatusBarContrast">false</item>
<item name="android:enforceNavigationBarContrast">false</item>
theme style 中增加上面三行。




