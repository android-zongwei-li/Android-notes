> version：2021/10/
>
> review：



目录

[TOC]



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



# 相关问题

## 一、String

<font color='orange'>Q：String为什么是不可变的？（字节跳动）</font>

String类中通过一个char数组value[]来保存每个字符，然后value本身也是用final修饰的，这样value数组引用就不能改变内存地址了，但是因为value数组引用的对象是可以改变的，因此在使用value数组的时候就需要特别注意不能改变对象的内容，不然就达不到不可变的效果。因此呢，在String类封装好以后，就需要加一个final来修饰String类本身，使其不能被继承，进而保证了String类的功能完善且不可改变，因为没有改变的必要了，如果可以继承，然后重写了方法，那功能可能就会受损。

> [在java中String类为什么要设计成final？](https://www.zhihu.com/question/31345592)

<font color='orange'>Q：String为什么设计成final的？</font>

String和其他的基本数据类型一样重要，在java中有大量的使用，特别是在集合中，因此设计成不可变的是很有必要的，这样能够保证数据的**使用安全**，如果String可以被继承，那它的方法可能就会被重写，java又是支持多态的，这样在运行的时候，结果就是不可预期的。

然后是**线程安全**，如果一个String类是可变的，那可能就会有多个引用指向同一个对象，这样就容易被改变，导致数据不一致，引起同步问题。

综上，将String设计成final就一劳永逸，很省事。

一句话，就是防止被子类改变其含义。

<font color='orange'>Q：Strings=new String(“”);创建了几个对象?</font>

如果测量池中有 “” 对象，则创建一个new String("")放到堆中。

如果常量池中没有 “” 对象，会创建两个，一个是“”放到常量池中，一个是new String("")放到堆中。

<font color='orange'>Q：Stringa=new String("abc")与Stringa="abc"的区别</font>

new String("abc");存放在堆中，“abc”存放在常量池中。

Stringa="abc"可能创建一个或不创建对象，取决于常量池中是否有。

new String("abc");至少创建一个，或者创建2个。

常量池，指的是在编译期确定，并被保存在已编译的字节码文件中的一些数据，它包括类、方法、接口等中的常量，存放字符串常量和基本类型常量（public static final）。

> [String a="abc"与String b=new String（"abc"）的区别](https://blog.csdn.net/yin_xing_ye/article/details/97046381)

<font color='orange'>Q：String类如何被加载的【类加载机制】</font>



<font color='orange'>Q：String转换成Integer的方式及原理</font>

1. Integer.parseInt(String s)--内部调用parseInt(s,10)（默认为10进制）
2. 正常判断null，进制范围，length等
3. 判断第一个字符是否是符号位
4. 循环遍历确定每个字符的十进制值
5. 通过*= 和-= 进行计算拼接
6. 判断是否为负值 返回结果。

> [java-string转换成integer的方式及原理](https://www.jianshu.com/p/9eebb4f2ccb1)

<font color='orange'>Q：abcde字符串在内存中占多大，这些字节在JVM的编码格式</font>



> [Java中的String到底占用多大的内存空间？带你一步步验证！](https://blog.csdn.net/XingXing_Java/article/details/109250697?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8.no_search_link&spm=1001.2101.3001.4242.8)

<font color='orange'>Q：String如何实现equals的怎么比较两个String。</font>

1、比较是否执行同一个对象

2、长度是否相等

3、遍历比较value[]中每个char是否相同。

```java
    public boolean equals(Object var1) {
        if (this == var1) {
            return true;
        } else {
            if (var1 instanceof String) {
                String var2 = (String)var1;
                int var3 = this.value.length;
                if (var3 == var2.value.length) {
                    char[] var4 = this.value;
                    char[] var5 = var2.value;

                    for(int var6 = 0; var3-- != 0; ++var6) {
                        if (var4[var6] != var5[var6]) {
                            return false;
                        }
                    }

                    return true;
                }
            }

            return false;
        }
    }
```



## 二、对比

<font color='orange'>Q、String、StringBuffer、StringBuilder的区别?</font>

> 从三方面回答：是否可变、效率、同步、适用范围。



<font color='orange'>Q：String、StringBuffer、StringBuilder在进行字符串操作时的效率（字节跳动）</font>

这里主要考察String在内存中是如何创建的。

见上文。

<font color='orange'>Q：StringBuilder是如何实现的？</font>





# 概述

关于String类的必知必会主要包括：

- String的常用函数
- equals()与==的区别
- String、StringBuffer 与 StringBuilder的区别
- Switch能否用string做参数？

------

# 1. String 常用函数

![img](https:////upload-images.jianshu.io/upload_images/944365-92f4917dae9295fe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 2. equals()与==的区别

![img](https:////upload-images.jianshu.io/upload_images/944365-eff8e7467c425887.png?imageMogr2/auto-orient/strip|imageView2/2/w/790/format/webp)

示意图

附：



```csharp
 /**
   * 附1：Object的equals（）原函数实现
   * 作用 = 比较的是对象的内存地址（内部实现实际 是 “==”，故作用同 “==”作用）
   */
  public boolean equals (Object obj）{
    return （this == obj）；
     }

 /**
   * 附2：复写了Object equals（）原函数的String 类中的equals
   * 作用：比较两个字符串的内容是否相同
   */
public boolean equals(Object obj){
  // 若2者指向同一个地址，那么它们的内容肯定相同
  // 使用 “==” 比较
  if (this == obj){
    return true;
   }

  // 若不指向同一地址，则判断规则为：
      // 1. 类型是否相同（ 即，传入对象是否是String类型，采用 instanceof 比较）
      // 2. 内容是否相同 = 字符串序列是否相同（String类 内部存储 采用char[]实现）
      if (anObject instanceof String) {

                  String anotherString = (String)anObject;
                   int n = value.length; // 注：比较次数 = 第1个String对象的长度n，而不是传入参数中的String对象长度
                   if (n == anotherString.value.length) {
                   char v1[] = value;
                   char v2[] = anotherString.value;

                   // 遍历过程中只要有1个字符不同，就返回false
                   int i = 0;
                    while (n-- != 0) {
                      if (v1[i] != v2[i])
                          return false;
                        i++;
                      }
                   return true;
                  }
              }
           return false;
        }
```

------

# 3. String、StringBuffer 与 StringBuilder的区别

3者 同样用于储存 & 操作字符串，区别如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-0dc3197cc9ae63d5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1160/format/webp)

示意图

------

# 4. Switch能否用string做参数？

- 在`Java7` 前，不支持；在Java 7后，支持

> `Java7` 前支持的类型：枚举、`byte`、`short`、`char`、`int` & 对应的封装类





# 参考

1、[https://blog.csdn.net/u011702479/article/details/82262823](https://blog.csdn.net/u011702479/article/details/82262823)
2、[https://blog.csdn.net/alankin/article/details/80406975](https://blog.csdn.net/alankin/article/details/80406975)

[Carson带你学Java：关于String类的必知必会](https://www.jianshu.com/p/f23fec94c5fb)
