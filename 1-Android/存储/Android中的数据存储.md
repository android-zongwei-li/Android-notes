> version：2022/10/7
>
> review：



目录

[TOC]





# 一、前置知识



# 二、概述

Android 提供了 5 种方式来让用户保存持久化应用程序数据，分别是： 

① SharedPreferences

② 文件

③ SQLite 数据库

④ ContentProvider

⑤ 网络（云端存储）

需要根据自己的需求来做选择，比如数据是否是应用程序私有的，是否能被其他程序访问，需要多少数据存储空间等。

Android 提供了一种方式来暴露你的数据（甚至是私有数据）给其他应用程序 - ContentProvider。它是一个可选组件，可公开读写你应用程序的数据。



# 三、SharedPreferences

SharedPreferences 可以用来保存键值对（key-value），支持所有的基本数据类型（boolean、float、int、long、string）。

通常用来存储一些配置信息。存储在 **data/data/程序包名/shared_prefs 目录** 下。



优缺点：使用方便。但是可存储的数据类型单一（只有基本数据类型），不能进行条件查询。



# 四、文件

## 4.1 内部存储

当文件被保存在内部存储中时，默认情况下，文件是应用程序私有的，其他应用不能访问。当用户卸载应用程序时这些文件也会被删除。 

文件默认存储位置：/data/data/包名/files/文件名。

### 4.1.1 创建和写入一个内部存储的私有文件

① 调用 Context 的 openFileOutput() 函数，填入文件名和操作模式，它会返回一个 FileOutputStream 对象。 

② 通过 FileOutputStream 对象的 write() 函数写入数据。 

③ FileOutputStream 对象的 close () 函数关闭流。 

例如： 

```java
String FILENAME = "a.txt"; 
String string = "test"; 
try { 
FileOutputStream fos = openFileOutput(FILENAME, Context.MODE_PRIVATE); 
fos.write(string.getBytes()); 
fos.close(); 
} catch (Exception e) { 
e.printStackTrace(); 
}
```

在 openFileOutput(String name, int mode)方法中 name 参数: 用于指定文件名称，不能包含路径分隔符“/” ，如果文件不存在，Android 会自动创建它。 

mode 参数用于指定操作模式，分为四种： 

Context.MODE_PRIVATE = 0 ：默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容。 

Context.MODE_APPEND = 32768 ：该模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。 

Context.MODE_WORLD_READABLE = 1 ：表示当前文件可以被其他应用读取。 

MODE_WORLD_WRITEABLE ：表示当前文件可以被其他应用写入。

### 4.1.2 读取一个内部存储的私有文件

① 调用 openFileInput( )，参数中填入文件名，会返回一个 FileInputStream 对象。

② 使用流对象的 read()方法读取字节 

③ 调用流的 close()方法关闭流

例如： 

```java
String FILENAME = "a.txt"; 

try { 
FileInputStream inStream = openFileInput(FILENAME); 
int len = 0;
byte[] buf = new byte[1024]; 
StringBuilder sb = new StringBuilder(); 
while ((len = inStream.read(buf)) != -1) { 
sb.append(new String(buf, 0, len)); 
}
inStream.close(); 
} catch (Exception e) { 
e.printStackTrace(); 
} 
```

其他一些经常用到的方法： 

getFilesDir()： 得到内存储文件的绝对路径

getDir()： 在内存储空间中**创建**或**打开一个已经存在**的目录

deleteFile()： 删除保存在内部存储的文件。 

fileList()： 返回当前由应用程序保存的文件的数组（内存储目录下的全部文件）。 

### 4.1.3 保存编译时的静态文件

