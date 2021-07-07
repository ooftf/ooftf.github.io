## [CUP](https://developer.android.google.cn/studio/profile/cpu-profiler)
点击Record按钮开始记录，点击Stop按钮结束选取
#### 功能
* 查看各个线程占用的CUP资原
* 查看函数调用

[官网文档](https://developer.android.com/studio/profile/cpu-profiler#configurations)
![CPU](https://raw.githubusercontent.com/ooftf/Material/master/img/blogprofiler-cup.png)
如图所示
* 红色部分：历史Record记录
* 黄色部分：操作台用于发起Record
* 蓝色部分：当前CPU占用情况
* 绿色部分：各个线程占用CUP情况，由可知现在线程com.chaitai.crm （主线程）占用大量CPU资原
* Trace Java Methods
    会记录每个方法的时间、CPU信息。对运行时性能影响较大。
* Sample Java Methods
    相比于Trace Java Methods会记录每个方法的时间、CPU信息，它会在应用的Java代码执行期间频繁捕获应用的调用堆栈，对运行时性能的影响比较小，能够记录更大的数据区域。
* Sample C/C++ Functions
    需部署到Android 8.0及以上设备，内部使用simpleperf跟踪应用的native代码，也可以命令行使用simpleperf。
* Trace System Calls
    检查应用与系统资源的交互情况。 查看所有核心的CPU瓶。 内部采用systrace，也可以使用systrace命令。

![CPU](https://raw.githubusercontent.com/ooftf/Material/master/img/blogprofiler_cup_detail.png)
* 红色区域选择查看线程
* 黄色区域查看选择线程的方法调用
* 蓝色区域选择查看的视图
* top down 为从调用者开始查看
* bottom up 从被调用者开始查看


## [MEMORY](https://developer.android.google.cn/studio/profile/memory-profiler)
点击 dump 按钮查看详情
#### 功能
* 查看内存占用情况
* 查看各个类的实例对象个数以及具体内容
* 查看垃圾回收触发时间

![MEMORY_LIST](https://raw.githubusercontent.com/ooftf/Material/master/img/blogprofiler_memory_first.png)
* 蓝色区域为手动触发GC按钮
* 绿色区域为内存占用图示
* 红色区域为发生GC时间点
* 黄色区域为查看当前内存详情按钮

![MEMORY_second](https://raw.githubusercontent.com/ooftf/Material/master/img/blogprofiler_memory_second.png)
* 红色区域搜索目标类
* 白色区域存在内存泄漏问题的类的个数
* 黄色区域为搜索结果列表
* 蓝色为选定类每个实例的详情
* 绿色为指定实例的详情，其中Fields为其成员变量，Refeeences为引用了当前类的实例
* Shallow Size 是指实例自身占用的内存, 可以理解为保存该'数据结构'需要多少内存, 注意不包括它引用的其他实例
* Retained Size, 当实例被回收时, 可以同时被回收的实例的Shallow Size之和

## [NETWORK](https://developer.android.google.cn/studio/profile/network-profiler)
用鼠标划取时间段，查看详情
![MEMORY_second](https://raw.githubusercontent.com/ooftf/Material/master/img/blogprofiler_network_summary.png)
![MEMORY_second](https://raw.githubusercontent.com/ooftf/Material/master/img/blogprofiler_network_detail.png)
* 红色为选取时间段所有请求列表
* 黄色区域为选择请求详情


#### 功能
* 查看网络占用情况
* 查看网络请求具体内容

## [ENERGY](https://developer.android.google.cn/studio/profile/energy-profiler)