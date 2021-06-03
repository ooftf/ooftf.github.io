---
layout: post
author: "ooftf"
top: true
tags:
    - Android
    - Optimization
---
## 启动过程
![启动耗时](https://github.com/ooftf/ooftf.github.io/blob/master/images/APP启动图.jpg?raw=true)
如图所示
* T1 预览窗口显示。系统在拉起进程之前，会先根据Application的 Theme 属性创建预览窗口。当然如果我们禁用预览窗口或者将预览窗口指定为透明，
    用户在这段时间依然看到的是桌面。
* T2 闪屏显示。在进程和闪屏窗口页面创建完毕，并且完成一系列 inflate view、onmeasure、onlayout 等准备工作后，用户终于可以看到启动Activity。
* T3 主页显示。在完成主窗口创建和页面显示的准备工作后，用户可以看到应用的主界面。
* T4 界面可操作。在启动完成后，会有比较多的工作需要继续执行，例如界面的预加载、小程序框架和进程的准备等。在这些工作完成后，用户才可以真正开始愉快地操作APP。



## App启动过程
在冷启动的过程中，首先会通过 AMS 在 System 进程展示一个 Starting Window (通常情况下是个白屏，可以通过设置 Application 的 theme 修改)，接着AMS会通过Zygote创建应用程序的进程，并通过一系列的步骤后调用 Application 的 attachBaseContext()、onCreate() 然后最终调用 Activity 的 onCreate() 以及进行View相关的初始化工作。在Activity展示出来后会替换掉之前的 Starting Window，这样启动过程结束。
## 启动时长
1. 计算从 Application 的 attachBaseContext 方法开始到 第一个Activity 的 onWindowFocusChanged() 方法之间的时间，这样就可以初步得到应用的冷启动时间
2. 在 Logcat 控制台搜索 Displayed 这个 tag ，就可以看到所有 App 的启动时长
3. 通过Adb命令获取启动时长
   ```command
   adb shell am start -S -R 5 -W {ApplicationId}/{入口Activity全路径： com.chaitai.LauncherActivity}
   ```
   -S：表示每次启动前先强行停止  
   -R：表示重复测试次数
4. Trace 获取启动时长 （不推荐，推荐 Profiler）
    ```java
    //开始
    Debug.startMethodTracing("onCreate")
    //结束
    Debug.stopMethodTracing()
    ```
    生成的 trace 文件的路径： /storage/emulated/0/Android/data/com.chaitai.crm/files/onCreate.trace  
    adb pull 方式导出文件，需要 root 权限  
    如果没有 root 权限，可以通过手机文件系统导出文件  
    打开 Profiler 工具点击加号选择“load from file...” 选择 xxx.trace 文件
5. Profiler获取启动时长（推荐）
    1. 打开 Edit configurations... 
    2. 点击 Profiling 标签
    3. 勾选 Start this recording on startup
    4. 点击 Run按钮右边的 Profile 按钮
    5. 运行到界面展示完成后点击 stop record

## 常见问题
* 点击图标很久都不响应
    如果我们禁用了预览窗口或者指定了透明的皮肤，那用户点击了图标之后，需要 T2 时间才能真正看到应用闪屏。对于用户体验来说，点击了图标，过了几秒还是停留在桌面，看起来就像没有点击成功，这在中低端机中更加明显。
* 首页显示太慢
    现在应用启动流程越来越复杂，闪屏广告、热修复框架、插件化框架、大前端框架，所有准备工作都需要集中在启动阶段完成。上面说的 T3 首页显示时间对于中低端机来说简直就是噩梦，经常会达到十几秒的时间。
* 首页显示后无法操作。
    既然首页显示那么慢，那我能不能把尽量多的工作都通过异步化延后执行呢？很多应用的确就是这么做的，但这会造成两种后果：要么首页会出现白屏，要么首页出来后用户根本无法操作。
    很多应用把启动结束时间的统计放到首页刚出现的时候，这对用户是不负责任的。看到一个首页，但是停住十几秒都不能滑动，这对用户来说完全没有意义。启动优化不能过于 KPI 化，要从用户的真实体验出发，要着眼从点击图标到用户可操作的整个过程。

## 具体优化内容
具体的优化方式，我把它们分为闪屏优化、业务梳理、业务优化、线程优化、GC 优化和系统调用优化。
* 闪屏优化
    要解决闪屏优化，就要明白为什么冷启动时为什么会有一段时间的白屏或者黑屏。  
    这是因为冷启动时首先展示的不是Acitivity而是一个 "Starting Window" 这个Window 可以通过设置 可以通过设置 Application 的 theme 的 android:windowBackground 修改  
    今日头条把预览窗口实现成闪屏的效果，这样用户只需要很短的时间就可以看到“预览闪屏”。这种完全“跟手”的感觉在高端机上体验非常好，但对于中低端机，会把总的的闪屏时间变得更长。
    如果点击图标没有响应，用户主观上会认为是手机系统响应比较慢。所以我比较推荐的做法是，只在 Android 6.0 或者 Android 7.0 以上才启用“预览闪屏”方案，让手机性能好的用户可以有更好的体验。
    微信做的另外一个优化是合并闪屏和主页面的 Activity，减少一个 Activity 会给线上带来 100 毫秒左右的优化。但是如果这样做的话，管理时会非常复杂，特别是有很多例如 PWA、扫一扫这样的第三方启动流程的时候。
* 业务梳理
    我们首先需要梳理清楚当前启动过程正在运行的每一个模块，哪些是一定需要的、哪些可以砍掉、哪些可以懒加载。我们也可以根据业务场景来决定不同的启动模式，例如通过扫一扫启动只需要加载需要的几个模块即可。对于中低端机器，我们要学会降级，学会推动产品经理做一些功能取舍。但是需要注意的是，懒加载要防止集中化，否则容易出现首页显示后用户无法操作的情形。
* 业务优化
    通过梳理之后，剩下的都是启动过程一定要用的模块。这个时候，我们只能硬着头皮去做进一步的优化。优化前期需要“抓大放小”，先看看主线程究竟慢在哪里。最理想是通过算法进行优化，例如一个数据解密操作需要 1 秒，通过算法优化之后变成 10 毫秒。退而求其次，我们要考虑这些任务是不是可以通过异步线程预加载实现，但需要注意的是过多的线程预加载会让我们的逻辑变得更加复杂。
    业务优化做到后面，会发现一些架构和历史包袱会拖累我们前进的步伐。比较常见的是一些事件会被各个业务模块监听，大量的回调导致很多工作集中执行，部分框架初始化“太厚”，例如一些插件化框架，启动过程各种反射、各种 Hook，整个耗时至少几百毫秒。还有一些历史包袱又非常沉重，而且“牵一发动全身”，改动风险比较大。但是我想说，如果有合适的时机，我们依然需要勇敢去偿还这些“历史债务”。
* 线程优化
    很多启动框架，会使用 Pipeline 机制，根据业务优先级规定业务初始化时机。比如微信内部使用的mmkernel、阿里最近开源的Alpha启动框架，它们为各个任务建立依赖关系，最终构成一个有向无环图。对于可以并发的任务，会通过线程池最大程度提升启动速度。如果任务的依赖关系没有配置好，很容易出现下图这种情况，即主线程会一直等待 taskC 结束，空转 2950 毫秒。
* 系统调用优化
    通过 systrace 的 System Service 类型，我们可以看到启动过程 System Server 的 CPU 工作情况。在启动过程，我们尽量不要做系统调用，例如 PackageManagerService 操作、Binder 调用等待。
    在启动过程也不要过早地拉起应用的其他进程，System Server 和新的进程都会竞争 CPU 资源。特别是系统内存不足的时候，当我们拉起一个新的进程，可能会成为“压死骆驼的最后一根稻草”。它可能会触发系统的 low memory killer 机制，导致系统杀死和拉起（保活）大量的进程，从而影响前台进程的 CPU。
    讲个实践的案例，之前我们的一个程序在启动过程会拉起下载和视频播放进程，改为按需拉起后，线上启动时间提高了 3%，对于 1GB 以下的低端机优化，整个启动时间可以优化 5%～8%，效果还是非常明显的。
* 保活
    保活可以减少 Application 创建跟初始化的时间，让冷启动变成温启动。不过在 Target 26 之后，保活的确变得越来越难。
    微信的 Hardcoder 方案和 OPPO 推出的Hyper Boost方案。根据 OPPO 的数据，对于手机 QQ、淘宝、微信启动场景会直接有 20% 以上的优化。
    有的时候你问为什么微信可以保活？为什么它可以运行的那么流畅？这里可能不仅仅是技术上的问题，当应用体量足够大，就可以倒逼厂商去专门为它们做优化。
    在App的主页面重写返回操作改为退出到后台，而不是关闭 Activity
* 线上监控
    Android Vitals可以对应用冷启动、温启动时间做监控。
    事实上，每个应用启动的流程都非常复杂，上面的图并不能真实反映每个应用的启动耗时。启动耗时的计算需要考虑非常多的细节，比如：
    启动结束的统计时机。是否是使用用户真正可以操作的时间作为启动结束的时间。
    启动时间扣除的逻辑。闪屏、广告和新手引导这些时间都应该从启动时间里扣除。
    启动排除逻辑。Broadcast、Server 拉起，启动过程进入后台这些都需要排除出统计。
    经过精密的扣除和排除逻辑，我们最终可以得到用户的线上启动耗时。正如我在上一期所说的，准确的启动耗时统计是非常重要的。有很多优化在实验室完成之后，还需要在线上灰度验证效果。这个前提是启动统计是准确的，整个效果评估是真实的。

## Application的创建以及初始化流程
LoadedApk.makeApplication
```java
  public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
        if (mApplication != null) {
            return mApplication;
        }
        app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext);

        instrumentation.callApplicationOnCreate(app);
        return app;
    }

```
Instrumentation.newApplication
```java
    public Application newApplication(ClassLoader cl, String className, Context context)
            throws InstantiationException, IllegalAccessException, 
            ClassNotFoundException {
        Application app = getFactory(context.getPackageName())
                .instantiateApplication(cl, className);
        app.attach(context);
        return app;
    }
```
Application.attach
```java
    final void attach(Context context) {
        attachBaseContext(context);
        mLoadedApk = ContextImpl.getImpl(context).mPackageInfo;
    }
```
由代码可知 attachBaseContext 是在 onCreate 之前被调用的
## 相关框架
* [Android异步启动框架 alpha](https://github.com/alibaba/alpha)



