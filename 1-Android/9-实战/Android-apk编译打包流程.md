![img](https://upload-images.jianshu.io/upload_images/9000209-7d0db4157b77ef06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



典型Android应用模块的构建流程通常按照一下步骤执行：

编译器会将源代码转换成 DEX 文件（Dalvik可执行文件，其中包括在 Android 设备上运行的字节码），并将其他所有内容转换成编译后的资源

APK 打包器将 DEX 文件和编译后的资源合并到一个 APK 中。不过，在将应用安装并部署到 Android 设备之前，必须先为 APK 签名

APK 打包器使用调试或发布密钥库为 APK 签名：

如果您构建的是调试版应用（即专用于测试和分析的应用），则打包器会使用调试密钥库作为应用签名。Android Studio 会自动使用调试密钥库配置新项目

如果您构建的是打算对外发布的发布版应用，则打包器会使用发布密钥库作为应用签名。要创建发布密钥库，请参阅 [在 Android Studio 中为应用签名](https://developer.android.google.cn/studio/publish/app-signing#studio)

在生成最终 APK 之前，打包器会使用[ zipalign ](https://developer.android.google.cn/studio/command-line/zipalign)工具对应用进行优化，以减少其在设备上运行时占用的内存。

构建流程结束时，您将获得应用的调试版 APK 或发布版 APK，以用于部署、测试或发布给外部用户。



下面是比较详细的构建流程：



![img](https://upload-images.jianshu.io/upload_images/9000209-0ed65b13f21921b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Android APK 编译打包流程

椭圆形部分是编译打包过程中所需使用的工具（tools）

长方形部分是Android应用程序的各种资源文件

首先看中间的部分：

1、第一步：Java Compiler(java编译器)把java代码编译成.class文件。

java代码有三个部分：R.java、Application Source Code、JavaInterfaces

R.java:由aapt工具生成，aapt后面介绍

Application Source Code

JavaInterfaces: （如果有.aidl文件的话）aidl工具会将.aidl files生成java接口文件。

注：AIDL是Android Interface Definition Language的缩写，是Android跨进程通讯的一种方式，该阶段会检索Project中所有的aidl文件，并转换为对应的Java文件。

2、第二步：dex工具把.class文件 + 3rd Party Libraries文件生成Delvik可执行的.dex文件

3rd Party Libraries文件:依赖的第三方库

在此过程中，dex工具不仅会把java字节码转化为Delvik字节码，还会压缩常量池，消除冗余信息等

3、第三步：apkbuilder工具将Compiled Resources + .dex files + Other Resources生成 .apk 文件

Compiled Resources:由aapt工具生成

Other Resources:

4、第四步：Jarsigner工具使用Debug or Release Keystore对 .apk 文件进行打包

5、第五步：zipalign工具将Signed .apk（已签名apk）进行对齐（优化签名包），生成最终的 .apk 文件



0、这个算是第零步：

aapt（Android Asset Packaging Tool，即Android资源打包工具）

aapt工具会根据res下面的资源和AndroidManifest.xml生成一个资源索引文件（resource.arsc，其中包含了各个资源文件的映射， 可以理解为索引， 通过该文件能找到对应的资源文件信息。）和一个R.java文件。

assets下面的资源不编译，直接压缩到apk中。



总结下来，整个过程就是：

编译 -》dex -》打包 -》签名和对齐



面试关于apk打包的问题：

1、apk编译打包流程

2、为什么要用aapt把xml文件编译成二进制文件?

首先二进制格式的XML文件占用空间更小。因为所有的XML元素的标签、属性名称、属性值和内容所涉及到的字符串都会被统一收集到一个字符串资源池中去，并且会去重。有了这个字符串资源池，原来使用字符串的地方就会被替换成一个索引字符串资源池的整数值，从而可以减少文件的大小

其次是二进制的XML文件解析速度更快，这是由于二进制的XML元素里面不再包含有字符串值，因此可以避免了进行字符串解析，从而提高速度。

3、Dex打包关于65536的问题

这个问题是由于DEX文件格式限制，一个DEX文件中的method个数使用原生类型short来索引文件的方法，也就是4个字节共计最多表达65536个method，field/class个数也均有此限制，对于DEX文件，则是将工程所需要全部class文件合并压缩到一个DEX文件期间，也就是Android打包的DEX过程中，单个DEX文件可被引用的方法总数(自己开发的代码以及所引用的Android框架、类库的代码)被限制为66536。

4、打包流程中最后一步，为什么要对齐？

对齐是为了加快资源的访问速度。如果每个资源的开始位置上都是一个资源之后的4*n字节，那么访问下一个资源就不用遍历，直接跳到4*字节处判断是不是一个新的资源即可。有点类似于资源数组化，数组的访问速度当然比链表快

5、Android是怎么通过R文件找到真正的资源文件?

aapt工具对每个资源文件都生成了唯一的ID，这些ID保存在R.java文件中。资源ID是一个4字节的的无符号证书，在R.java文件中用16进制表示。其中，最高的1字节表示Package ID，次高1个字节表示Type ID，最低2字节表示Entry ID。只有一个ID 如何能引用到实际资源？实际上aapt工具还生成一个文件resources.arsc，相当于一个资源索引表，或者你理解成一个map也行，map的key是资源ID，value是资源在apk文件中的路径。resource.arsc里面还有其他信息，这里就不多说了。通过resource.arsc配合，就能引用到实际的资源文件。