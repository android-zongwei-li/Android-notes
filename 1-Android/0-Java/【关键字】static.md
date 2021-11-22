> version：2021/10/24
>
> review：



目录

[TOC]





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

## 【精益求精】我还能做（补充）些什么？

1、



# 参考

