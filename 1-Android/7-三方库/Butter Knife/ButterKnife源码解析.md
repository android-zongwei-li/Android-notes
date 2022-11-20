目录

1、什么是**编译时技术**？

2、编译时技术的两大核心要素

3、ButterKnife的核心思想剖析

4、运用编译时技术手写ButterKnife实战



在使用ButterKnife的时候，都需要写这样一行代码：



选择来分析一下它的源码：

```java
  public static Unbinder bind(@NonNull Object target, @NonNull Dialog source) {
    return bind(target, source, Finder.DIALOG);
  }

  static Unbinder bind(@NonNull Object target, @NonNull Object source, @NonNull Finder finder) {
    Class<?> targetClass = target.getClass();
    try {
      if (debug) Log.d(TAG, "Looking up view binder for " + targetClass.getName());
      ViewBinder<Object> viewBinder = findViewBinderForClass(targetClass);
      return viewBinder.bind(finder, target, source);
    } catch (Exception e) {
      throw new RuntimeException("Unable to bind views for " + targetClass.getName(), e);
    }
  }
```

1、Class<?> 是什么意思？（尝试：采用问答的形式做笔记/学习。）

泛型、通配符





@Retention

```kotlin
这个元注解表示他所修饰的注解的生命周期，即需要在什么级别保存该注解信息，
Retention有一个属性value，是RetentionPolicy类型的，RetentionPolicy是一个枚举
类型，这个枚举决定了Retention注解应该如何去保持。RetentionPolicy有以下三个值      
**RetentionPolicy.SOURCE**：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；
**RetentionPolicy.CLASS**：注解被保留到class文件，但jvm加载class文件的时候被遗弃，这是默认的生命周期；
**RetentionPolicy.RUNTIME**：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在；
```





ButterKnife实战

1、创建两个module（Java library）：annotation、

1. annotation module 用来定义注解
2. annotation_compiler module 用来定义注解处理器

这里再做一件事：给annotation_compiler module添加对annotation module的依赖。毕竟要在处理器模块对注解做处理。

2、定义注解

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.CLASS)
public @interface BindView {
    int value();
}
```

3、定义注解处理器

Step 1：创建处理器，继承自AbstractProcessor

```java
public class AnnotationCompiler extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        return false;
    }
}
```

Step 2：注册服务

处理器想要生效，还必须先注册一下，操作如下：

1. 新建文件夹resources/META-INF.services
2. 新建文件javax.annotation.processing.Processor，文件内容：com.lizw.annotation_compiler.AnnotationCompiler（包名+注解处理器名）

Step 3：实现注解处理器

首先思考一个问题：这个处理器要做什么？

这个处理器的最终目的是：根据注解信息生成一个Java文件来绑定View。下面来具体实现：

3.1 继承AbstractProcessor，实现process

```java
public class AnnotationCompiler extends AbstractProcessor {
    // 生成文件的对象
    Filer mFilter;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        mFilter = processingEnvironment.getFiler();
    }

    /**
     * 声明要处理哪些注解
     *
     * @return
     */
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        HashSet<String> types = new HashSet<>();
        types.add(BindView.class.getCanonicalName());
        types.add(OnClick.class.getCanonicalName());
        return types;
    }

    /**
     * 声明支持的Java版本
     *
     * @return
     */
    @Override
    public SourceVersion getSupportedSourceVersion() {
        //最新版Java
        return processingEnv.getSourceVersion();
    }

    /**
     * 此方法可以获取到注解标记的内容
     *
     * @param set
     * @param roundEnvironment
     * @return
     */
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        System.out.println("process");
        // 获取当前模块所有用到BindView的节点（Element）
        // 类节点 TypeElement
        // 方法节点 ExecutableElement
        // 成员变量节点 VariableElement
        Set<? extends Element> elementsAnnotated = roundEnvironment.getElementsAnnotatedWith(BindView.class);
        // 找到各个类中被BindView注解的field，按类名储存到variableElementMap中
        Map<String, List<VariableElement>> variableElementMap = new HashMap<>();
        for (Element element : elementsAnnotated) {
            VariableElement variableElement = (VariableElement) element;
            // 获取类名
            TypeElement typeElement = (TypeElement) variableElement.getEnclosingElement();
            String activityName = typeElement.getSimpleName().toString();
            // 根据类名，获取当前类被注解的field集合
            List<VariableElement> variableElements = variableElementMap.get(activityName);
            if (variableElements == null) {
                variableElements = new ArrayList<>();
                variableElementMap.put(activityName, variableElements);
            }
            variableElements.add(variableElement);
        }
         // 根据注解生成对应的Java文件，用于绑定view
        // 要生成需要的Java文件，我们需要：包名、类名、新文件名（新文件就是要生成出来用于处理findViewById的类）
        if (variableElementMap.size() > 0) {
            Writer writer = null;
            Iterator<String> iterator = variableElementMap.keySet().iterator();
            while (iterator.hasNext()) {
                // 获取 类名、新文件名、包名
                String activityName = iterator.next();
                String newName = activityName + "$$ViewBinder";
                List<VariableElement> variableElements = variableElementMap.get(activityName);
                String packageName = getPackageName(variableElements.get(0));
                // 生成文件
                try {
                    JavaFileObject sourceFile = mFilter.createSourceFile(packageName + "." + newName);
                    writer = sourceFile.openWriter();
                    StringBuilder sb = new StringBuilder();
                    sb.append("package ").append(packageName).append(";\n");
                    sb.append("import android.view.View;\n");
                    sb.append("public class ").append(newName).append("{\n");
                    sb.append("    public ").append(newName).append("(final ").append(packageName).append(".").append(activityName).append(" target){\n");
                    for (VariableElement variableElement : variableElements) {
                        // 成员变量名
                        String fieldName = variableElement.getSimpleName().toString();
                        // 注解value
                        int resId = variableElement.getAnnotation(BindView.class).value();
                        sb.append("        target." + fieldName + " = target.findViewById(" + resId + ");\n");
                    }
                    sb.append("    }\n}");
                    writer.write(sb.toString());
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    if (writer != null) {
                        try {
                            writer.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
        return false;
    }

    private String getPackageName(VariableElement variableElement) {
        Element typeElement = variableElement.getEnclosingElement();
        PackageElement packageOf = processingEnv.getElementUtils().getPackageOf(typeElement);
        Name qualifiedName = packageOf.getQualifiedName();
        return qualifiedName.toString();
    }
}
```

这一步完成后，可以通过make project生成绑定view的Java文件。

Step 4：提供一个调用接口

新建一个ButterKnife类，用于传入使用了注解的类，并在其中调用绑定类的绑定代码。

```java
public class ButterKnife {
    public static void bind(Object activity) {
        Class<?> activityClass = activity.getClass();
        try {
            // getName 带包名
            Class<?> binder = Class.forName(activityClass.getName() + "$$ViewBinder");
            Constructor constructor = binder.getConstructor(activity.getClass());
            constructor.newInstance(activity);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

Step 5：使用

```java
public class MainActivity extends AppCompatActivity {
    @BindView(R.id.tv_main_act)
    TextView tvMainAct;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        tvMainAct.setText("success");
    }
}
```

到此，一个极简的ButterKnife就完成了。



总结：

1、



2、注解涉及到的类/接口：

class

​	Filer

abstract class

​	AbstractProcessor

interface

​	RoundEnviroment

​	ProcessingEnviroment

enum

​	SourceVersion



方法总结：

> 学习Android/Java的过程就是不断认识新对象的过程。提高就是学习如何组织对象的过程。