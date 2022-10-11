> version：2021/10/20
>
> review：



目录

[TOC]

# 一、final

1、可以声明成员变量、方法、类以及本地变量

2、final 成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误

3、final 变量是只读的

4、final 申明的方法不可以被子类的方法重写

5、final 类通常功能是完整的，不能被继承

6、final 变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销

7、final 提高了性能，JVM 和 Java 应用都会缓存 final变量，会对方法、变量及类进行优化

8、方法的内部类要访问方法中的局部变量，必须用final修饰。



# 相关问题

<font color='orange'>Q：finally和return的顺序</font>

先finally后return。

<font color='orange'>Q：final关键字作用</font>

final用于声明属性，方法和类，分别表示属性不可交变，方法不可覆盖，类不可继承。

<font color='orange'>Q：final可以修饰什么内容，有什么区别？</font>

final用于声明属性，方法和类，分别表示属性不可交变，方法不可覆盖，类不可继承。

<font color='orange'>Q：final、finally、finalize区别</font>

final用于声明属性，方法和类，分别表示属性不可交变，方法不可覆盖，类不可继承。
finally是异常处理语句结构的一部分，表示总是执行。
finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，供垃圾收集时的其他资源回收，例如关闭文件等。

> [Java中final、finally、finalize的区别与用法](https://www.cnblogs.com/smart-hwt/p/8257330.html)

<font color='orange'>Q：final说一下。修饰在内部类的方法参数前有什么作用。</font>

为了保证数据一致性。

匿名内部类之所以可以访问局部变量，是因为在底层将这个局部变量的值传入到了匿名内部类中，并且以匿名内部类的成员变量的形式存在，这个值的传递过程是通过匿名内部类的构造器完成的。

匿名内部类在方法内时，匿名内部类对象生命周期可能超过方法内的局部变量的生命周期；为了延续生命周期Java复制了局部变量到匿名内部类，之后需要保证复制值与原始值始终一致；保证一致的方式是将局部变量声明为final使其不可变。

> [JDK8之前，匿名内部类访问的局部变量为什么必须要用final修饰](https://blog.csdn.net/tianjindong0804/article/details/81710268)
>
> [匿名内部类为什么访问外部类局部变量必须是final的？](https://blog.csdn.net/qq_41841298/article/details/79900826?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163473862916780269851647%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163473862916780269851647&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-79900826.first_rank_v2_pc_rank_v29&utm_term=http%3A%2F%2Fcuipengfei.me%2Fblog%2F2013%2F06%2F22%2Fwhy-does-it-have-to-be-final%2F&spm=1018.2226.3001.4187)
>
> [为什么Java匿名内部类访问的方法参数或方法局部变量需要被final修饰](https://www.cnblogs.com/z-sm/p/7058864.html)

<font color='orange'>Q：try、catch、finally，try里有return，finally还执行么？</font>

*Condition 1：*     如果try中**没有异常**且try中**有return**  （执行顺序）

```
try ---- finally --- return
```

*Condition 2：*   如果try中**有异常**并且try中**有return**

```
try----catch---finally--- return
```

总之 **finally 永远执行**！

*Condition 3：*   try中有异常，try-catch-finally里都没有return ，finally 之后有个return 

```
try----catch---finally
```

try中有异常以后，根据java的异常机制先执行catch后执行finally，此时错误异常已经抛出，程序因异常而终止，所以你的return是不会执行的

*Condition 4：*  当 try和finally中都有return时，finally中的return会覆盖掉其它位置的return（多个return会报unreachable code，编译不会通过）。

*Condition 5：* 当finally中不存在return，而catch中存在return，但finally中要修改catch中return 的变量值时

```
int ret = 0;
try{ 
	throw new Exception();
}
catch(Exception e)
{
	ret = 1;  return ret;
}
finally{
	ret = 2;
} 

```

最后返回值是1，因为return的值在执行finally之前已经确定下来了



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 参考

