> version：2022/11/15
>
> review：



目录

[TOC]

# 笔记简介





# 关键词 & 概念





# 一、简介



# 二、创建（被观察者）操作符

## 1、create()

创建一个被观察者

```java
public static <T> Observable<T> create(ObservableOnSubscribe<T> source)
```

```java
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> e) throws Exception {
        e.onNext("Hello Observer");
        e.onComplete();
    }
});
```

创建 ObservableOnSubscribe 并重写其 subscribe 方法，就可以通过 ObservableEmitter 发射器向观察者发送事件。

下面创建一个观察者，来验证这个被观察者是否成功创建。

```java
Observer<String> observer = new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(String s) {
        Log.d("chan","=============onNext " + s);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {
        Log.d("chan","=============onComplete ");
    }
};
        
observable.subscribe(observer);
```

```
05-20 16:16:50.654 22935-22935/com.example.louder.rxjavademo D/chan: =============onNext Hello Observer
=============onComplete
```

## 2、just()

创建一个被观察者，并发送事件，发送的事件不可以超过10个以上。

```111
public static <T> Observable<T> just(T item) 
......
public static <T> Observable<T> just(T item1, T item2, T item3, T item4, T item5, T item6, T item7, T item8, T item9, T item10)
```

```typescript
Observable.just(1, 2, 3)
.subscribe(new Observer<Integer> () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "=================onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "=================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "=================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "=================onComplete ");
    }
});
```

```ini
05-20 16:27:26.938 23281-23281/? D/chan: =================onSubscribe
=================onNext 1
=================onNext 2
=================onNext 3
=================onComplete 
```

## 3、From 操作符

### 3.1 fromArray()

```java
public static <T> Observable<T> fromArray(T... items)
```

这个方法和 just() 类似，只不过 fromArray 可以传入多于10个的变量，并且可以传入一个数组。

```typescript
Integer array[] = {1, 2, 3, 4};
Observable.fromArray(array)
.subscribe(new Observer<Integer> () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "=================onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "=================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "=================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "=================onComplete ");
    }
});
```

```ini
05-20 16:35:23.797 23574-23574/com.example.louder.rxjavademo D/chan: =================onSubscribe
=================onNext 1
=================onNext 2
=================onNext 3
=================onNext 4
=================onComplete 
```

### 3.2 fromCallable()

```swift
public static <T> Observable<T> fromCallable(Callable<? extends T> supplier)
```

这里的 Callable 是 java.util.concurrent 中的 Callable，Callable 和 Runnable 的用法基本一致，只是它会返回一个结果值，这个结果值就是发给观察者的。

```java
Observable.fromCallable(new Callable < Integer > () {
    @Override
    public Integer call() throws Exception {
        return 1;
    }
})
.subscribe(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "================accept " + integer);
    }
});
```

```yaml
05-26 13:01:43.009 6890-6890/? D/chan: ================accept 1
```

### 3.3 fromFuture()

```swift
public static <T> Observable<T> fromFuture(Future<? extends T> future)
```

参数中的 Future 是 java.util.concurrent 中的 Future，Future 的作用是增加了 cancel() 等方法操作 Callable，它可以通过 get() 方法来获取 Callable 返回的值。

```java
FutureTask < String > futureTask = new FutureTask < > (new Callable < String > () {
    @Override
    public String call() throws Exception {
        Log.d(TAG, "CallableDemo is Running");
        return "返回结果";
    }
});

Observable.fromFuture(futureTask)
    .doOnSubscribe(new Consumer < Disposable > () {
    @Override
    public void accept(Disposable disposable) throws Exception {
        futureTask.run();
    }
})
.subscribe(new Consumer < String > () {
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG, "================accept " + s);
    }
});
```

doOnSubscribe() 的作用就是只有订阅时才会发送事件，具体会在下面讲解。

```bash
05-26 13:54:00.470 14429-14429/com.example.rxjavademo D/chan: CallableDemo is Running
================accept 返回结果
```

### 3.4 fromIterable()

```swift
public static <T> Observable<T> fromIterable(Iterable<? extends T> source)
```

发送一个 List 集合数据给观察者

```typescript
List<Integer> list = new ArrayList<>();
list.add(0);
list.add(1);
list.add(2);
list.add(3);
Observable.fromIterable(list)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "=================onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "=================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "=================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "=================onComplete ");
    }
});
```

```ini
05-20 16:43:28.874 23965-23965/? D/chan: =================onSubscribe
=================onNext 0
=================onNext 1
=================onNext 2
=================onNext 3
=================onComplete
```

## 4 defer()

```swift
public static <T> Observable<T> defer(Callable<? extends ObservableSource<? extends T>> supplier)
```

直到被观察者被订阅后才会创建被观察者。

```less
// i 要定义为成员变量
Integer i = 100;
        
Observable<Integer> observable = Observable.defer(new Callable<ObservableSource<? extends Integer>>() {
    @Override
    public ObservableSource<? extends Integer> call() throws Exception {
        return Observable.just(i);
    }
});

i = 200;

Observer observer = new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
};

observable.subscribe(observer);

i = 300;

observable.subscribe(observer);
```

```diff
05-20 20:05:01.443 26622-26622/? D/chan: ================onNext 200
================onNext 300
```

defer() 只有观察者订阅的时候才会创建新的被观察者，所以每订阅一次就会打印一次，并且都是打印 i 最新的值。

## 5 timer()

```arduino
public static Observable<Long> timer(long delay, TimeUnit unit) 
```

当到指定时间后就会发送一个 0L 的值给观察者。

```less
Observable.timer(2, TimeUnit.SECONDS)
.subscribe(new Observer < Long > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Long aLong) {
        Log.d(TAG, "===============onNext " + aLong);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```bash
