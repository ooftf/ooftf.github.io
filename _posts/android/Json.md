## Problems
---
* 如果json中有某个对象不想解析，但是想找个对象去接收，不要使用org.json.JSONObject，
如果使用org.json.JSONObject;在解析的过程中会被忽略掉，不会得到对应的结果。
如果你使用的是fastjson可以使用com.alibaba.fastjson.JSONObject 可以得到想要的结果但是不推荐（FastJson已确认可以）
同理Gson等也要使用对应包下的JSONObject才可以（Gson未确认，处于猜想阶段）
* 如果json解析过程中出错，fastjson和gson都会立即中断解析，后面字段将不会解析
但是jackjson会跳过出错字段继续解析，最终结果只会缺少出错的字段
* JavaBean set 和 get 方法都要设置为public

### Kotlin
//"{"name":"lihang","desc":"dashuaibi"}"
```kotlin
data class GsonBean(val name: String, val desc: String) {
}
val gsonString = "{\"desc\":\"dashuaibi\"}"
// 结论：name 字段为null
//使用 deployDefaultValue  name : null


val gString = "{\"name\":null,\"desc\":\"dashuaibi\"}"
//使用 deployDefaultValue  name : ""
```
```kotlin
val gsonString = "{\"desc\":\"dashuaibi\"}"
data class GsonBean(val name: String = "lihang", val desc: String) {
}
// name : null

```


```kotlin

class GsonBean {
    var name: String = "lihang"
    val desc: String = ""
}
val gsonString = "{\"desc\":\"dashuaibi\"}"
// name = lihang
// desc = dashuaibi


val gString = "{\"name\":null,\"desc\":\"dashuaibi\"}"
// name = null
// desc = dashuaibi

//使用 deployDefaultValue  name : ""
```

结论：  
构造方法默认值是没有作用的  
成员方法默认值是有作用的，在Json没有该变量的时候会采用默认变量  
deployDefaultValue只会在Json中为null的时候起作用，如果没有该变量是没有作用的