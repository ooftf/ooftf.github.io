# 内存
## 内存模型（JMM）
![内存模型](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTQ4OTY2OS8yMDE4MTAvMTQ4OTY2OS0yMDE4MTAwOTE4NTUyNzMxNi0xNzA4NzkwOTc0LnBuZw?x-oss-process=image/format,png)
## 内存区域
### 相关文章
https://blog.csdn.net/laomo_bible/article/details/83067810
https://blog.csdn.net/singwhatiwanna/article/details/111713858?spm=1001.2014.3001.5501
    但类似于 “基本变量++” / “volatile++” 这种复合操作并没有原子性。比如 i++;
### 线程私有
1. 线程私有：程序计数器、栈、本地方法栈
2. 线程共享：堆、方法区
* 本地方法栈
* 程序计数器
* 堆
* 栈（虚拟机栈）
* 方法区（JDK 1.7）
* 元空间（JDK 1.8）
### 问题
* 1.7和1.8有什么不同
* 常量池指的什么
* 字符串常量存放在哪里
## GC
* 如何判定为垃圾
* 如何进行回收
* 垃圾回收是如何减少空间碎片和提高效率
* 什么时候会触发GC
* 什么时候对象从年轻代转移到老年代
    经历15此回收（15这个值可改变）
    大对象（字符串与数组），即超过了设定的值的对象，直接在老年代中分配
    首先根据 TargetSurvivorRatio 指定的比例，乘以 survivor 一个区的大小，得出目标晋升空间大小。然后将分代对象大小，按照分代年龄从小到大相加，直到大于目标晋升空间大小。之后，将得出的这个分代年龄以上的对象全部晋升。
* 为什么新生代内存需要有两个Survivor区
  减少内存碎片化 [为什么新生代内存需要有两个Survivor区](https://blog.csdn.net/antony9118/article/details/51425581)
* Minor GC 和Full GC
### GC时机 (?)
    YGC（Minor GC）的时机:
    edn空间不足
    FGC（Full GC）的时机：
    1.old空间不足；
    2.perm空间不足；
    3.显示调用System.gc() ，包括RMI等的定时触发;
    4.YGC时的悲观策略；
    5.dump live的内存信息时(jmap –dump:live)。
### GC Root  两个版本
来自LeakCanary文档
* Local variables, which belong to the stack of a thread.
* Instances of active Java threads.
* System Classes, which never unload.
* Native references, which are controlled by native code.
出自sun公司java团队员工书籍
* Temporary variables on the stack (of any thread)
* Static variables (from any class)
* Special references from JNI native code。

### 资料
[Java对象循环引用，Java gc 如何回收](https://blog.csdn.net/leonardo9029/article/details/50241115)
[GC算法](https://www.cnblogs.com/feng9exe/p/7268524.html)



