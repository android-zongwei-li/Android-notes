<font color='orange'>Q1、进程和线程的区别？</font>

一般来说，一个进程对应着一个应用程序。它里面可以包括多个线程。

多线程的优点可以提高程序的执行效率，提高资源利用率。

> 概念

<font color='orange'>Q2、wait和 sleep 的区别？</font>

wait()是Object类的方法，wait是对象锁，锁定方法不让继续执行，当执行notify()方法后就会继续执行；

sleep() 是Thread类的方法，sleep 使线程睡眠，让出cpu，结束后（时间到）自动继续执行。

> 方法的归属、方法的作用、与之相关的其他方法