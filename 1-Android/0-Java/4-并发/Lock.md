> version：2021/4/7
>
> review：2021/4/7

### 二、Lock

```java
public interface Lock {
    void lock();

    void lockInterruptibly() throws InterruptedException;

    boolean tryLock();

    boolean tryLock(long var1, TimeUnit var3) throws InterruptedException;

    void unlock();

    Condition newCondition();
}
```

- lock()

  ![image-20240718183715829](images/Lock/image-20240718183715829.png)

  ![image-20240718183728353](images/Lock/image-20240718183728353.png)

- lockInterruptibly()

  

- tryLock()

  

- tryLock(long var1, TimeUnit var3)

  

#### 2.1 锁的分类

![image-20240718183746173](images/Lock/image-20240718183746173.png)

#### 2.2 悲观锁、乐观锁

![image-20240718183809947](images/Lock/image-20240718183809947.png)

#### 2.3 自旋锁、适应性自旋锁

![image-20240718183819893](images/Lock/image-20240718183819893.png)

#### 2.4 死锁

![image-20240718183826411](images/Lock/image-20240718183826411.png)



# 参考

《Android核心知识点笔记V2020.03.30》