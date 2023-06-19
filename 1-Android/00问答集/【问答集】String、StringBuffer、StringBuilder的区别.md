> 知识路径：Android > Java
>
> version：2020/09/28
>
> review：2020/09/28---2021/4/1

<font color='orange'>Q1、String,StringBuffer,StringBuilder的区别?</font>

> 从三方面回答：是否可变、效率、同步、适用范围。
>

### 一、String（字符串常量）

在Java中，字符串属于对象。对应的类是String。
对于String，除了基本的操作以外，我们还需要知道：
**String的值是不可变的。**

```java
String s = "hello";
s = s + " world";
System.out.print(s);// result : hello world
```
如果简单的从代码来看，s 确实改变了，但是我们需要知道引用和对象的区别，在这里，s 是 String 对象的一个引用，指向“hello”,在执行s + " world";操作后，s实际上是指向了一个新的对象，这个对象的值为“hello world”。我们所说的常量，指的也就是String对象的值不改变，改变了，也就是换了一个对象了。

内存图如下：

![String + 操作内存变化图](https://upload-images.jianshu.io/upload_images/9000209-d2999138693d0b39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分析：
可以看到简单的一次 + 操作，一共创建了三个对象。这样不仅效率低下，而且大量这种操作也会浪费内存空间。因此Google引入了两个新的类StringBuffer和StringBuilder来对这种变化字符串进行处理。

思考：String为什么是不可变的？
上源码：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    ...
}
```
分析：String类使用final修饰，因此String类是不能被继承的。所以当有一个String s = "hello"时，s引用我们可以肯定是指向String类型的对象，而不是String类型的子类，从Jvm的角度考虑，提高了一定的效率。然后可以看到char calue[]也是final，因此char数组也不能改变，只能赋值一次，并且数组长度确定后，不能扩容，因此String对象一旦创建，char数组也刚好够存下一个字符串，以后就没法扩容了。

### 二、StringBuffer和StringBuilder（字符串变量）

和String不同的是，StringBuffer和StringBuilder的值是可以改变的。

当**对字符串进行频繁修改**时，建议使用StringBuffer和StringBuilder类。
和String不同，StringBuffer和StringBuilder类的对象有一个共同点：
**被多次修改并不会产生新的未使用对象**。

说完了共同点，说一下区别：
StringBuilder类是在Java 5中加入的，它和StringBuffer最大的区别在于：
StringBuilder 的方法不是线程安全的（不能同步访问）。

疑问：线程安全？

由于**StringBuilder相较于StringBuffer有速度优势**，所以多数情况下建议使用StringBuilder类。当然了，如果程序要求线程安全的话，就必须使用StringBuffer了。

关于线程安全与同步的问题，看下源码：
StringBuilder：
```java
public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, Appendable, CharSequence {
   public StringBuilder() {
        super(16);
   }
   public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
    public StringBuilder delete(int start, int end) {
        super.delete(start, end);
        return this;
    }
}
```

StringBuffer：
```java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, Appendable, CharSequence
{
    public StringBuffer() {
        super(16);
    }
    public synchronized StringBuffer append(int i) {
        super.append(i);
        return this;
    }
    public synchronized StringBuffer delete(int start, int end) {
        super.delete(start, end);
        return this;
    }
}
```

可以看到StringBuffer的append（），delete（）方法多了个synchronized修饰。

### 三、三者的继承结构

![String、StringBuffer、StringBuilder的继承结构](https://upload-images.jianshu.io/upload_images/9000209-af279e5490995259.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 四、三者的速度区别

从执行效率快慢来看：
String < StringBuffer < StringBuilder
但是存在一种情况，就是当String全是字面量相加的时候，这种情况会很快，因为字面量在编译期时，编译器会优化处理，将字面量全部合成一个字面量然后扔进方法区的常量池中，所以运行时当在执行S1指向的时候，这个对象就已经存在与常量池中了，不需要计算了。而StringBuffer 则需要在运行时进行append操作，所以这就造成了这种现象。

```java
//String效率是远要比StringBuffer快的：
String S1 = “This is only a” + “ simple” + “ test”;
StringBuffer Sb = new StringBuilder(“This is only a”).append(“simple”).append(“ test”);
```
把上面的代码换为：
```java
//String速度是非常慢的：
String S2 = “This is only a”;
String S3 = “ simple”;
String S4 = “ test”;
String S1 = S2 + S3 + S4;
```
此时，String的执行效率就低了很多，因为运行时需要重新创建一个对象，将S2，S3 ，S4的值相加后再复制给这个新对象，S1再重新指向这个新对象。

疑问：常量池（https://blog.csdn.net/alankin/article/details/80408988），方法区

### 五、总结

1、StringBuffer和StringBuilder非常相似，均为可变的字符串序列，方法也一样
2、String：不可变字符串
3、StringBuffer：可变字符串，线程安全，效率低
4、StringBuilder：可变字符串，线程不安全，效率高
5、

![image.png](https://upload-images.jianshu.io/upload_images/9000209-20d0e87af721746a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后，再总结一下：
1、如果操作数据少，使用String，毕竟方便呀
2、单线程操作大量数据，使用StringBuilder
3、多线程操作大量数据，使用StringBuffer

可以使用toString()将StringBuffer/StringBuilder转化为String。

参考：
1、[https://blog.csdn.net/u011702479/article/details/82262823](https://blog.csdn.net/u011702479/article/details/82262823)
2、[https://blog.csdn.net/alankin/article/details/80406975](https://blog.csdn.net/alankin/article/details/80406975)