如果你想在应用编译时保存静态文件，应该把文件保存在项目的 **res/raw/** 目录下，你可以通过openRawResource()方法去打开它（传入参数 R.raw.filename），这个方法返回一个 InputStream 流对象你可以读取文件但是不能修改原始文件。 

```java
InputStream is = this.getResources().openRawResource(R.raw.filename);
```

### 4.1.4 保存内存缓存文件

有时候我们只想缓存一些数据而不是持久化保存，可以使用 getCacheDir（）去打开一个文件，文件的存储目录（ /data/data/包名/cache ）是一个应用专门用来保存临时缓存文件的内存目录。 

当设备的内部存储空间比较低的时候，Android 可能会删除这些缓存文件来恢复空间，但是你不应该依赖系统来回收，要自己维护这些缓存文件把它们的大小限制在一个合理的范围内，比如 1ＭＢ。当你卸载应用的时候这些缓存文件也会被移除。



## 4.2 外部存储（sdcard）

因为内部存储容量限制，有时候需要存储的数据比较大的时候需要用到外部存储，使用外部存储分为以下几个步骤：

### 4.2.1 添加外部存储访问限权

首先，要在 AndroidManifest.xml 中加入访问 SDCard 的权限，如下: 

```xml
<!-- 在 SDCard 中创建与删除文件权限 --> 

<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/> 

<!-- 往 SDCard 写入数据权限 --> 

<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/> 
```

### 4.2.2 检测外部存储的可用性

在使用外部存储时我们需要检测其状态，它可能被连接到计算机、丢失或者只读等。下面代码将说明如何检查状态： 

```java
//获取外存储的状态 
String state = Environment.getExternalStorageState(); 
if (Environment.MEDIA_MOUNTED.equals(state)) { 
// 可读可写 
mExternalStorageAvailable = mExternalStorageWriteable = true; 
} else if (Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) { 
// 可读 
} else { 
// 可能有很多其他的状态，但是我们只需要知道，不能读也不能写 
}
```

### 4.2.3 访问外部存储器中的文件

１、如果 **API** **版本大于或等于８**

使用 getExternalFilesDir (String type) 

该方法打开一个外存储目录，此方法需要一个类型，指定你想要的子目录， 如类型参数 DIRECTORY_MUSIC 和 DIRECTORY_RINGTONES（传 null 就是你应用程序的文件目录的根目录）。通过指定目录的类型，确保 Android 的媒体扫描仪将扫描到分类系统中的文件（例如，铃声被确定为铃声）。如果用户卸载应用程序，这个目录及其所有内容将被删除。 

例如： 

```java
File file = new File(getExternalFilesDir(null), "test.jpg"); 
```

**２、如果** **API** **版本小于** **8** （7 或者更低） 

getExternalStorageDirectory () 

通过该方法打开外存储的根目录，你应该在以下目录下写入你的应用数据，这样当卸载应用程序时该目录及其所有内容也将被删除。 

/Android/data/<package_name>/files/

读写数据： 

```java
if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) { 
File sdCardDir = Environment.getExternalStorageDirectory();//获取 SDCard 目录 "/sdcard" 
File saveFile = new File(sdCardDir,"a.txt"); 
//写数据 
try { 
FileOutputStream fos = new FileOutputStream(saveFile);
fos.write("test".getBytes()); 
fos.close(); 
} catch (Exception e) { 
e.printStackTrace(); 
}

//读数据 
try { 
FileInputStream fis= new FileInputStream(saveFile); 
int len =0; 
byte[] buf = new byte[1024]; 
StringBuffer sb = new StringBuffer(); 
while((len=fis.read(buf))!=-1){ 
sb.append(new String(buf, 0, len)); 
}
fis.close(); 
} catch (Exception e) { 
e.printStackTrace(); 
} 
} 
```

我们也可以在 /Android/data/package_name/cache/目录下做外部缓存。 



五、SQLite 数据库



六、网络





























# 相关问题

**1、描述一下 Android 数据持久存储方式？**

**SharedPreferences 存储**：一种轻型的数据存储方式，本质是基于 XML 文件存储的 key-value 键值对数据，通常用来存储一些简单的配置信息（如应用程序的各种配置信息）

**SQLite 数据库存储**：一种轻量级嵌入式数据库引擎，它的运算速度非常快，占用资源很少，常用来存储大量复杂的关系数据；

**ContentProvider**：四大组件之一，用于数据的存储和共享，不仅可以让不同应用程序之间进行数据共享，还可以选择只对哪一部分数据进行共享，可保证程序中的隐私数据不会有泄漏风险；

**File 文件存储**：写入和读取文件的方法和Java 中实现 I/O 的程序一样；

**网络存储**：主要在远程的服务器中存储相关数据，用户操作的相关数据可以同步到服务器上；

**2、SharedPreferences 的应用场景？注意事项？**

SharedPreferences 是一种轻型的数据存储方式，本质是基于 XML 文件存储的 key-value 键值对数据，通常用来存储一些简单的配置信息，如 int，String，boolean、float 和 long； 

注意事项：

勿存储大型复杂数据，这会引起内存 GC、阻塞主线程使页面卡顿产生 ANR

勿在多进程模式下，操作 Sp

不要多次 edit 和 apply，尽量批量修改一次提交

建议 apply，少用 commit

**3、SharedPrefrences 的 apply 和 commit 有什么区别？**

apply 没有返回值而 commit 返回 boolean 表明修改是否提交成功。

apply 是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而 commit 是**同步**的提交到硬件磁盘，因此，在多个并发的提交 commit 的时候，他们会等待正在处理的commit 保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内存，后面有调用 apply 的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。

apply 方法没有任何失败的提示。由于在一个进程中，sharedPreference 是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用 apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。

**4、了解 SQLite 中的事务操作吗？是如何做的**

SQLite 在做 CRDU（create、read、delete、update） 操作时都默认开启了事务，然后把 SQL语句翻译成对应的 SQLiteStatement 并调用其相应的 CRUD方法，此时整个操作还是在 rollback journal 这个临时文件上进行，只有操作顺利完成才会更新 db 数据库，否则会被回滚

**5、使用 SQLite 做批量操作有什么好的方法吗？**

使用 SQLiteDatabase 的 beginTransaction 方法开启一个事务，将批量操作 SQL 语句转化为 SQLiteStatement 并进行批量操作，结束后 endTransaction()

**6、如何删除 SQLite 中表的个别字段**

SQLite 数据库只允许增加字段而不允许修改和删除表字段，只能创建新表保留原有字段，删除原表

**7、使用 SQLite 时会有哪些优化操作？**

使用事务做批量操作

及时关闭 Cursor，避免内存泄露

耗时操作异步化：数据库的操作属于本地 IO 耗时操作，建议放入异步线程中处理

ContentValues 的容量调整：ContentValues 内部采用HashMap 来存储 Key-Value 数据，ContentValues 初始容量为 8，扩容时翻倍。因此建议对 ContentValues 填入的内容进行估量，设置合理的初始化容量，减少不必要的内部扩容操作

使用索引加快检索速度：对于查询操作量级较大、业务对查询要求较高的推荐使用索引



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结





# 参考

