## 一些概念
* 高阶函数
* DSL
* 带接收者的lambda
* init方法
* apply等方法
* [invoke约定](https://www.jianshu.com/p/e802954a0695)
* 扩展函数
* inline内联
* [函数库](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt)
* 协变和逆变

## init
    是在构造方法之后自动调用

### 在kotlin项目中 注解处理器要使用kapt代替annotationProcessor，annotationProcessor（可能会失效？）
    添加 apply plugin: 'kotlin-kapt'
### databinding 在布局中条用 kotlin 伴生类的方法，调用不到（使用@JvmStatic）

### kotlin   ARouter在autowired 的变量要添加@JvmFiled  
### data class 为空会怎样  
### 高阶函数
### [DSL](https://www.jianshu.com/p/e802954a0695)  
### 带接收者的lambda