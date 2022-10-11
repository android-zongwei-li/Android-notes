> version：2021/10/24
>
> review：



目录

[TOC]



- static 关键字修饰的方法或者变量可以直接通过类名访问。
- 不能直接使用对象实例访问静态方法和静态变量。见例1。
- 静态变量被所有的对象所共享，在内存中仅有一个副本，当且仅当类初次加载时会被初始化。
- 能通过this访问静态变量吗？所有的静态方法和静态变量都可以通过对象访问。（只要权限足够）
- static不能用来修饰局部变量。



例1：

```java
public class A {
    public static int x = 0;

    public void m2() {
        x = 3;
        m1();
    }

    public static void m1() {
        System.out.println("sss");
    }
}
```

不能通过对象直接访问（调用） x 和 m1 。



# 相关问题

<font color='orange'>Q：static编译时有啥不同,static语句块,static变量,static方法,构造初始化顺序(静态绑定)</font>

静态变量，static代码块，static方法，然后初始化非静态的。

> 这个问题很多细节还没有掌握。

<font color='orange'>Q：static方法可以被覆盖吗？为什么？</font>

不可以。static方法是属于类的，从概念上讲，覆盖是为了在多态中进行动态的调用，属于动态绑定，但是static在静态绑定时就与所属的类关联起来了，父类引用调用static方法，即使对象实际是子类的，子类也有相同方法，但是实际调用的也是父类中的，不存在多态的情况。

<font color='orange'>Q：private和static修饰的方法可以重写吗；</font>

都不能重写。

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、



# 参考

