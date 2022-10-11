# 一、获取APK

1、如果有apk的话，直接拷贝到电脑上就行
 2、如果Android设备上的apk已经被删除了，可以通过下面的方法获取
 1）获取当前打开App的包名

```dart
adb shell dumpsys window | findstr mCurrentFocus
```

2）获取到对应的apk文件路径

```cpp
adb shell pm list package -f packageName
```

3）将apk复制到本地

```undefined
adb pull apk地址 导入目标文件
```

4）示例：获取微信Apk

```kotlin
C:\Users\NJCS>adb shell dumpsys window | findstr mCurrentFocus
* daemon not running; starting now at tcp:5037
* daemon started successfully
  mCurrentFocus=Window{56ee0e0 u0 com.lbe.security.miui}
  mCurrentFocus=Window{6d0a48d u0 com.tencent.mm/com.tencent.mm.ui.LauncherUI}

C:\Users\NJCS>adb shell pm list package -f com.tencent.mm
package:/data/app/com.tencent.mm-U4FAEMQ7dpLdAJRTp1EvXw==/base.apk=com.tencent.mm

C:\Users\NJCS>adb pull /data/app/com.tencent.mm-U4FAEMQ7dpLdAJRTp1EvXw==/base.apk D:\论文\OneDrive\桌面\Apps
/data/app/com.tencent.mm-U4FAEMQ7dpLdAJRTp1EvXw==/base.apk... pulled, 0 skipped. 12.6 MB/s (167830040 bytes in 12.698s)
```

![image-20221011113216829](images/反编译APK获取资源&代码/image-20221011113216829-5459141.png)

# 二、获取Apk资源

### 1、把Apk的后缀.apk改为.zip，然后解压出来

![image-20221011113253436](images/反编译APK获取资源&代码/image-20221011113253436-5459176.png)

> 到此我们可以获取一些.png，或者.jpg这样的位图文件资源，如果是xml类的资源，打开我们会发现是乱码，并且假如我们想看APK程序的Java代码，也是行不通的，因为他们都打被打包到classes.dex文件中了。
>  另外，切勿拿反编译来做违法的事，比如把人家的APK重新打包后使用自己的签名然后发布到相关市场...另外，我们是参考别人的代码，而不是完全拷贝！！！切记！！

### 2、然后，要准备三个工具

1. **apktool：**获取资源文件，提取图片文件，布局文件，还有一些XML的资源文件
2. **dex2jar：**将APK反编译成Java源码(将classes.dex转化为jar文件)
3. **jd-gui：**查看2中转换后的jar文件，即查看Java文件 为了方便各位读者，这里将三个打包到一起放到云盘中，又需要的可以进行下载： [反编译相关的三个工具.zip](https://links.jianshu.com/go?to=http%3A%2F%2Fstatic.runoob.com%2Fdownload%2F%E5%8F%8D%E7%BC%96%E8%AF%91%E7%9B%B8%E5%85%B3%E7%9A%84%E4%B8%89%E4%B8%AA%E5%B7%A5%E5%85%B7.zip)

### 3、使用apktool反编译APK获得图片与XML资源

双击cmd.exe，来到命令行，键入： apktool.bat d csdn.apk 即可，Enter回车。
 然后就可以看到生成的weixin文件夹，里面就有我们想要资源。

![image-20221011113429506](images/反编译APK获取资源&代码/image-20221011113429506.png)

注：如果出现以下错误

```css
D:\论文\OneDrive\桌面\反编译相关的三个工具\apktool2.2>apktool.bat d D:\论文\OneDrive\桌面\Apps\weixin.apk
I: Using Apktool 2.0.0-Beta9 on weixin.apk
I: Loading resource table...
Exception in thread "main" brut.androlib.AndrolibException: Could not decode arsc file
        at brut.androlib.res.decoder.ARSCDecoder.decode(ARSCDecoder.java:54)
        at brut.androlib.res.AndrolibResources.getResPackagesFromApk(AndrolibResources.java:604)
        at brut.androlib.res.AndrolibResources.loadMainPkg(AndrolibResources.java:74)
        at brut.androlib.res.AndrolibResources.getResTable(AndrolibResources.java:66)
        at brut.androlib.Androlib.getResTable(Androlib.java:49)
        at brut.androlib.ApkDecoder.decode(ApkDecoder.java:93)
        at brut.apktool.Main.cmdDecode(Main.java:169)
        at brut.apktool.Main.main(Main.java:85)
Caused by: java.io.IOException: Expected: 0x001c0001, got: 0x00000000
        at brut.util.ExtDataInput.skipCheckInt(ExtDataInput.java:48)
        at brut.androlib.res.decoder.StringBlock.read(StringBlock.java:43)
        at brut.androlib.res.decoder.ARSCDecoder.readPackage(ARSCDecoder.java:95)
        at brut.androlib.res.decoder.ARSCDecoder.readTable(ARSCDecoder.java:81)
        at brut.androlib.res.decoder.ARSCDecoder.decode(ARSCDecoder.java:49)
        ... 7 more
```

> 1.登陆[http://ibotpeaches.github.io/Apktool/](https://links.jianshu.com/go?to=http%3A%2F%2Fibotpeaches.github.io%2FApktool%2F) 下载最新版本的apktool.jar，目前最新版本为2.x
>  2.将下载到的apktool_xxx.jar文件改名为apktool.jar,然后替换掉老版本的apktool.jar
>  3.现在可以正常反编译apk文件了

### 4、使用dex2jar将classes.dex转换成jar文件

1）打开dex2jar文件夹，把解压后的apk中的classes.dex复制到dex2jar.bat所在的目录下：

![image-20221011113502594](images/反编译APK获取资源&代码/image-20221011113502594.png)

2）进入dex2jar.bat所在的目录中，执行命令：d2j-dex2jar.bat classes.dex

```css
D:\论文\OneDrive\桌面\反编译相关的三个工具\dex2jar-2.0>d2j-dex2jar.bat classes.dex
dex2jar classes.dex -> .\classes-dex2jar.jar
```

3）然后可以看到多了一个classes-dex2jar.jar文件：

![image-20221011113523341](images/反编译APK获取资源&代码/image-20221011113523341.png)

到此，转换完成。

### 5、使用jd-gui查看jar包中的Java代码

![image-20221011113550510](images/反编译APK获取资源&代码/image-20221011113550510.png)
