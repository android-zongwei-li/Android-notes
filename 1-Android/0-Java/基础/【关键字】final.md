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



# 前言

- 在`Java`中，不同情形下`return` 和 `finally`的执行顺序很多人混淆不清
- 本文全面 & 详细解析不同情形下`return` 和 `finally`的执行顺序，希望你们会喜欢

------

# 目录

- 储备知识
- 终极结论
- 场景分析
- 总结
- 额外补充：final、finally和finallize的区别

------

# 1. 储备知识

- `try / catch`是常见的捕捉异常 & 处理的语句
- 只有`try`语句中抛出异常，才会执行`catch`中的语句



```php
/**
  * try中无抛出异常，则catch中的代码不执行 
  */
    try{ 
        // 代码无抛出异常 
        return  result; 

      }catch(Exception e){ 
        // catch代码 
     }

 /**
   * try中抛出异常，则执行catch中的语句
   */
     try{ 
        //代码抛出异常 
        throw Exception; 
        return1  result1; 

      } catch(Exception e){ 
       return2  result2; // 执行catch中的语句
     }
```

------

# 2. 终极结论

无论什么情况（异常与否、`try / catch` 前面存在`return`），`finally`块代码一定会执行

> 必须谨记！！

------

# 3. 具体场景分析

下面，我将根据具体的使用场景来全面解析不同情形下`return` 和 `finally`的执行顺序

### 3.1  try 或 catch中存在return语句、finally无return语句

- 执行顺序 结论
   `return`后的语句-> `finally`语句 -> `return`结束函数 & 返回值

> `finally`语句不影响最终返回值，即返回值在`finally`前就决定

- 详细讲解
   此处细分为2种情况：
   a. `try`中有`return`、无抛出异常
   b. `try`中有`return`、抛出异常 、`catch`有`return`



```dart
/**
  * 情况1：try中有return、无抛出异常 
  * 实际执行顺序：
  *            1. 执行 try块语句
  *            2. 执行 return后 的语句：得到结果result & 保存下来
  *            3. 执行 finally块语句：不影响上述保存的返回值，哪怕修改了变量的值
  *            4. 执行 return，结束函数，返回result的值：依旧返回步骤2保存的结果
  */
    try{ 
      //代码无抛出异常 
      return  result; 

        }catch(Exception e){ 
     
         }finally{ 
          // finally代码 
     }

/**
  * 情况2：try中有return、抛出异常 、catch有return
  * 实际执行顺序：
  *            1. 执行 try块语句
  *            2. 执行 throw 语句 ：此时已抛出异常，运行因异常而终止，故不执行return1
  *            3. 执行 catch块语句 
  *            4. 执行 return2后 的语句：得到结果result2 & 保存下来
  *          5. 执行 finally块语句：不影响上述保存的返回值，哪怕修改了变量的值
  *          6. 执行 return2，结束函数，返回result2的值：依旧返回步骤4保存的结果   
  */
    try{ 
      //代码抛出异常 
      throw Exception; 
      return1  result1; 

       }catch(Exception e){ 
        return2  result2; 

       }finally{ 
         // finally代码 
      }
```

### 3.2 finally中存在return语句（无论 try 或 catch之一 或 都存在return语句 ）

- 执行顺序 结论
   当执行到`finally`语句的 `return`时，程序就直接返回

> ```
> finally`中的`return`会覆盖掉其它位置的`return
> ```

- 详细讲解
   此处细分为2种情况：
   a. `try` & `catch`中都无`return`、无抛出异常  & `finally`中 有 `return`
   b. `try` / `catch`中任意1者 或 都有`return`（`try`中的`return`和`catch`中的`return`最多只有1个会执行）、`finally`中 有 `return`



```python
/**
  * 情况1：try & catch中都无return、无抛出异常 & finally中 有 return
  * 实际执行顺序：
  *            1. 执行 try块语句
  *            2. 执行 finally块语句：会影响返回值
  *            3. 执行 return，结束函数，返回result的值
  */
    try{ 

      }catch(Exception e){ 

      }finally{ 
         return result ;
     }

/**
  * 情况2：try / catch中任意1者 或 都有return（try中的return和catch中的return最多只有1个会执行）、finally中 有 return
  * 实际执行顺序：
  *            1. 执行 try块语句：设无抛出异常，则不执行catch语句 & return2
  *            2. 执行 return1 后 的语句：得到结果result & 保存下来
  *            3. 执行 finally块语句：不影响上述保存的返回值，哪怕修改了变量的值
  *            4. 执行finally内的 return3 后语句：finally中的return会覆盖掉其它位置的return
  *            5. 执行return3 ，结束函数，返回result3的值
  */
    try{ 
      //throw Exception; 
      return1 result1; 

        }catch(){ 
             return2 result2;
         
        }finally{ 
             return3 result3; 
        }
```

### 特别注意

`finally`中的语句最好：

1. 不要包含`return`语句，否则程序会提前退出
2. 返回值 ≠ `try` 或 `catch`中保存的返回值

至此，关于不同情形下`return` 和 `finally`的执行顺序 情况讲解完毕。

------

# 4. 总结

本文主要讲解了不同情形下`return` 和 `finally`的执行顺序，总结如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-93ea35900afafa71.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

示意图

------

# 5. 额外补充：final、finally和finallize的区别

![img](https:////upload-images.jianshu.io/upload_images/944365-5d52bf1396f75fa4.png?imageMogr2/auto-orient/strip|imageView2/2/w/963/format/webp)



# 参考

[Carson带你学Java：一文带你了解多种情形下return 和 finally的执行顺序](https://www.jianshu.com/p/f87ce04b429e)