05-20 20:27:48.004 27204-27259/com.example.louder.rxjavademo D/chan: ===============onNext 0
```

## 6 interval()

```java
public static Observable<Long> interval(long period, TimeUnit unit)
public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit)
```

每隔一段时间就会发送一个事件，这个事件是从0开始，不断增1的数字。

```typescript
Observable.interval(4, TimeUnit.SECONDS)
.subscribe(new Observer < Long > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==============onSubscribe ");
    }

    @Override
    public void onNext(Long aLong) {
        Log.d(TAG, "==============onNext " + aLong);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```bash
05-20 20:48:10.321 28723-28723/com.example.louder.rxjavademo D/chan: ==============onSubscribe 
05-20 20:48:14.324 28723-28746/com.example.louder.rxjavademo D/chan: ==============onNext 0
05-20 20:48:18.324 28723-28746/com.example.louder.rxjavademo D/chan: ==============onNext 1
05-20 20:48:22.323 28723-28746/com.example.louder.rxjavademo D/chan: ==============onNext 2
05-20 20:48:26.323 28723-28746/com.example.louder.rxjavademo D/chan: ==============onNext 3
05-20 20:48:30.323 28723-28746/com.example.louder.rxjavademo D/chan: ==============onNext 4
05-20 20:48:34.323 28723-28746/com.example.louder.rxjavademo D/chan: ==============onNext 5
```

从时间就可以看出每隔4秒就会发出一次数字递增1的事件。

interval() 第三个方法的 initialDelay 参数，这个参数的意思就是 onSubscribe 回调之后，再次回调 onNext 的间隔时间。

## 7 intervalRange()

```java
public static Observable<Long> intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit)
public static Observable<Long> intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit, Scheduler scheduler)
```

指定发送事件的开始值和数量，其他与 interval() 的功能一样。

```typescript
Observable.intervalRange(2, 5, 2, 1, TimeUnit.SECONDS)
.subscribe(new Observer < Long > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==============onSubscribe ");
    }

    @Override
    public void onNext(Long aLong) {
        Log.d(TAG, "==============onNext " + aLong);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```bash
05-21 00:03:01.672 2504-2504/com.example.louder.rxjavademo D/chan: ==============onSubscribe 
05-21 00:03:03.674 2504-2537/com.example.louder.rxjavademo D/chan: ==============onNext 2
05-21 00:03:04.674 2504-2537/com.example.louder.rxjavademo D/chan: ==============onNext 3
05-21 00:03:05.674 2504-2537/com.example.louder.rxjavademo D/chan: ==============onNext 4
05-21 00:03:06.673 2504-2537/com.example.louder.rxjavademo D/chan: ==============onNext 5
05-21 00:03:07.674 2504-2537/com.example.louder.rxjavademo D/chan: ==============onNext 6
```

可以看出收到5次 onNext 事件，并且是从 2 开始的。

## 8 range()

```sql
public static Observable<Integer> range(final int start, final int count)
```

同时发送一定范围的事件序列。

```typescript
Observable.range(2, 5)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==============onSubscribe ");
    }

    @Override
    public void onNext(Integer aLong) {
        Log.d(TAG, "==============onNext " + aLong);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```ini
05-21 00:09:17.202 2921-2921/? D/chan: ==============onSubscribe 
==============onNext 2
==============onNext 3
==============onNext 4
==============onNext 5
==============onNext 6
```

## 9 rangeLong()

```java
public static Observable<Long> rangeLong(long start, long count)
```

作用与 range() 一样，只是数据类型为 Long

## 10 empty() & never() & error()

```java
public static <T> Observable<T> empty()
public static <T> Observable<T> never()
public static <T> Observable<T> error(final Throwable exception)
```

1. empty() ： 直接发送 onComplete() 事件
2. never()：不发送任何事件
3. error()：发送 onError() 事件

```typescript
Observable.empty()
.subscribe(new Observer < Object > () {

    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe");
    }

    @Override
    public void onNext(Object o) {
        Log.d(TAG, "==================onNext");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError " + e);
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete");
    }
});
```

```bash
05-26 14:06:11.881 15798-15798/com.example.rxjavademo D/chan: ==================onSubscribe
==================onComplete
```

换成 never() 的打印结果：

```bash
05-26 14:12:17.554 16805-16805/com.example.rxjavademo D/chan: ==================onSubscribe
```

换成 error() 的打印结果：

```bash
05-26 14:12:58.483 17817-17817/com.example.rxjavademo D/chan: ==================onSubscribe
==================onError java.lang.NullPointerException
```



# 三、转换操作符

## 1、map()

```swift
public final <R> Observable<R> map(Function<? super T, ? extends R> mapper)
```

将被观察者发送的数据类型转变成其他的类型

以下代码将 Integer 类型的数据转换成 String。

```typescript
Observable.just(1, 2, 3)
.map(new Function < Integer, String > () {
    @Override
    public String apply(Integer integer) throws Exception {
        return "I'm " + integer;
    }
})
.subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.e(TAG, "===================onSubscribe");
    }

    @Override
    public void onNext(String s) {
        Log.e(TAG, "===================onNext " + s);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```ini
05-21 09:16:03.490 5700-5700/com.example.rxjavademo E/chan: ===================onSubscribe
===================onNext I'm 1
===================onNext I'm 2
===================onNext I'm 3
```

## 2、flatMap()

```swift
public final <R> Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper)
```

将事件序列中的元素进行整合加工，返回一个新的被观察者。

flatMap() 与 map() 类似，但是 flatMap() 返回的是一个 Observerable。现在用一个例子来说明 flatMap() 的用法。

假设有一个 Person 类，这个类的定义如下：

```typescript
public class Person {

    private String name;
    private List<Plan> planList = new ArrayList<>();

    public Person(String name, List<Plan> planList) {
        this.name = name;
        this.planList = planList;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Plan> getPlanList() {
        return planList;
    }

    public void setPlanList(List<Plan> planList) {
        this.planList = planList;
    }

}
```

Person 类有一个 name 和 planList 两个变量，分别代表的是人名和计划清单。

Plan 类的定义如下：

```typescript
public class Plan {

    private String time;
    private String content;
    private List<String> actionList = new ArrayList<>();

    public Plan(String time, String content) {
        this.time = time;
        this.content = content;
    }

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public List<String> getActionList() {
        return actionList;
    }

    public void setActionList(List<String> actionList) {
        this.actionList = actionList;
    }
}
```

现在有一个需求就是要将 Person 集合中的每个元素中的 Plan 的 action 打印出来。 首先用 map() 来实现这个需求看看：

```less
Observable.fromIterable(personList)
.map(new Function < Person, List < Plan >> () {
    @Override
    public List < Plan > apply(Person person) throws Exception {
        return person.getPlanList();
    }
})
.subscribe(new Observer < List < Plan >> () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(List < Plan > plans) {
        for (Plan plan: plans) {
            List < String > planActionList = plan.getActionList();
            for (String action: planActionList) {
                Log.d(TAG, "==================action " + action);
            }
        }
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

可以看到 onNext() 用了嵌套 for 循环来实现，如果代码逻辑复杂起来的话，可能需要多重循环才可以实现。

现在看下使用 flatMap() 实现：

```typescript
Observable.fromIterable(personList)
.flatMap(new Function < Person, ObservableSource < Plan >> () {
    @Override
    public ObservableSource < Plan > apply(Person person) {
        return Observable.fromIterable(person.getPlanList());
    }
})
.flatMap(new Function < Plan, ObservableSource < String >> () {
    @Override
    public ObservableSource < String > apply(Plan plan) throws Exception {
        return Observable.fromIterable(plan.getActionList());
    }
})
.subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "==================action: " + s);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

从代码可以看出，只需要两个 flatMap() 就可以完成需求，并且代码逻辑非常清晰。

## 3、concatMap()

```swift
public final <R> Observable<R> concatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper)
public final <R> Observable<R> concatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper, int prefetch)
```

concatMap() 和 flatMap() 基本上是一样的，只不过 concatMap() 转发出来的事件是有序的，而 flatMap() 是无序的。

使用上面 flatMap() 的例子，首先来试下 flatMap() 验证发送的事件是否是无序的，代码如下：

```typescript
Observable.fromIterable(personList)
.flatMap(new Function < Person, ObservableSource < Plan >> () {
    @Override
    public ObservableSource < Plan > apply(Person person) {
        if ("chan".equals(person.getName())) {
            return Observable.fromIterable(person.getPlanList()).delay(10, TimeUnit.MILLISECONDS);
        }
        return Observable.fromIterable(person.getPlanList());
    }
})
.subscribe(new Observer < Plan > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Plan plan) {
        Log.d(TAG, "==================plan " + plan.getContent());
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

为了更好的验证 flatMap 是无序的，使用了一个 delay() 方法来延迟：

```diff
05-21 13:57:14.031 21616-21616/com.example.rxjavademo D/chan: ==================plan chan 上课
==================plan chan 写作业
==================plan chan 打篮球
05-21 13:57:14.041 21616-21641/com.example.rxjavademo D/chan: ==================plan Zede 开会
==================plan Zede 写代码
==================plan Zede 写文章
```

可以看到本来 Zede 的事件发送顺序是排在 chan 事件之前，但是经过延迟后， 这两个事件序列发送顺序互换了。

现在来验证下 concatMap() 是否是有序的，使用上面同样的代码，只是把 flatMap() 换成 concatMap()，打印结果如下：

```diff
05-21 13:58:42.917 21799-21823/com.example.rxjavademo D/chan: ==================plan Zede 开会
==================plan Zede 写代码
==================plan Zede 写文章
==================plan chan 上课
==================plan chan 写作业
==================plan chan 打篮球
```

这就代表 concatMap() 转换后发送的事件序列是有序的了。

## 4、buffer()

```java
public final Observable<List<T>> buffer(int count, int skip)
```

从需要发送的事件当中获取一定数量的事件，并将这些事件放到缓冲区当中一并发出。

buffer 有两个参数，一个是 count，另一个 skip。count 缓冲区元素的数量，skip 就代表缓冲区满了之后，发送下一次事件序列的时候要跳过多少元素。这样说可能还是有点抽象，直接看代码：

```less
Observable.just(1, 2, 3, 4, 5)
.buffer(2, 1)
.subscribe(new Observer < List < Integer >> () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(List < Integer > integers) {
        Log.d(TAG, "================缓冲区大小： " + integers.size());
        for (Integer i: integers) {
            Log.d(TAG, "================元素： " + i);
        }
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

打印结果：

```ini
05-21 14:09:34.015 22421-22421/com.example.rxjavademo D/chan: ================缓冲区大小： 2
================元素： 1
================元素： 2
================缓冲区大小： 2
================元素： 2
================元素： 3
================缓冲区大小： 2
================元素： 3
================元素： 4
================缓冲区大小： 2
================元素： 4
================元素： 5
================缓冲区大小： 1
================元素： 5
```

从结果可以看出，每次发送事件，指针都会往后移动一个元素再取值，直到指针移动到没有元素的时候就会停止取值。

## 5、groupBy()

```swift
public final <K> Observable<GroupedObservable<K, T>> groupBy(Function<? super T, ? extends K> keySelector)
```

将发送的数据进行分组，每个分组都会返回一个被观察者。

```typescript
Observable.just(5, 2, 3, 4, 1, 6, 8, 9, 7, 10)
.groupBy(new Function < Integer, Integer > () {
    @Override
    public Integer apply(Integer integer) throws Exception {
        return integer % 3;
    }
})
.subscribe(new Observer < GroupedObservable < Integer, Integer >> () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "====================onSubscribe ");
    }

