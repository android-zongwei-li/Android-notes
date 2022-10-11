> 知识路径：Android > 四大组件 > Activity
>
> version：2021/4/14
>
> review：2021/4/14
>
> 掌握程度：初学



前言：（可选）

## 一、预备知识

可选

## 二、**Activity 组件之间的通信**

### 2.1 Activity->Activity

[1]Intent/Bundle

这种方式多用于 Activity 之间传递数据。示例代码如下：

\1. 

//首先创建一个 Bundle 对象

\2. 

Bundle bundle = **new** Bundle();

\3. 

bundle.putString("data_string","数据");

\4. 

bundle.putInt("data_int",10);

\5. 

bundle.putChar("da_char",'a');

6.

\7. 

//然后创建一个 Intent 对象

\8. 

Intent intent = **new** Intent(FirstActivity.**this**,SecondActivity.**class**);

\9. 

intent.putExtras(bundle);

\10. startActivity(intent);

[2]类静态变量

在 Activity 内部定义静态的变量，这种方式见于少量的数据通信，如果数据过多，还是使

用第一种方式。示例代码如下：

\1. 

**public class** FirstActivity extends AppCompatActivity {

2.

\3. 

//声明为静态

\4. 

**static** boolean isFlag = **false**;

5.

\6. 

@Override

\7. 

**protected void** onCreate(Bundle savedInstanceState) {

\8. 

super.onCreate(savedInstanceState);

\9. 

setContentView(R.layout.activity_first);

10.

\11. //首先创建一个 Bundle 对象

\12. Bundle bundle = **new** Bundle();13. bundle.putString("data_string","数据");

\14. bundle.putInt("data_int",10);

\15. bundle.putChar("da_char",'a');

16.

\17. //然后创建一个 Intent 对象

\18. Intent intent = **new** Intent(FirstActivity.**this**,SecondActivity.**class**);

\19. intent.putExtras(bundle);

\20. startActivity(intent);

21.

22.

\23. }

24.

\25. }

[3]全局变量

创建一个类，里面定义一批静态变量，Activity 之间通信都可以访问这个类里面的静态变

量，这就是全局变量。这种方式笔者就不给代码了。

### 2.2 

使用

原理

## 三、总结

本文总结：核心知识点

## 四、拓展

相关知识，横向、纵向对比；使用场景（示例）、适用范围、注意事项；

头脑风暴：联想相关知识，准备对比，为进一步深入学习做准备。

## 五、思维导图

总结本文；把本文知识融入整体知识体系。



**参考：**

1、《Activity篇.pdf》