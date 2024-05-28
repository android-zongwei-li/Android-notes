

> review：2024/5/27

# 接口回调中Lambda使用

使用Lambda函数可以简化接口回调方法

> 注：仅支持单个抽象方法回调，多个回调方法不支持。

```kotlin
 // Java接口回调
mVar.setEventListener(new ExamEventListener(){

    public void onSuccess(Data data){
      // ...
    }
 
 });

// 同等效果的Kotlin接口回调（无使用lambda表达式）
mVar.setEventListener(object: ExamEventListener{
     
    public void onSuccess(Data data){
      // ...
    } 
});

// Kotlin接口回调（使用lambda表达式，仅留下参数）
mVar.setEventListener({
   data: Data ->
   // ... 
})

// 继续简化
// 简化1：借助kotlin的智能类型推导，忽略数据类型
mVar.setEventListener({
   data ->
   // ... 
})

// 简化2：若参数无使用，可忽略
mVar.setEventListener({
   // ... 
})

// 简化3：若setEventListener函数最后一个参数是一个函数，可把括号的实现提到圆括号外
mVar.setEventListener(){
   // ... 
}

// 简化3：若setEventListener函数只有一个参数 & 无使用到，可省略圆括号
mVar.setEventListener{
   // ... 
}
```
