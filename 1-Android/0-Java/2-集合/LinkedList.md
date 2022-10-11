> 知识路径：Android > Java > 集合
>
> version：2021/4/6
>
> review：2021/4/6
>
> 掌握程度：了解



前言：

### 一、预备知识

数据结构：双链表

### 二、LinkedList

![image-20210406124731092](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406124731092.png)

1、继承结构

类图：

2、LinkedList.java

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }    
    // 增加元素
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
    // 删除元素
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }    
    // 查找元素
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }    
}
```



### 三、总结

1、LinkedList本质上是对双链表的封装。

2、

![image-20210406140408647](C:\Users\NJCS\AppData\Roaming\Typora\typora-user-images\image-20210406140408647.png)

### 四、思维导图

​	总结本文；把本文知识融入整体知识体系。

### 五、拓展

​	相关知识，横向、纵向对比；使用场景（示例）、适用范围、注意事项；

​	头脑风暴：联想相关知识，准备对比，为进一步深入学习做准备。



**参考：**

1、《Android核心知识点笔记V2020.03.30》