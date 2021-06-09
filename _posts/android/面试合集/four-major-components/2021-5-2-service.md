## 获取当前Service 列表,高版本只能获取到自己APP的 Service 无法获取到其他App的 Service
```kotlin
  fun listServices(mContext: Context): MutableList<ActivityManager.RunningServiceInfo> {
        val activityManager = mContext.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
        return activityManager.getRunningServices(100)
    }
```

### Service生命周期概述
* onCreate：当用Context.startService,Context.bindService启动服务时，会回调onCreate，在整个生命周期中，只会回调一次。也是生命周期方法中第一个被回调的，系统将调用次方法来执行一次性设置程序
* onStartCommand：用Context.startService启动Service会回调这个方法，每次启动都会回调，Context.bindService启动Service不会回调这个方法。
* onBind：Context.bindService启动Service回调这个方法，启动者可通过该方法返回对象来对Service对象进行操控。
* onReBind：当使用startService启动Service，又调用bindService启动Service，且 onUnbind 返回值为 true 时，下次再次调用 Context.bindService 将触发方法。
* onUnBind：用 Context.unbindService 触发此方法，默认返回 false, 当返回值 true 后，再次调用 Context.bindService 时将触发 onRebind 方法。
* onDestory：1.以Context.startService启动service，调用Context.stopService停止服务回调此方法;2.以Context.bindService启动service,以Context.unbindService停止服务回调此方法;3.先以Context.startService 启动服务，再用Context.bindService绑定服务，结束时必须先调用Context.unbindService解绑再使用Context.stopService停止service才会回调此方法，否则会报错。这里做一些资源释放操作

### 按启动方式分两类：
1. 通过Context.startService()启动，调用Context.stopService()停止服务。
   多次启动结果
   ```
   startService     onCreate
                    onStartCommand
   startService     onStartCommand
   startService     onStartCommand
   startService     onStartCommand
   startService     onStartCommand
   startService     onStartCommand
   stopService      onDestroy
   ```
   一个Service是可以多次启动的，但是onCreate只会回调一次，而onStartCommand是启动一次就回调一次。onStartCommand这个方法是有返回值的并且是我们开发者可以控制返回什么值。我们知道当系统内存是有限的，当系统内存资源不足，Service是会被销毁的，如果你在Service里做了什么重要事情，那被销毁显然是你不愿意看到的，所以要有一种方法让系统帮我们重启该服务，那要不要重启就由这个返回值决定了
   * START_STICKY：如果service进程被kill掉，系统会尝试重新创建Service，如果在此期间没有任何启动命令被传递到  Service，那么参数intent将为null。
   * START_NOT_STICKY：使用这个返回值时，如果在执行完onStartCommand()后，服务被异常kill掉，系统不会自动重启 该服务。
   * START_REDELIVER_INTENT：使用这个返回值时，如果在执行完onStartCommand()后，服务被异常kill掉，系统会自动 重启该服务，并将intent的值传入。
   * START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启。
   如果把返回值设置成START_STICKY和START_REDELIVER_INTENT，这样就可以处于不死状态了（这样做其实是不好的）还 有  一种情况是用户主动在设置界面把你的Service给杀死，但是这会回调onDestroy，你可以在这里发送广播重新启动。
   onStartCommand()第二个输入参数flags正是代表此次onStartCommand()方法的启动方式，正常启动时，flags默认为 0，被kill后重新启动，参数分为以下两种：
   * START_FLAG_RETRY：代表service被kill后重新启动，由于上次返回值为START_STICKY，所以参数 intent 为null
   * START_FLAG_REDELIVERY：代表service被kill后重新启动，由于上次返回值为START_REDELIVER_INTENT，所以带输 入  参数intent
2. 通过Context.bindService启动service,以Context.unbindService停止服务。
   bindService(Intent,ServiceConnection,flags);
   * Intent：构建的连接Activity和Service的对象
   * ServiceConnection： 重写两个方法，就是通过这个类的onServiceconnected方法获得MyBind实例，从而让Activity可以调用Service类的公共方法；onServiceDisconnected是服务意外中断的时候调用的，不是开发者主动停止服务调用的
   * flags：这是一个int型数，一般填入Context.BIND_AUTO_CREATE,表示Activity和Service建立关联后自动创建Service，可以让Service的onCreate执行，但是onStartCommand不会执行
   ```
   bindService                Service:onCreate
                              Service:onBind
                              ServiceConnection::onServiceConnected
   unbindService(connection)  onUnbind
                              onDestroy                 
   ```
  * 不管是startService还是bindService，最后都要stopService和unBindService与之对应，要不然容易造成内存泄漏。
  * bindService后调用unBindService成功后，如果再调用unBindService将会报异常，可以加一个boolean变量判断。
  * 这里有一种特殊情况是先startService，此时回调onCreate->onStartCommand，然后再bindService，回调onBind->onServiceConnected;这时候要想销毁Service必须调用 unBindService，再调用stopService。
  * bindService成功后，Service就与当前Activity绑定了，它就跟随Activity生命周期走了，如果服务没有销毁，而与之绑定的Activity销毁了，那这个绑定的Service也会被销 毁，但是Service里启动的子线程不会被销毁。
  * 服务是运行在后台，没错，但是不能进行耗时操作，否则会ANR，页面卡死情况；如果有耗时操作一定要新建一个子线程，放在里进行，这里也说明Service和Thread不是同一种东  西，就跟猫和鱼一样，是两个不同物种。但是通常情况下这两个是组合起来使用。
  * 有人会问了，既然Service不能做耗时操作，就是做也得放在子线程里去做，那为啥不直接在Activity里开一个子线程去做呢，反而弄Service这么麻烦。有道理，但是Activity是  很难对线程去操作的，当线程所在的Activity销毁了，那这个线程就是脱缰的野马，没人能管的了了，你没办法再获取这个线程的实例了，除非把进程给杀了；但是如果线程放  在      Service里，就算Activity销毁了，以后只要重新与这个Service进行绑定，在onServiceConnected获取binder实例，这样就可以继续操作线程了，要它GG还是要它继续与  你作伴，就看你高兴了。
  * 通过startService启动服务后，服务就与启动它的组件没有联系了，组件销毁了，服务还是继续运行；通过bindService启动服务，与之绑定的组件就可以控制Service的具体执行  逻辑了。
  * 
## 推荐链接
[Service 详解](https://blog.csdn.net/qq_30993595/article/details/78452064)