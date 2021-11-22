> version：2021/9/12
>
> review：



<font color='orange'>Q1、Java中==和equals和hashCode的区别？</font>

首先我说一下它们分别是什么：

1）==。如果是基本数据类型，比较它们的值；如果是引用类型，比较它们指向对象的内存地址。对象存放在堆中，引用存放在栈中，==是对栈中的值进行比较，若返回true则代表变量的内存地址相等；

2）equals是Object类中的方法，默认实现是进行==比较，因此是比较对象的内存地址是否相等（是不是同一个对象）。这个方法可以被子类覆盖，一般覆盖后都是通过对象的内容来判断是否相等。比如一个Person类，可以判断它的身份证id是否相等。

3）hashCode()用于计算对象实例的哈希码。这个哈希值的作用是，当对象作为key值存入到Map等集合中的时候，实际存入的Key值是对象的哈希值。为什么这么做呢？因为在大量的对象比较中，hashCode要比equals快。

> 为什么hashCode比equals快？

下面我说一下它们的关系和区别：

4）equals与hashCode方法关系

hashCode()是一个本地方法，实现是根据本地机器有关。equals()相等的对象，hashCode()也一定相等；hashCode()不等，equals()一定也不等；hashCode()相等，equals()可能相等，也可能不等。

所以在重写equals(Object obj)方法的时候，也要重写hashCode()，确保equals()返回true的时候，两个对象hashCode()返回值也相等。这样才能正常使用Map集合的功能。

5）equals与==的关系

equals()只能比较引用类型，不能直接比较基本数据类型。==是Java已经规定好的，用来比较基本数据类型和引用类型。equals()可以进行重写，然后根据实际场景进行比较。

<font color='orange'>Q2、关于equals和hashcode的理解？？</font>

见Q1，2）、3）、4）。

<font color='orange'>Q3、为什么重写equals方法的时候也需要重写hashCode方法？？</font>

在使用Map等集合的时候，可以把对象作为Key值传入，实际使用的Key值是对象的hashCode()值。在进行add等操作时，会先比较hashCode返回值，如果hashCode和equals()不配对，那么Map集合就会丧失其高效率。

> 还需梳理。

<font color='orange'>Q、子类复写父类的equals方法,但是子类增加了一个成员变量int,请问equals方法怎么办？</font>



<font color='orange'>Q、？</font>

