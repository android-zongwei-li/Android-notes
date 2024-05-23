# 背景

![img](https:////upload-images.jianshu.io/upload_images/944365-8fe90db664407e3b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

------

# 阅读本文可了解

1. Activity如何保存临时数据 & 状态
2. Activity如何恢复临时数据 & 状态

------

# Activity如何保存临时数据 & 状态

### 1. 核心方法

onSaveInstanceState()

### 2. 调用时机

当系统 **未经你许可** 时，**可能** 销毁了你的Activity，则会被系统调用 。

特别说明：

- “可能“ 仅表达一种可能性，而不是确实销毁，下面会继续讲解
- 若是 被用户**主动销毁**（如 用户按Back键），则不会调用
- 肯定在 **调用onStop()前**被调用，但不保证在onPause（）前 / 后

### 3. 具体调用场景

假定为Activity A显示在当前Activity栈的最上层时，以下情况会执行onSaveInstanceState()

注：系统不知道你切换到其他地方后要运行多少其他的程序，自然也不知Activity A是否会被销毁，故系统会调用onSaveInstanceState()，下面所说的所有情况该遵循这原则

![img](https:////upload-images.jianshu.io/upload_images/944365-9032e0caba5504cf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 4. 使用说明



```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {

// 通过Bundle参数以键值对的方式进行数据的存储
// 数据恢复：onRestoreInstanceState（） & onCreate（）
// 上述二者都有一个Bundle类型的参数用于恢复数据
        savedInstanceState.putBoolean("MyBoolean", true);
        savedInstanceState.putDouble("myDouble", 1.9);
        savedInstanceState.putInt("MyInt", 1);
        savedInstanceState.putString("MyString", "Welcome back to Android");
        // ...
        super.onSaveInstanceState(savedInstanceState);
}
```

补充说明：

- 布局每1个View默认实现：onSaveInstanceState()，即UI的任何改变都会自动的存储和在activity重新创建的时候自动的恢复（只有在为该UI提供了唯一ID后才起作用）
- 若需复写该方法从而存储额外的状态信息时，应先调用父类的onSaveInstanceState（）（因为默认的onSaveInstanceState（）帮助UI存储它的状态）
- 只使用该方法记录Activity的瞬间状态（UI的状态），而不是去存储持久化数据，因为onSaveInstanceState（）调用时机不确定性；可使用 onPause（）存储 持久化数据

------

# Activity如何恢复临时数据 & 状态

### 1. 核心方法

onRestoreInstanceState（）

### 2. 调用时机

当系统“未经你许可”时，确实销毁了你的Activity，则重新启动时会被系统调用

特别说明：

- 与onSaveInstanceState（）区别：此处是 **“确实销毁”后才调用**
- 若是 **被用户主动销毁（如 用户按Back键）**，则不会调用
- 肯定在调用 **onStop()前被调用**，但不保证在onPause（）前 / 后

### 3. 具体调用场景

若 异常关闭了Activity，即调用了onSaveInstanceState（） & 下次启动时会调用onRestoreInstanceState（）

注：此时结合Activity的生命周期的调用顺序是：

1. onCreate（）
2. onStart（）
3. onRestoreInstanceState（）
4. onResume（）

### 4. 使用示例



```java
@Override
public void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);

        boolean myBoolean = savedInstanceState.getBoolean("MyBoolean");
        double myDouble = savedInstanceState.getDouble("myDouble");
        int myInt = savedInstanceState.getInt("MyInt");
        String myString = savedInstanceState.getString("MyString");
}
```

- onSaveInstanceState（）、onRestoreInstanceState（）不一定 成对被调用

如：当正在显示Activity A时，用户按下HOME键回到主界面，然后用户紧接着又返回到Activity A，此时Activity A一般不会因为内存的原因被系统销毁，故Activity A的onRestoreInstanceState（）不会被执行

- onSaveInstanceState的bundle参数会传递到onCreate方法中，可选择在onCreate（）中做数据还原

**至此，关于Activity的临时数据 & 状态 保存 & 恢复，讲解完毕。**



# 参考

[Carson带你学Android：该如何保存 & 恢复Activity状态缓存-onSaveInstanceState()、onRestoreInstanceState()](https://www.jianshu.com/p/ad00c64cf30d)