
# AOP（面向切面编程）方案
    AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑
    各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
### 主要用途
    日志记录，性能统计，安全控制，事务处理，异常处理等等。
    将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改变
    这些行为的时候不影响业务逻辑的代码。

### AspectJ
    它在代码的编译期间扫描目标程序，根据切点（PointCut）匹配,将开发者编写的Aspect程序编织（Weave）到目标程序的.class文件中，  
    对目标程序作了重构（重构单位是JoinPoint），  
    目的就是建立目标程序与Aspect程序的连接（获得执行的对象、方法、参数等上下文信息），从而达到AOP的目的。
#### 不支持kotlin
* [资料](https://blog.csdn.net/weelyy/article/details/78987087)
* [AspectJX](https://github.com/HujiangTechnology/gradle_plugin_android_aspectjx)
##  Transform API
### 能力
    在编译器生成.class文件之后，混淆之前，获取class文件并进行修改
比如Android打包过程的混淆就是依靠Transform API技术来完成的  
![图片](https://oscimg.oschina.net/oscnet/86f8adb0-8713-4349-b2f7-384fcf81a64b.png)

[推荐文章,Transform API+ASM完成代码织入](https://my.oschina.net/u/4568043/blog/4521743)  
##### 相关框架
* [lancet](https://github.com/eleme/lancet)
* [AndAOP](https://github.com/luckybilly/AndAOP)
* [AutoRegister](https://github.com/luckybilly/AutoRegister)
#### ASM
    Transfrom API获取到class文件后通过ASM框架完成字节码的修改
[JVM 指令](https://blog.csdn.net/tanggao1314/article/details/53260891)
[ASM 指令](https://www.jianshu.com/p/d8c2ada6e82f)
[ASM 实操](https://segmentfault.com/a/1190000022403863)
[ASM 实操](https://blog.csdn.net/fedorafrog/article/details/104538652/)
### 相关库
Hunter
ByteBuddy
#### 相关文章
## Javassist API
[【Android】函数插桩（Gradle + ASM）](https://www.jianshu.com/p/16ed4d233fd1)  
[AOP 的利器：ASM 3.0 介绍](https://developer.ibm.com/zh/articles/j-lo-asm30/)

## Javassist相比于ASM
* **Javassist**API比**ASM**友好，**ASM**性能是**Javassist**的五倍左右