    @Override
    public void onNext(GroupedObservable < Integer, Integer > integerIntegerGroupedObservable) {
        Log.d(TAG, "====================onNext ");
        integerIntegerGroupedObservable.subscribe(new Observer < Integer > () {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "====================GroupedObservable onSubscribe ");
            }

            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "====================GroupedObservable onNext  groupName: " + integerIntegerGroupedObservable.getKey() + " value: " + integer);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "====================GroupedObservable onError ");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "====================GroupedObservable onComplete ");
            }
        });
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "====================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "====================onComplete ");
    }
});
```

在 groupBy() 方法返回的参数是分组的名字，每返回一个值，那就代表会创建一个组，以上的代码就是将1~10的数据分成3组，来看看打印结果：

```ini
05-26 14:38:02.062 21451-21451/com.example.rxjavademo D/chan: ====================onSubscribe 
05-26 14:38:02.063 21451-21451/com.example.rxjavademo D/chan: ====================onNext 
====================GroupedObservable onSubscribe     ====================GroupedObservable onNext  groupName: 2 value: 5
====================GroupedObservable onNext  groupName: 2 value: 2
====================onNext 
====================GroupedObservable onSubscribe 
====================GroupedObservable onNext  groupName: 0 value: 3
05-26 14:38:02.064 21451-21451/com.example.rxjavademo D/chan: ====================onNext 
====================GroupedObservable onSubscribe 
====================GroupedObservable onNext  groupName: 1 value: 4
====================GroupedObservable onNext  groupName: 1 value: 1
====================GroupedObservable onNext  groupName: 0 value: 6
====================GroupedObservable onNext  groupName: 2 value: 8
====================GroupedObservable onNext  groupName: 0 value: 9
====================GroupedObservable onNext  groupName: 1 value: 7
====================GroupedObservable onNext  groupName: 1 value: 10
05-26 14:38:02.065 21451-21451/com.example.rxjavademo D/chan: ====================GroupedObservable onComplete 
====================GroupedObservable onComplete 
====================GroupedObservable onComplete 
====================onComplete 
```

可以看到返回的结果中是有3个组的。

## 6、scan()

```r
public final Observable<T> scan(BiFunction<T, T, T> accumulator)
```

将数据以一定的逻辑聚合起来。

```php
Observable.just(1, 2, 3, 4, 5)
.scan(new BiFunction < Integer, Integer, Integer > () {
    @Override
    public Integer apply(Integer integer, Integer integer2) throws Exception {
        Log.d(TAG, "====================apply ");
        Log.d(TAG, "====================integer " + integer);
        Log.d(TAG, "====================integer2 " + integer2);
        return integer + integer2;
    }
})
.subscribe(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "====================accept " + integer);
    }
});
```

打印结果：

```ini
05-26 14:45:27.784 22519-22519/com.example.rxjavademo D/chan: ====================accept 1
====================apply 
====================integer 1
====================integer2 2
====================accept 3
====================apply 
05-26 14:45:27.785 22519-22519/com.example.rxjavademo D/chan: ====================integer 3
====================integer2 3
====================accept 6
====================apply 
====================integer 6
====================integer2 4
====================accept 10
====================apply 
====================integer 10
====================integer2 5
====================accept 15
```

## 2.7 window()

```java
public final Observable<Observable<T>> window(long count)
```

发送指定数量的事件时，就将这些事件分为一组。window 中的 count 的参数就是代表指定的数量，例如将 count 指定为2，那么每发2个数据就会将这2个数据分成一组。

```typescript
Observable.just(1, 2, 3, 4, 5)
.window(2)
.subscribe(new Observer < Observable < Integer >> () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "=====================onSubscribe ");
    }

    @Override
    public void onNext(Observable < Integer > integerObservable) {
        integerObservable.subscribe(new Observer < Integer > () {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "=====================integerObservable onSubscribe ");
            }

            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "=====================integerObservable onNext " + integer);
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "=====================integerObservable onError ");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "=====================integerObservable onComplete ");
            }
        });
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "=====================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "=====================onComplete ");
    }
});
```

打印结果：

```ini
05-26 15:02:20.654 25838-25838/com.example.rxjavademo D/chan: =====================onSubscribe 
05-26 15:02:20.655 25838-25838/com.example.rxjavademo D/chan: =====================integerObservable onSubscribe 
05-26 15:02:20.656 25838-25838/com.example.rxjavademo D/chan: =====================integerObservable onNext 1
=====================integerObservable onNext 2
=====================integerObservable onComplete 
=====================integerObservable onSubscribe 
=====================integerObservable onNext 3
=====================integerObservable onNext 4
=====================integerObservable onComplete 
=====================integerObservable onSubscribe 
=====================integerObservable onNext 5
=====================integerObservable onComplete 
=====================onComplete 
```

从结果可以发现，window() 将 1~5 的事件分成了3组。



# 四、组合操作符

## 1、concat()

```swift
public static <T> Observable<T> concat(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2, ObservableSource<? extends T> source3, ObservableSource<? extends T> source4)
```

可以将多个被观察者组合在一起，然后按照之前发送顺序发送事件。需要注意的是，concat() 最多只可以发送4个事件。

```less
Observable.concat(Observable.just(1, 2),
Observable.just(3, 4),
Observable.just(5, 6),
Observable.just(7, 8))
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```ini
05-21 15:40:26.738 7477-7477/com.example.rxjavademo D/chan: ================onNext 1
================onNext 2
05-21 15:40:26.739 7477-7477/com.example.rxjavademo D/chan: ================onNext 3
================onNext 4
================onNext 5
================onNext 6
================onNext 7
================onNext 8
```

## 2、concatArray()

```swift
public static <T> Observable<T> concatArray(ObservableSource<? extends T>... sources)
```

