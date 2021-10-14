## Flow
* Flow 库是在 Kotlin Coroutines 1.3.2 发布之后新增的库，也叫做异步流，类似 RxJava 的 Observable 、 Flowable 等等
* [在 MVVM 架构中使用 Kotlin Flow](https://www.jianshu.com/p/0696c7252b50)


### 冷流和热流

* 冷流 :只有订阅者订阅时，才开始执行发射数据流的代码。并且冷流和订阅者只能是一对一的关系，当有多个不同的订阅者时，消息是重新完整发送的。也就是说对冷流而言，有多个订阅者的时候，他们各自的事件是独立的。
* 热流:无论有没有订阅者订阅，事件始终都会发生。当 热流有多个订阅者时，热流与订阅者们的关系是一对多的关系，可以与多个订阅者共享信息。

Flow 和 RxJava 中 Obsevable 都是冷流

传统观察者者模式是热流
#### Flow创建
* ​flow { ... }​
* flowOf 创建一个保护固定数量的flow，类似listOf
* 任意集合类或者squence通过​.asFlow()​转成一个flow  
  (1..3).asFlow().collect { value -> println(value) }

```kotlin
fun foo(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}
 
fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    foo().collect { value -> println(value) } 
}
//  输出结果：
//  I'm not blocked 1
//  1
//  I'm not blocked 2
//  2
//  I'm not blocked 3
//  3

```
通过上面例子可以看到，Flow有以下特征:
* ​flow{ ... }​ 构建一个Flow类型
* ​flow { ... }​内可以使用suspend函数.
* foo()​不需要是suspend函数
* emit方法用来发射数据
* collect方法用来遍历结果
* Flows是冷流，即在调用collect之前，flow{ ... }中的代码不会执行

## Kotlin 泛型 * 和 Any 有什么不同
kotlin <*> 相当于 <out Any?>

## 可变参数 和 数组 

```kotlin
fun main(argsArray: Array<out String>,vararg args: String) {
    foo(*argsArray)
    foo(*args)
    fooArray(argsArray)
    fooArray(args)
}

fun foo(vararg args: String) {
  
}
fun fooArray(vararg args: String) {
  
}
```