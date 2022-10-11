> 知识路径：Android > Java > 集合
>
> version：2021/4/6
>
> review：2021/4/6
>
> 掌握程度：了解



前言：（可选）

### 一、预备知识

volatile、synchronized、数组

### 二、CopyOnWriteArrayList

![image-20210406141630517](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406141630517.png)

1、继承结构

类图

2、CopyOnWriteArrayList.java

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
 
	final transient Object lock = new Object();    
    
    // The array, accessed only via getArray/setArray.
    private transient volatile Object[] array;
    
    final Object[] getArray() {
        return array;
    }
    
    final void setArray(Object[] a) {
        array = a;
    }    
    
    public boolean add(E e) {
        synchronized (lock) {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        }
    }
    
    public E remove(int index) {
        synchronized (lock) {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        }
    }   
    
    private E get(Object[] a, int index) {
        return (E) a[index];
    }    
}
```



### 三、总结

CopyOnWriteArrayList是线程安全容器。其内部持有的array数组使用volatile修饰保证了线程见的可见性；又通过synchronized对add()、remove()方法进行加锁，保证了线程安全、数据一致性。

本质上也是对数组操作的封装。

对数组的复制涉及到了两个重要方法：Arrays.copyOf、System.copyarray；

### 四、思维导图

​	总结本文；把本文知识融入整体知识体系。

### 五、拓展

​	相关知识，横向、纵向对比；使用场景（示例）、适用范围、注意事项；

​	头脑风暴：联想相关知识，准备对比，为进一步深入学习做准备。



**参考：**

1、《Android核心知识点笔记V2020.03.30》