与 concat() 作用一样，不过 concatArray() 可以发送多于 4 个被观察者。

```less
Observable.concatArray(Observable.just(1, 2),
Observable.just(3, 4),
Observable.just(5, 6),
Observable.just(7, 8),
Observable.just(9, 10))
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```ini
05-21 15:47:18.581 9129-9129/com.example.rxjavademo D/chan: ================onNext 1
================onNext 2
================onNext 3
================onNext 4
================onNext 5
================onNext 6
================onNext 7
================onNext 8
================onNext 9
================onNext 10
```

## 3、merge()

```swift
public static <T> Observable<T> merge(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2, ObservableSource<? extends T> source3, ObservableSource<? extends T> source4)
```

这个方法与 concat() 作用基本一样，只是 concat() 是串行发送事件，而 merge() 并行发送事件。

```less
Observable.merge(
Observable.interval(1, TimeUnit.SECONDS).map(new Function < Long, String > () {
    @Override
    public String apply(Long aLong) throws Exception {
        return "A" + aLong;
    }
}),
Observable.interval(1, TimeUnit.SECONDS).map(new Function < Long, String > () {
    @Override
    public String apply(Long aLong) throws Exception {
        return "B" + aLong;
    }
}))
    .subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "=====================onNext " + s);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```

```bash
05-21 16:10:31.125 12801-12850/com.example.rxjavademo D/chan: =====================onNext B0
05-21 16:10:31.125 12801-12849/com.example.rxjavademo D/chan: =====================onNext A0
05-21 16:10:32.125 12801-12849/com.example.rxjavademo D/chan: =====================onNext A1
05-21 16:10:32.126 12801-12850/com.example.rxjavademo D/chan: =====================onNext B1
05-21 16:10:33.125 12801-12849/com.example.rxjavademo D/chan: =====================onNext A2
05-21 16:10:33.125 12801-12850/com.example.rxjavademo D/chan: =====================onNext B2
05-21 16:10:34.125 12801-12849/com.example.rxjavademo D/chan: =====================onNext A3
05-21 16:10:34.125 12801-12850/com.example.rxjavademo D/chan: =====================onNext B3
05-21 16:10:35.124 12801-12849/com.example.rxjavademo D/chan: =====================onNext A4
05-21 16:10:35.125 12801-12850/com.example.rxjavademo D/chan: =====================onNext B4
05-21 16:10:36.125 12801-12849/com.example.rxjavademo D/chan: =====================onNext A5
05-21 16:10:36.125 12801-12850/com.example.rxjavademo D/chan: =====================onNext B5
```

从结果可以看出，A 和 B 的事件序列都可以发出，将以上的代码换成 concat() 看看打印结果：

```bash
05-21 16:17:52.352 14597-14621/com.example.rxjavademo D/chan: =====================onNext A0
05-21 16:17:53.351 14597-14621/com.example.rxjavademo D/chan: =====================onNext A1
05-21 16:17:54.351 14597-14621/com.example.rxjavademo D/chan: =====================onNext A2
05-21 16:17:55.351 14597-14621/com.example.rxjavademo D/chan: =====================onNext A3
05-21 16:17:56.351 14597-14621/com.example.rxjavademo D/chan: =====================onNext A4
05-21 16:17:57.351 14597-14621/com.example.rxjavademo D/chan: =====================onNext A5
```

从结果可以知道，只有等到第一个被观察者发送完事件之后，第二个被观察者才会发送事件。

mergeArray() 与 merge() 的作用是一样的，只是它可以发送4个以上的被观察者，这里就不再赘述了。

## 4、concatArrayDelayError() & mergeArrayDelayError()

```swift
public static <T> Observable<T> concatArrayDelayError(ObservableSource<? extends T>... sources)
public static <T> Observable<T> mergeArrayDelayError(ObservableSource<? extends T>... sources)
```

在 concatArray() 和 mergeArray() 两个方法当中，如果其中有一个被观察者发送了一个 Error 事件，那么就会停止发送事件，如果你想 onError() 事件延迟到所有被观察者都发送完事件后再执行的话，就可以使用 concatArrayDelayError() 和 mergeArrayDelayError()

首先使用 concatArray() 来验证一下发送 onError() 事件是否会中断其他被观察者发送事件，代码如下：

```typescript
Observable.concatArray(
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onError(new NumberFormatException());
    }
}), Observable.just(2, 3, 4))
    .subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "===================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "===================onError ");
    }

    @Override
    public void onComplete() {

    }
});
```

打印结果：

```bash
05-21 16:38:59.725 17985-17985/com.example.rxjavademo D/chan: ===================onNext 1
===================onError 
```

从结果可以知道，确实中断了，现在换用 concatArrayDelayError()，代码如下：

```typescript
Observable.concatArrayDelayError(
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onError(new NumberFormatException());
    }
}), Observable.just(2, 3, 4))
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "===================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "===================onError ");
    }

    @Override
    public void onComplete() {

    }
});
```

打印结果如下：

```ini
05-21 16:40:59.329 18199-18199/com.example.rxjavademo D/chan: ===================onNext 1
===================onNext 2
===================onNext 3
===================onNext 4
===================onError 
```

从结果可以看到，onError 事件是在所有被观察者发送完事件才发送的。mergeArrayDelayError() 也是有同样的作用，这里不再赘述。

## 5、zip()

```swift
public static <T1, T2, R> Observable<R> zip(ObservableSource<? extends T1> source1, ObservableSource<? extends T2> source2, BiFunction<? super T1, ? super T2, ? extends R> zipper)
```

将多个被观察者合并，根据各个被观察者发送事件的顺序一个个结合起来，最终发送的事件数量会与源 Observable 中最少事件的数量一样。

```typescript
Observable.zip(Observable.intervalRange(1, 5, 1, 1, TimeUnit.SECONDS)
    .map(new Function<Long, String>() {
        @Override
        public String apply(Long aLong) throws Exception {
            String s1 = "A" + aLong;
            Log.d(TAG, "===================A 发送的事件 " + s1);
            return s1;
        }}),
        Observable.intervalRange(1, 6, 1, 1, TimeUnit.SECONDS)
            .map(new Function<Long, String>() {
            @Override
            public String apply(Long aLong) throws Exception {
                String s2 = "B" + aLong;
                Log.d(TAG, "===================B 发送的事件 " + s2);
                return s2;
            }
        }),
        new BiFunction<String, String, String>() {
            @Override
            public String apply(String s, String s2) throws Exception {
                String res = s + s2;
                return res;
            }
        })
.subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "===================onSubscribe ");
    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "===================onNext " + s);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "===================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "===================onComplete ");
    }
});
```

上面代码中有两个 Observable，第一个发送事件的数量为5个，第二个发送事件的数量为6个。现在来看下打印结果：

```bash
05-22 09:10:39.952 5338-5338/com.example.rxjavademo D/chan: ===================onSubscribe 
05-22 09:10:40.953 5338-5362/com.example.rxjavademo D/chan: ===================A 发送的事件 A1
05-22 09:10:40.953 5338-5363/com.example.rxjavademo D/chan: ===================B 发送的事件 B1
===================onNext A1B1
05-22 09:10:41.953 5338-5362/com.example.rxjavademo D/chan: ===================A 发送的事件 A2
05-22 09:10:41.954 5338-5363/com.example.rxjavademo D/chan: ===================B 发送的事件 B2
===================onNext A2B2
05-22 09:10:42.953 5338-5362/com.example.rxjavademo D/chan: ===================A 发送的事件 A3
05-22 09:10:42.953 5338-5363/com.example.rxjavademo D/chan: ===================B 发送的事件 B3
05-22 09:10:42.953 5338-5362/com.example.rxjavademo D/chan: ===================onNext A3B3
05-22 09:10:43.953 5338-5362/com.example.rxjavademo D/chan: ===================A 发送的事件 A4
05-22 09:10:43.953 5338-5363/com.example.rxjavademo D/chan: ===================B 发送的事件 B4
05-22 09:10:43.954 5338-5363/com.example.rxjavademo D/chan: ===================onNext A4B4
05-22 09:10:44.953 5338-5362/com.example.rxjavademo D/chan: ===================A 发送的事件 A5
05-22 09:10:44.953 5338-5363/com.example.rxjavademo D/chan: ===================B 发送的事件 B5
05-22 09:10:44.954 5338-5363/com.example.rxjavademo D/chan: ===================onNext A5B5
===================onComplete 
```

可以发现最终接收到的事件数量是5，那么为什么第二个 Observable 没有发送第6个事件呢？因为在这之前第一个 Observable 已经发送了 onComplete 事件，所以第二个 Observable 不会再发送事件。

## 6、combineLatest() & combineLatestDelayError()

```swift
public static <T1, T2, R> Observable<R> combineLatest(ObservableSource<? extends T1> source1, ObservableSource<? extends T2> source2, BiFunction<? super T1, ? super T2, ? extends R> combiner)
```

combineLatest() 的作用与 zip() 类似，但是 combineLatest() 发送事件的序列是与发送的时间线有关的，当 combineLatest() 中所有的 Observable 都发送了事件，只要其中有一个 Observable 发送事件，这个事件就会和其他 Observable 最近发送的事件结合起来发送，这样可能还是比较抽象，看看以下例子代码。

```typescript
Observable.combineLatest(
Observable.intervalRange(1, 4, 1, 1, TimeUnit.SECONDS)
    .map(new Function < Long, String > () {@Override
    public String apply(Long aLong) throws Exception {
        String s1 = "A" + aLong;
        Log.d(TAG, "===================A 发送的事件 " + s1);
        return s1;
    }
}),
Observable.intervalRange(1, 5, 2, 2, TimeUnit.SECONDS)
    .map(new Function < Long, String > () {@Override
    public String apply(Long aLong) throws Exception {
        String s2 = "B" + aLong;
        Log.d(TAG, "===================B 发送的事件 " + s2);
        return s2;
    }
}),
new BiFunction < String, String, String > () {@Override
    public String apply(String s, String s2) throws Exception {
        String res = s + s2;
        return res;
    }
})
.subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "===================onSubscribe ");
    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "===================最终接收到的事件 " + s);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "===================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "===================onComplete ");
    }
});
```

分析上面的代码，Observable A 会每隔1秒就发送一次事件，Observable B 会隔2秒发送一次事件。来看看打印结果：

```bash
05-22 11:41:20.859 15104-15104/? D/chan: ===================onSubscribe 
05-22 11:41:21.859 15104-15128/com.example.rxjavademo D/chan: ===================A 发送的事件 A1
05-22 11:41:22.860 15104-15128/com.example.rxjavademo D/chan: ===================A 发送的事件 A2
05-22 11:41:22.861 15104-15129/com.example.rxjavademo D/chan: ===================B 发送的事件 B1
05-22 11:41:22.862 15104-15129/com.example.rxjavademo D/chan: ===================最终接收到的事件 A2B1
05-22 11:41:23.860 15104-15128/com.example.rxjavademo D/chan: ===================A 发送的事件 A3
===================最终接收到的事件 A3B1
05-22 11:41:24.860 15104-15128/com.example.rxjavademo D/chan: ===================A 发送的事件 A4
05-22 11:41:24.861 15104-15129/com.example.rxjavademo D/chan: ===================B 发送的事件 B2
05-22 11:41:24.861 15104-15128/com.example.rxjavademo D/chan: ===================最终接收到的事件 A4B1
05-22 11:41:24.861 15104-15129/com.example.rxjavademo D/chan: ===================最终接收到的事件 A4B2
05-22 11:41:26.860 15104-15129/com.example.rxjavademo D/chan: ===================B 发送的事件 B3
05-22 11:41:26.861 15104-15129/com.example.rxjavademo D/chan: ===================最终接收到的事件 A4B3
05-22 11:41:28.860 15104-15129/com.example.rxjavademo D/chan: ===================B 发送的事件 B4
05-22 11:41:28.861 15104-15129/com.example.rxjavademo D/chan: ===================最终接收到的事件 A4B4
05-22 11:41:30.860 15104-15129/com.example.rxjavademo D/chan: ===================B 发送的事件 B5
05-22 11:41:30.861 15104-15129/com.example.rxjavademo D/chan: ===================最终接收到的事件 A4B5
===================onComplete 
```

分析上述结果可以知道，当发送 A1 事件之后，因为 B 并没有发送任何事件，所以根本不会发生结合。当 B 发送了 B1 事件之后，就会与 A 最近发送的事件 A2 结合成 A2B1，这样只有后面一有被观察者发送事件，这个事件就会与其他被观察者最近发送的事件结合起来了。

因为 combineLatestDelayError() 就是多了延迟发送 onError() 功能，这里就不再赘述了。

## 3.7 reduce()

```r
public final Maybe<T> reduce(BiFunction<T, T, T> reducer)
```

与 scan() 操作符的作用也是将发送数据以一定逻辑聚合起来，这两个的区别在于 scan() 每处理一次数据就会将事件发送给观察者，而 reduce() 会将所有数据聚合在一起才会发送事件给观察者。

```php
Observable.just(0, 1, 2, 3)
.reduce(new BiFunction < Integer, Integer, Integer > () {
    @Override
    public Integer apply(Integer integer, Integer integer2) throws Exception {
        int res = integer + integer2;
        Log.d(TAG, "====================integer " + integer);
        Log.d(TAG, "====================integer2 " + integer2);
        Log.d(TAG, "====================res " + res);
        return res;
    }
})
.subscribe(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "==================accept " + integer);
    }
});
```

打印结果：

```ini
05-22 14:21:46.042 17775-17775/? D/chan: ====================integer 0
====================integer2 1
====================res 1
====================integer 1
====================integer2 2
====================res 3
====================integer 3
====================integer2 3
====================res 6
==================accept 6
```

从结果可以看到，其实就是前2个数据聚合之后，然后再与后1个数据进行聚合，一直到没有数据为止。

## 3.8 collect()

```swift
public final <U> Single<U> collect(Callable<? extends U> initialValueSupplier, BiConsumer<? super U, ? super T> collector)
```

将数据收集到数据结构当中。

```java
Observable.just(1, 2, 3, 4)
.collect(new Callable < ArrayList < Integer >> () {
    @Override
    public ArrayList < Integer > call() throws Exception {
        return new ArrayList < > ();
    }
},
new BiConsumer < ArrayList < Integer > , Integer > () {
    @Override
    public void accept(ArrayList < Integer > integers, Integer integer) throws Exception {
        integers.add(integer);
    }
})
.subscribe(new Consumer < ArrayList < Integer >> () {
    @Override
    public void accept(ArrayList < Integer > integers) throws Exception {
        Log.d(TAG, "===============accept " + integers);
    }
});
```

打印结果：

```bash
05-22 16:47:18.257 31361-31361/com.example.rxjavademo D/chan: ===============accept [1, 2, 3, 4]
```

## 3.9 startWith() & startWithArray()

```java
public final Observable<T> startWith(T item)
public final Observable<T> startWithArray(T... items)
```

在发送事件之前追加事件，startWith() 追加一个事件，startWithArray() 可以追加多个事件。追加的事件会先发出。

```java
Observable.just(5, 6, 7)
.startWithArray(2, 3, 4)
.startWith(1)
.subscribe(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "================accept " + integer);
    }
});
```

打印结果：

```ini
05-22 17:08:21.282 4505-4505/com.example.rxjavademo D/chan: ================accept 1
================accept 2
================accept 3
================accept 4
================accept 5
================accept 6
================accept 7
```

## 3.10 count()

```java
public final Single<Long> count()
```

返回被观察者发送事件的数量。

```java
Observable.just(1, 2, 3)
.count()
.subscribe(new Consumer < Long > () {
    @Override
    public void accept(Long aLong) throws Exception {
        Log.d(TAG, "=======================aLong " + aLong);
    }
});
```

```bash
05-22 20:41:25.025 14126-14126/? D/chan: =======================aLong 3
```



# 五、功能操作符

## 1、delay()

```arduino
public final Observable<T> delay(long delay, TimeUnit unit)
```

延迟一段事件发送事件。

```typescript
Observable.just(1, 2, 3)
.delay(2, TimeUnit.SECONDS)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "=======================onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "=======================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {
        Log.d(TAG, "=======================onSubscribe");
    }
});
```

这里延迟了两秒才发送事件，来看看打印结果：

```bash
05-22 20:53:43.618 16880-16880/com.example.rxjavademo D/chan: =======================onSubscribe
05-22 20:53:45.620 16880-16906/com.example.rxjavademo D/chan: =======================onNext 1
05-22 20:53:45.621 16880-16906/com.example.rxjavademo D/chan: =======================onNext 2
=======================onNext 3
=======================onSubscribe
```

可以看出 onSubscribe 回调2秒之后 onNext 才会回调。

## 2、doOnEach()

```swift
public final Observable<T> doOnEach(final Consumer<? super Notification<T>> onNotification)
```

Observable 每发送一件事件之前都会先回调这个方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        //      e.onError(new NumberFormatException());
        e.onComplete();
    }
})
.doOnEach(new Consumer < Notification < Integer >> () {
    @Override
    public void accept(Notification < Integer > integerNotification) throws Exception {
        Log.d(TAG, "==================doOnEach " + integerNotification.getValue());
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 09:07:05.547 19867-19867/? D/chan: ==================onSubscribe 
==================doOnEach 1
==================onNext 1
==================doOnEach 2
==================onNext 2
==================doOnEach 3
==================onNext 3
==================doOnEach null
==================onComplete 
```

