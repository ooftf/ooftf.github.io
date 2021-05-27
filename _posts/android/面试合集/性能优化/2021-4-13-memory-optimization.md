---
layout: post
author: "ooftf"
top: true
tags:
    - Android
    - Optimization
---

# 内存优化
###  误区：内存占用的越少越好

内存不足常常会引起一些崩溃问题，有些同学就会认为内存占用的越少性能越好，导致优化内存过程中“用力过猛”
应用是否占用了过多的内存，和系统和设备当时的情况有关，而不是一个绝对数值。当系统内存充足时，我们可以占用多用一些获得更好的的性能。当系统内存不足是，希望可以做到“用时分配，及时释放”，
能够迅速释放各种缓存减少系统的压力

###  采用设备分级机制

判断手机等级
在低级手机上减少一些大消耗内存的动作，比如动画，和加载大图的动作，减少某些功能，使用565格式的图片

###  缓存管理

需要有一套统一的缓存管理机制，可以适当的使用内存，当系统内存不足时，及时的释放内存，使用onTrimMemory监控内存情况


Glide.get(applicationContext).clearMemory()

###  进程管理

一个空进程也会占用10MB的内存，减小应用启动的进程数，减少常驻进程，有节操的保活，对低端机非常重要

###  安装包大小

安装包中的代码、资原、图片以及SO库的体积，跟它们占用的内存有很大的关系，我们可以考虑针对低端机用户推出4MB的轻量版本，例如今日头条极速版

### Bitmap优化

##### 收拢图片调用

图片内存优化的前提时收拢图片的调用，这样我们可以做整体的控制策略。可以使用Glide,Fresco或者自研都可以。而且需要进一步将所有Bitmap.createBitmap、BitmapFactory相关的接口也一并收拢

##### 统一监控

* **大图监控**  
    在开发中如果检测到长宽远远大于View甚至屏幕的长宽的图片体积提醒开发人员，在灰度或者线上情况，可以将异常信息上报到后台。  
    Glide设置全局图片加载监听
    ```kotlin
    Glide.init(this, GlideBuilder().addGlobalRequestListener(object :    RequestListener<Any> {
             override fun onLoadFailed(
                 e: GlideException?,
                 model: Any?,
                 target: Target<Any>?,
                 isFirstResource: Boolean
             ): Boolean {
                 return false
             }

             override fun onResourceReady(
                 resource: Any?,
                 model: Any?,
                 target: Target<Any>?,
                 dataSource: DataSource?,
                 isFirstResource: Boolean
             ): Boolean {

                 Log.e("onResourceReady",Thread.currentThread().toString()+""    +resource.toString())
                 return false
             }

         }))
    ``` 
* **重复图片监控**  
    Bitmap像素完全一致，但是有多个不同的对象存在。
* **图片总内存**  
    通过收拢图片使用，我们还可以统计所有图片占用的内存

### 内存泄露

内存泄露大多都是因为单例、子线程不合理的使用导致的，LeakCanary自动化检测方案，可以做到Activity和Fragment的泄露

### 内存监控

应用在前台的时候，可以每过5分钟采集一次PSS、Java堆内存、图片总内存、建议通过采样只统计部分用户
PSS值可以通过Debug.MemoryInfo看到

### GC监控
开发过程或者内部试用环境可以通过Debug.startAllocCounting来监控Java内存分配和GC情况
