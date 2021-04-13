---
layout: post
author: "ooftf"
tags: Android
---
# 四大组件
1. Activity
2. Service
3. BroadcastReceiver
4. ContentProvider

* Android的四大组件中除了BroadcastReceiver意外，其他三种组件都必须在AndroidManifest中注册
* 显示Intent可以分为显示Intent和饮食Intent，显示Intent明确的指向一个Activity组件，隐式Intent则指向零个或多个目标Activity
* service有两种状态：启动状态和绑定状态。
* Service本身是运行在主线程中的，因此好事的后台计算需要在单独的线程中去完成
* BroadcastReceiver两种注册方式：静态注射和动态注册。静态注册是指在AndroidManifest中注册广播，这种广播在应用安装时会被系统解析，此种形式的广播不需要应用启动就可以收到相应的广播；动态注册广播需要通过Context。registerReceiver()来实现的，ing且在不需要的时候要通过Context.unRegisterReceiver()来解除广播
* ContentProvider内部的 inset、delete、update、query方法需要处理好线程同步，应为这几个方法实在Binder线程池中被调用的，
* ContentProvider不需要手动停止


## Activity
![生命周期](http://hi.csdn.net/attachment/201109/1/0_1314838777He6C.gif)
[生命周期](https://blog.csdn.net/xiajun2356033/article/details/78741121)
* onCreate
* setContentView
* LifecycleObserver onCreate
* onStart
* onPostCreate
* activeCou
* onResume
* 
--------------------------------------------------------------------
#### Activity启动流程
#### 从点击图标到Activity展示过程
#### 启动模式
###### singleInstance
**字面上理解为单一实例。  它具备所有singleTask的特点，唯一不同的是，它是存在于另一个任务栈中。上面的三种模式都存在于同一个任务栈中，
而这种模式则是存在于另一个任务栈中。举个例子，上面的启动模式都存在于地球上，而这种模式存在于火星上。
整个Android系统就是个宇宙。**

* singleInstance之一坑  
此时有三个activity，ActivityA，ActivityB，ActivityC，除了ActivityB的启动模式为singleInstance，
其他的启动模式都为默认的。startActivity了一个ActivityA，在ActivityA里startActivity了一个ActivityB，
在ActivityB里startActivity了一个ActivityC。此时在当前的任务栈中的顺序是，ActivityA->ActivityB->ActivityC。
照理来说在当前ActivityC页面按返回键，finish当前界面后应当回到ActivityB界面。但是事与愿违，奇迹出现了，页面直接回到了ActivityA。
这是为什么呢？其实想想就能明白了，上面已经说过，singleInstance模式是存在于另一个任务栈中的。也就是说ActivityA和ActivityC是处于同一个任务栈中的，ActivityB则是存在另个栈中。
所以当关闭了ActivityC的时候，它自然就会去找当前任务栈存在的activity。当前的activity都关闭了之后，才会去找另一个任务栈中的activity。
也就是说当在ActivityC中finish之后，会回到ActivityA的界面，在ActivityA里finish之后会回到ActivityB界面。
*  singleInstance之二坑  
此时有两个个activity，ActivityA，ActivityB，ActivityA的启动模式为默认的，
ActivityB的启动模式为singleInstance。当在ActivityA里startActivity了ActivityB，
当前页面为ActivityB。按下home键。应用退到后台。此时再点击图标进入APP，按照天理来说，
此时的界面应该是ActivityB，可是奇迹又出现了，当前显示的界面是ActivityA。这是因为当重新启动的时候，
系统会先去找主栈（我是这么叫的）里的activity，也就是APP中LAUNCHER的activity所处在的栈。
查看是否有存在的activity。没有的话则会重新启动LAUNCHER。