从结果就可以看出每发送一个事件之前都会回调 doOnEach 方法，并且可以取出 onNext() 发送的值。

## 3、doOnNext()

```swift
public final Observable<T> doOnNext(Consumer<? super T> onNext)
```

Observable 每发送 onNext() 之前都会先回调这个方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doOnNext(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "==================doOnNext " + integer);
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

打印结果：

```ini
05-23 09:09:36.769 20020-20020/com.example.rxjavademo D/chan: ==================onSubscribe 
==================doOnNext 1
==================onNext 1
==================doOnNext 2
==================onNext 2
==================doOnNext 3
==================onNext 3
==================onComplete 
```

## 4、doAfterNext()

```swift
public final Observable<T> doAfterNext(Consumer<? super T> onAfterNext)
```

Observable 每发送 onNext() 之后都会回调这个方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doAfterNext(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "==================doAfterNext " + integer);
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

打印结果：

```ini
05-23 09:15:49.215 20432-20432/com.example.rxjavademo D/chan: ==================onSubscribe 
==================onNext 1
==================doAfterNext 1
==================onNext 2
==================doAfterNext 2
==================onNext 3
==================doAfterNext 3
==================onComplete 
```

## 5、doOnComplete()

```java
public final Observable<T> doOnComplete(Action onComplete)
```

