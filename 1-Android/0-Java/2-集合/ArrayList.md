> 知识路径：Android > Java > 集合
>
> version：2021/4/6
>
> review：2021/4/6
>
> 掌握程度：了解



### 一、预备知识



### 二、ArrayList

![image-20210406111251709](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406111251709.png)

ArrayList.java

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
 
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    public E remove(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }    
    
    public E get(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        return (E) elementData[index];
    }
}
```





参考：

1、《Android核心知识点笔记V2020.03.30》