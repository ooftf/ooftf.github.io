---
layout: post
author: "ooftf"
tags: [Android,Activity,Service,BroadcastReceiver,ContentProvider]
---
# 四大组件
1. Activity
2. Service
3. BroadcastReceiver
4. ContentProvider

* Android的四大组件中除了BroadcastReceiver意外，其他三种组件都必须在AndroidManifest中注册
* 显示Intent可以分为显示Intent和饮食Intent，显示Intent明确的指向一个Activity组件，隐式Intent则指向零个或多个目标Activity

* BroadcastReceiver 两种注册方式：静态注射和动态注册。静态注册是指在AndroidManifest中注册广播，这种广播在应用安装时会被系统解析，此种形式的广播不需要应用启动就可以收到相应的广播；动态注册广播需要通过Context。registerReceiver()来实现的，ing且在不需要的时候要通过Context.unRegisterReceiver()来解除广播
* ContentProvider 内部的 inset、delete、update、query方法需要处理好线程同步，应为这几个方法实在Binder线程池中被调用的，
* ContentProvider不需要手动停止

## Service
* service 有两种状态：启动状态和绑定状态。
* Service 本身是运行在主线程中的，因此耗时的任务需要在单独的线程中去完成

#### IntentService
* 内部实现是由HandlerThread + Handler方式实现。
* 在子线程中处理任务。
* 在所有任务处理完成后自动推出。
* 只处理了onStartCommand方法，onBind为空实现，代表只能由startService方式启动才有效果，并不支持bindService
* Android 8.0以上不推荐使用IntentService了，Google推荐使用JobIntentService