Observable 每发送 onComplete() 之前都会回调这个方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doOnComplete(new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doOnComplete ");
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

打印结果：

```ini
05-23 09:32:18.031 20751-20751/? D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================doOnComplete 
==================onComplete 
```

## 4.6 doOnError()

```swift
public final Observable<T> doOnError(Consumer<? super Throwable> onError)
```

Observable 每发送 onError() 之前都会回调这个方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onError(new NullPointerException());
    }
})
.doOnError(new Consumer < Throwable > () {
    @Override
    public void accept(Throwable throwable) throws Exception {
        Log.d(TAG, "==================doOnError " + throwable);
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

打印结果：

```ini
05-23 09:35:04.150 21051-21051/? D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================doOnError java.lang.NullPointerException
==================onError 
```

## 7、doOnSubscribe()

```swift
public final Observable<T> doOnSubscribe(Consumer<? super Disposable> onSubscribe)
```

Observable 每发送 onSubscribe() 之前都会回调这个方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doOnSubscribe(new Consumer < Disposable > () {
    @Override
    public void accept(Disposable disposable) throws Exception {
        Log.d(TAG, "==================doOnSubscribe ");
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 09:39:25.778 21245-21245/? D/chan: ==================doOnSubscribe 
==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================onComplete 
```

## 8、doOnDispose()

```java
public final Observable<T> doOnDispose(Action onDispose)
```

当调用 Disposable 的 dispose() 之后回调该方法。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doOnDispose(new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doOnDispose ");
    }
})
.subscribe(new Observer < Integer > () {
    private Disposable d;
    
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
        this.d = d;
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
        d.dispose();
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 09:55:48.122 22023-22023/com.example.rxjavademo D/chan: ==================onSubscribe 
==================onNext 1
==================doOnDispose 
```

## 9、doOnLifecycle()

```swift
public final Observable<T> doOnLifecycle(final Consumer<? super Disposable> onSubscribe, final Action onDispose)
```

在回调 onSubscribe 之前回调该方法的第一个参数的回调方法，可以使用该回调方法决定是否取消订阅。

doOnLifecycle() 第二个参数的回调方法的作用与 doOnDispose() 是一样的，现在用下面的例子来讲解：

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doOnLifecycle(new Consumer<Disposable>() {
    @Override
    public void accept(Disposable disposable) throws Exception {
        Log.d(TAG, "==================doOnLifecycle accept");
    }
}, new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doOnLifecycle Action");
    }
})
.doOnDispose(
    new Action() {
        @Override
        public void run() throws Exception {
            Log.d(TAG, "==================doOnDispose Action");
        }
})
.subscribe(new Observer<Integer>() {
    private Disposable d;
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
        this.d = d;
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
        d.dispose();
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    } 
});
```

```ini
05-23 10:20:36.345 23922-23922/? D/chan: ==================doOnLifecycle accept
==================onSubscribe 
==================onNext 1
==================doOnDispose Action
==================doOnLifecycle Action
```

可以看到当在 onNext() 方法进行取消订阅操作后，doOnDispose() 和 doOnLifecycle() 都会被回调。

如果使用 doOnLifecycle 进行取消订阅，来看看打印结果：

```bash
05-23 10:32:20.014 24652-24652/com.example.rxjavademo D/chan: ==================doOnLifecycle accept
==================onSubscribe 
```

可以发现 doOnDispose Action 和 doOnLifecycle Action 都没有被回调。

## 10、doOnTerminate() & doAfterTerminate()

```java
public final Observable<T> doOnTerminate(final Action onTerminate)
public final Observable<T> doAfterTerminate(Action onFinally)
```

doOnTerminate 是在 onError 或者 onComplete 发送之前回调，而 doAfterTerminate 则是 onError 或者 onComplete 发送之后回调。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
//      e.onError(new NullPointerException());
        e.onComplete();
    }
})
.doOnTerminate(new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doOnTerminate ");
    }
})
.subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
    
});
```

打印结果：

```ini
05-23 10:00:39.503 22398-22398/com.example.rxjavademo D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
05-23 10:00:39.504 22398-22398/com.example.rxjavademo D/chan: ==================onNext 3
==================doOnTerminate 
==================onComplete 
```

doAfterTerminate 也是差不多，这里就不再赘述。

## 11、doFinally()

```java
public final Observable<T> doFinally(Action onFinally)
```

在所有事件发送完毕之后回调该方法。

这里可能你会有个问题，那就是 doFinally() 和 doAfterTerminate() 到底有什么区别？区别就是在于取消订阅，如果取消订阅之后 doAfterTerminate() 就不会被回调，而 doFinally() 无论怎么样都会被回调，且都会在事件序列的最后。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.doFinally(new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doFinally ");
    }
})
.doOnDispose(new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doOnDispose ");
    }
})
.doAfterTerminate(new Action() {
    @Override
    public void run() throws Exception {
        Log.d(TAG, "==================doAfterTerminate ");
    }
})
.subscribe(new Observer<Integer>() {
    private Disposable d;
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
        this.d = d;
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
        d.dispose();
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

打印结果：

```diff
05-23 10:10:10.469 23196-23196/? D/chan: ==================onSubscribe 
05-23 10:10:10.470 23196-23196/? D/chan: ==================onNext 1
==================doOnDispose 
==================doFinally 
```

可以看到如果调用了 dispose() 方法，doAfterTerminate() 不会被回调。

现在试试把 dispose() 注释掉看看，看看打印结果：

```ini
05-23 10:13:34.537 23439-23439/com.example.rxjavademo D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================onComplete 
==================doAfterTerminate 
==================doFinally 
```

doAfterTerminate() 已经成功回调，doFinally() 还是会在事件序列的最后。

## 12、onErrorReturn()

```swift
public final Observable<T> onErrorReturn(Function<? super Throwable, ? extends T> valueSupplier)
```

当接受到一个 onError() 事件之后回调，返回的值会回调 onNext() 方法，并正常结束该事件序列。

```typescript
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onError(new NullPointerException());
    }
})
.onErrorReturn(new Function<Throwable, Integer>() {
    @Override
    public Integer apply(Throwable throwable) throws Exception {
        Log.d(TAG, "==================onErrorReturn " + throwable);
        return 404;
    }
})
.subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 18:35:18.175 19239-19239/? D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================onErrorReturn java.lang.NullPointerException
==================onNext 404
==================onComplete 
```

## 13、onErrorResumeNext()

```swift
public final Observable<T> onErrorResumeNext(Function<? super Throwable, ? extends ObservableSource<? extends T>> resumeFunction)
```

当接收到 onError() 事件时，返回一个新的 Observable，并正常结束事件序列。

```typescript
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onError(new NullPointerException());
    }
})
.onErrorResumeNext(new Function<Throwable, ObservableSource<? extends Integer>>() {
    @Override
    public ObservableSource<? extends Integer> apply(Throwable throwable) throws Exception {
        Log.d(TAG, "==================onErrorResumeNext " + throwable);
        return Observable.just(4, 5, 6);
    }
})
.subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 18:43:10.910 26469-26469/? D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================onErrorResumeNext java.lang.NullPointerException
==================onNext 4
==================onNext 5
==================onNext 6
==================onComplete 
```

## 14、onExceptionResumeNext()

```swift
public final Observable<T> onExceptionResumeNext(final ObservableSource<? extends T> next)
```

与 onErrorResumeNext() 作用基本一致，但是这个方法只能捕捉 Exception。

先来试试 onExceptionResumeNext() 是否能捕捉 Error。

```typescript
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onError(new Error("404"));
    }
})
.onExceptionResumeNext(new Observable<Integer>() {
    @Override
    protected void subscribeActual(Observer<? super Integer> observer) {
        observer.onNext(333);
        observer.onComplete();
    }
})
.subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 22:23:08.873 1062-1062/com.example.louder.rxjavademo D/chan: ==================onSubscribe 
05-23 22:23:08.874 1062-1062/com.example.louder.rxjavademo D/chan: ==================onNext 1
==================onNext 2
==================onNext 3
==================onError 
```

从打印结果可以知道，观察者收到 onError() 事件，证明 onErrorResumeNext() 不能捕捉 Error 事件。

将被观察者的 e.onError(new Error("404")) 改为 e.onError(new Exception("404"))，现在看看是否能捕捉 Exception 事件：

```ini
05-23 22:32:14.563 10487-10487/com.example.louder.rxjavademo D/chan: ==================onSubscribe 
==================onNext 1
==================onNext 2
==================onNext 3
==================onNext 333
==================onComplete 
```

从打印结果可以知道，这个方法成功捕获 Exception 事件。

## 15、retry()

```java
public final Observable<T> retry(long times)
```

如果出现错误事件，则会重新发送所有事件序列。times 是代表重新发的次数。

```typescript
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onError(new Exception("404"));
    }
})
.retry(2)
.subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 22:46:18.537 22239-22239/com.example.louder.rxjavademo D/chan: ==================onSubscribe 
05-23 22:46:18.538 22239-22239/com.example.louder.rxjavademo D/chan: ==================onNext 1
==================onNext 2
==================onNext 3
==================onNext 1
==================onNext 2
==================onNext 3
==================onNext 1
==================onNext 2
==================onNext 3
==================onError 
```

## 16、retryUntil()

```arduino
public final Observable<T> retryUntil(final BooleanSupplier stop)
```

出现错误事件之后，可以通过此方法判断是否继续发送事件。

```java
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onError(new Exception("404"));
    }
})
.retryUntil(new BooleanSupplier() {
    @Override
    public boolean getAsBoolean() throws Exception {
        if (i == 6) {
            return true;
        }
        return false;
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        i += integer;
        Log.d(TAG, "==================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```ini
05-23 22:57:32.905 23063-23063/com.example.louder.rxjavademo D/chan: ==================onSubscribe 
05-23 22:57:32.906 23063-23063/com.example.louder.rxjavademo D/chan: ==================onNext 1
==================onNext 2
==================onNext 3
==================onError 
```

## 17、retryWhen()

```java
public final void safeSubscribe(Observer<? super T> s)
```

当被观察者接收到异常或者错误事件时会回调该方法，这个方法会返回一个新的被观察者。如果返回的被观察者发送 Error 事件则之前的被观察者不会继续发送事件，如果发送正常事件则之前的被观察者会继续不断重试发送事件。

```typescript
Observable.create(new ObservableOnSubscribe < String > () {
    @Override
    public void subscribe(ObservableEmitter < String > e) throws Exception {
        e.onNext("chan");
        e.onNext("ze");
        e.onNext("de");
        e.onError(new Exception("404"));
        e.onNext("haha");
    }
})
.retryWhen(new Function < Observable < Throwable > , ObservableSource <? >> () {
    @Override
    public ObservableSource <? > apply(Observable < Throwable > throwableObservable) throws Exception {
        return throwableObservable.flatMap(new Function < Throwable, ObservableSource <? >> () {
            @Override
            public ObservableSource <? > apply(Throwable throwable) throws Exception {
                if(!throwable.toString().equals("java.lang.Exception: 404")) {
                    return Observable.just("可以忽略的异常");
                } else {
                    return Observable.error(new Throwable("终止啦"));
                }
            }
        });
    }
})
.subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "==================onSubscribe ");
    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "==================onNext " + s);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "==================onError " + e.toString());
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete ");
    }
});
```

```bash
05-24 09:13:25.622 28372-28372/com.example.rxjavademo D/chan: ==================onSubscribe 
05-24 09:13:25.623 28372-28372/com.example.rxjavademo D/chan: ==================onNext chan
==================onNext ze
==================onNext de
05-24 09:13:25.624 28372-28372/com.example.rxjavademo D/chan: ==================onError java.lang.Throwable: 终止啦
```

将 onError(new Exception("404")) 改为 onError(new Exception("303")) 看看打印结果：

```ini
==================onNext chan
05-24 09:54:08.653 29694-29694/? D/chan: ==================onNext ze
==================onNext de
==================onNext chan
==================onNext ze
==================onNext de
==================onNext chan
==================onNext ze
==================onNext de
==================onNext chan
==================onNext ze
==================onNext de
==================onNext chan
==================onNext ze
==================onNext de
==================onNext chan
......
```

从结果可以看出，会不断重复发送消息。

## 18、repeat()

```java
public final Observable<T> repeat(long times)
```

重复发送被观察者的事件，times 为发送次数。

```typescript
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.repeat(2)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "===================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "===================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {
        Log.d(TAG, "===================onComplete ");
    }
});
```

```ini
05-24 11:33:29.565 8544-8544/com.example.rxjavademo D/chan: ===================onSubscribe 
===================onNext 1
===================onNext 2
===================onNext 3
===================onNext 1
===================onNext 2
===================onNext 3
05-24 11:33:29.565 8544-8544/com.example.rxjavademo D/chan: ===================onComplete 
```

从结果可以看出，该事件发送了两次。

## 19、repeatWhen()

```swift
public final Observable<T> repeatWhen(final Function<? super Observable<Object>, ? extends ObservableSource<?>> handler)
```

这个方法可以会返回一个新的被观察者设定一定逻辑来决定是否重复发送事件。

这里分三种情况，如果新的被观察者返回 onComplete 或者 onError 事件，则旧的被观察者不会继续发送事件。如果被观察者返回其他事件，则会重复发送事件。

现在试验发送 onComplete 事件，代码如下：

```typescript
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.repeatWhen(new Function < Observable < Object > , ObservableSource <? >> () {
    @Override
    public ObservableSource <? > apply(Observable < Object > objectObservable) throws Exception {
        return Observable.empty();
    //  return Observable.error(new Exception("404"));
    //  return Observable.just(4); null;
    }
})
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "===================onSubscribe ");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "===================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "===================onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "===================onComplete ");
    }
});
```

```bash
05-24 11:44:33.486 9379-9379/com.example.rxjavademo D/chan: ===================onSubscribe 
05-24 11:44:33.487 9379-9379/com.example.rxjavademo D/chan: ===================onComplete 
```

下面直接看看发送 onError 事件和其他事件的打印结果。

发送 onError 打印结果：

```bash
05-24 11:46:29.507 9561-9561/com.example.rxjavademo D/chan: ===================onSubscribe 
05-24 11:46:29.508 9561-9561/com.example.rxjavademo D/chan: ===================onError 
```

发送其他事件的打印结果：

```ini
05-24 11:48:35.844 9752-9752/com.example.rxjavademo D/chan: ===================onSubscribe 
===================onNext 1
===================onNext 2
===================onNext 3
===================onComplete 
```

## 20、subscribeOn()

```arduino
public final Observable<T> subscribeOn(Scheduler scheduler)
```

指定被观察者的线程，要注意的时，如果多次调用此方法，只有第一次有效。

```typescript
Observable.create(new ObservableOnSubscribe < Integer > () {
    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        Log.d(TAG, "=========================currentThread name: " + Thread.currentThread().getName());
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
//.subscribeOn(Schedulers.newThread())
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "======================onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "======================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "======================onError");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "======================onComplete");
    }
});
```

现在不调用 subscribeOn() 方法，来看看打印结果：

```ini
05-26 10:40:42.246 21466-21466/? D/chan: ======================onSubscribe
05-26 10:40:42.247 21466-21466/? D/chan: =========================currentThread name: main
======================onNext 1
======================onNext 2
======================onNext 3
======================onComplete
```

可以看到打印被观察者的线程名字是主线程。

接着调用 subscribeOn(Schedulers.newThread()) 来看看打印结果：

```bash
05-26 10:43:26.964 22530-22530/com.example.rxjavademo D/chan: ======================onSubscribe
05-26 10:43:26.966 22530-22569/com.example.rxjavademo D/chan: =========================currentThread name: RxNewThreadScheduler-1
05-26 10:43:26.967 22530-22569/com.example.rxjavademo D/chan: ======================onNext 1
======================onNext 2
======================onNext 3
======================onComplete
```

可以看到打印结果被观察者是在一条新的线程。

现在看看多次调用会不会有效，代码如下：

```typescript
Observable.create(new ObservableOnSubscribe < Integer > () {

    @Override
    public void subscribe(ObservableEmitter < Integer > e) throws Exception {
        Log.d(TAG, "=========================currentThread name: " + Thread.currentThread().getName());
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onComplete();
    }
})
.subscribeOn(Schedulers.computation())
.subscribeOn(Schedulers.newThread())
.subscribe(new Observer < Integer > () {@Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "======================onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "======================onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "======================onError");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "======================onComplete");
    }
});
```

```ini
05-26 10:47:20.925 23590-23590/com.example.rxjavademo D/chan: ======================onSubscribe
05-26 10:47:20.930 23590-23629/com.example.rxjavademo D/chan: =========================currentThread name: RxComputationThreadPool-1
======================onNext 1
======================onNext 2
======================onNext 3
======================onComplete
```

可以看到第二次调动的 subscribeOn(Schedulers.newThread()) 并没有效果。

## 21、observeOn()

```arduino
public final Observable<T> observeOn(Scheduler scheduler)
```

指定观察者的线程，每指定一次就会生效一次。

```typescript
Observable.just(1, 2, 3)
.observeOn(Schedulers.newThread())
.flatMap(new Function < Integer, ObservableSource < String >> () {
    @Override
    public ObservableSource < String > apply(Integer integer) throws Exception {
        Log.d(TAG, "======================flatMap Thread name " + Thread.currentThread().getName());
        return Observable.just("chan" + integer);
    }
})
.observeOn(AndroidSchedulers.mainThread())
.subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "======================onSubscribe");
    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "======================onNext Thread name " + Thread.currentThread().getName());
        Log.d(TAG, "======================onNext " + s);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "======================onError");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "======================onComplete");
    }
});
```

打印结果：

```ini
05-26 10:58:04.593 25717-25717/com.example.rxjavademo D/chan: ======================onSubscribe
05-26 10:58:04.594 25717-25753/com.example.rxjavademo D/chan: ======================flatMap Thread name RxNewThreadScheduler-1
05-26 10:58:04.595 25717-25753/com.example.rxjavademo D/chan: ======================flatMap Thread name RxNewThreadScheduler-1
======================flatMap Thread name RxNewThreadScheduler-1
05-26 10:58:04.617 25717-25717/com.example.rxjavademo D/chan: ======================onNext Thread name main
======================onNext chan1
======================onNext Thread name main
======================onNext chan2
======================onNext Thread name main
======================onNext chan3
05-26 10:58:04.618 25717-25717/com.example.rxjavademo D/chan: ======================onComplete
```

从打印结果可以知道，observeOn 成功切换了线程。

下表总结了 RxJava 中的调度器：

| 调度器                         | 作用                                       |
| ------------------------------ | ------------------------------------------ |
| Schedulers.computation( )      | 用于使用计算任务，如事件循环和回调处理     |
| Schedulers.immediate( )        | 当前线程                                   |
| Schedulers.io( )               | 用于 IO 密集型任务，如果异步阻塞 IO 操作。 |
| Schedulers.newThread( )        | 创建一个新的线程                           |
| AndroidSchedulers.mainThread() | Android 的 UI 线程，用于操作 UI。          |



# 六、过滤操作符





# 七、条件操作符



































































# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 脑图



# 参考

1、https://juejin.cn/post/6844903617124630535
