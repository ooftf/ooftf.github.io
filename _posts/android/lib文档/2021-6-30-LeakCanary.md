---
layout: post
author: "ooftf"
tags: Android
title: LeakCanary
top: true
---

## [LeakCanary](https://square.github.io/leakcanary)

## 使用
```groovy
dependencies {
  // debugImplementation because LeakCanary should only run in debug builds.
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.7'
}
```

## LeakCanary 有哪些功能
* 检查 Activity 、Fragment 、 Fragment 的 view、ViewModel 、Dialog 的 RootView、 Service 的内存泄漏问题
* com.squareup.leakcanary:plumber-android:2.7 这个库修复了一些系统内存泄漏问题
* AndroidLeakFixes 这个类是 plumber 修复的一些问题

## 系统级内存泄漏
* MediaSessionLegacyHelper
* TextLine
* UserManager
* HandlerThread
* AccessibilityNodeInfo
* ConnectivityManager
* ClipboardUIManager
* EditText
* TextView
* ActivityManager
* ViewLocationHolder
* InputMethodManager
* ViewRootImpl
* TextView


## LeakCanary 是如何做到初始化不用编写代码

LeakCanary 添加了一个名字叫做 AppWatcherInstaller 的 ContentProvider，由于 ContentProvider 在App 启动的时候会被调用 onCreate 方法，所以在 AppWatcherInstaller.onCreate 做了 LeakCanary 的初始化工作
```XML
<application>
    <provider
        android:name="leakcanary.internal.AppWatcherInstaller$MainProcess"
        android:authorities="${applicationId}.leakcanary-installer"
        android:enabled="@bool/leak_canary_watcher_auto_install"
        android:exported="false" />
</application>
```
```kotlin
  override fun AppWatcherInstaller.onCreate(): Boolean {
    val application = context!!.applicationContext as Application
    AppWatcher.manualInstall(application)
    return true
  }
```
从 provider 的代码可知，默认使用 leakcanary.internal.AppWatcherInstaller$MainProcess 初始化 LeakCanary ， 如果不想自动初始化 LeakCanary ，可以覆盖 leak_canary_watcher_auto_install 变量为 false ，然后调用 AppWatcher.manualInstall(application) 初始化

## LeakCanary 初始化工作
```kotlin
  // retainedDelayMillis 是内存泄漏检测时间，当需要被检测的类生成弱引用添加到观察队列，retainedDelayMillis 后会对该弱引用进行检查判断是否已经被回收

  //watchersToInstall 用来监测对象生命周期的观察者
  fun AppWatcher.manualInstall(
    application: Application,
    retainedDelayMillis: Long = TimeUnit.SECONDS.toMillis(5),
    watchersToInstall: List<InstallableWatcher> = appDefaultWatchers(application)
  ) {
    this.retainedDelayMillis = retainedDelayMillis
    watchersToInstall.forEach {
      it.install()
    }
  }
```

```kotlin
  // 从 AppWatcher.appDefaultWatchers 可知 LeakCanary 有四个内存泄漏监测模块，其实在 FragmentAndViewModelWatcher 内部还使用了 ViewModelClearedWatcher ,所以一共是5个
  fun appDefaultWatchers(
    application: Application,
    reachabilityWatcher: ReachabilityWatcher = objectWatcher
  ): List<InstallableWatcher> {
    return listOf(
      ActivityWatcher(application, reachabilityWatcher),
      FragmentAndViewModelWatcher(application, reachabilityWatcher),
      RootViewWatcher(reachabilityWatcher),
      ServiceWatcher(reachabilityWatcher)
    )
  }
```
## 5个内存泄漏监听模块

#### 1. ActivityWatcher(application, reachabilityWatcher)
```kotlin
class ActivityWatcher(
  private val application: Application,
  private val reachabilityWatcher: ReachabilityWatcher
) : InstallableWatcher {

  private val lifecycleCallbacks =
    object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
      override fun onActivityDestroyed(activity: Activity) {
        reachabilityWatcher.expectWeaklyReachable(
          activity, "${activity::class.java.name} received Activity#onDestroy() callback"
        )
      }
    }

  override fun install() {
    application.registerActivityLifecycleCallbacks(lifecycleCallbacks)
  }

  override fun uninstall() {
    application.unregisterActivityLifecycleCallbacks(lifecycleCallbacks)
  }
}
```

总结：通过 application.registerActivityLifecycleCallbacks 监听到 activity 生命结束

#### 2. FragmentAndViewModelWatcher(application, reachabilityWatcher)
```kotlin

class FragmentAndViewModelWatcher(
  private val application: Application,
  private val reachabilityWatcher: ReachabilityWatcher
) : InstallableWatcher {

  private val fragmentDestroyWatchers: List<(Activity) -> Unit> = run {
    val fragmentDestroyWatchers = mutableListOf<(Activity) -> Unit>()
    //android.app.Fragment
    // Android 8 以上监听 onFragmentViewDestroyed 声明周期的的 观察者
    if (SDK_INT >= O) {
      fragmentDestroyWatchers.add(
        AndroidOFragmentDestroyWatcher(reachabilityWatcher)
      )
    }
    //androidx.fragment.app.Fragment
    //查看是否支持 AndroidX 如果支持，创建 AndroidXFragmentDestroyWatcher 观察者实例 
    getWatcherIfAvailable(
      ANDROIDX_FRAGMENT_CLASS_NAME,
      ANDROIDX_FRAGMENT_DESTROY_WATCHER_CLASS_NAME,
      reachabilityWatcher
    )?.let {
      fragmentDestroyWatchers.add(it)
    }
    //android.support.v4.app.Fragment
    //查看是否支持 Support 如果支持，创建 AndroidSupportFragmentDestroyWatcher 观察者实例 
    getWatcherIfAvailable(
      ANDROID_SUPPORT_FRAGMENT_CLASS_NAME,
      ANDROID_SUPPORT_FRAGMENT_DESTROY_WATCHER_CLASS_NAME,
      reachabilityWatcher
    )?.let {
      fragmentDestroyWatchers.add(it)
    }
    fragmentDestroyWatchers
  }
  // 检测 Activity.onCreate 生命周期，当检测到 Activity 创建后调用 Watcher 的 invoke 方法 
  private val lifecycleCallbacks =
    object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
      override fun onActivityCreated(
        activity: Activity,
        savedInstanceState: Bundle?
      ) {
        for (watcher in fragmentDestroyWatchers) {
          // 调用 AndroidOFragmentDestroyWatcher，AndroidSupportFragmentDestroyWatcher，AndroidXFragmentDestroyWatcher 的 invoke(activity: Activity)方法
          watcher(activity)
        }
      }
    }

  override fun install() {
    application.registerActivityLifecycleCallbacks(lifecycleCallbacks)
  }

  override fun uninstall() {
    application.unregisterActivityLifecycleCallbacks(lifecycleCallbacks)
  }

}
```

总结：通过 application.registerActivityLifecycleCallbacks 监听到 activity 创建，然后通过 activity.supportFragmentManager.registerFragmentLifecycleCallbacks 监听到 Fragment.view 和 Fragment 生命结束，涉及到一些不同类型 Fragment 的兼容
```kotlin
internal class AndroidXFragmentDestroyWatcher(
  private val reachabilityWatcher: ReachabilityWatcher
) : (Activity) -> Unit {

  private val fragmentLifecycleCallbacks = object : FragmentManager.FragmentLifecycleCallbacks() {

    override fun onFragmentCreated(
      fm: FragmentManager,
      fragment: Fragment,
      savedInstanceState: Bundle?
    ) {
      // 这个地方注册了 Fragment 的 ViewModel 的泄漏监听
      ViewModelClearedWatcher.install(fragment, reachabilityWatcher)
    }
    //当监听到 Fragment.onDestroyed 将 fragment 添加到泄漏检测
    override fun onFragmentViewDestroyed(
      fm: FragmentManager,
      fragment: Fragment
    ) {
      val view = fragment.view
      if (view != null) {
        reachabilityWatcher.expectWeaklyReachable(
          view, "${fragment::class.java.name} received Fragment#onDestroyView() callback " +
          "(references to its views should be cleared to prevent leaks)"
        )
      }
    }
  // 当监听到到 Fragment.onViewDestroyed 将 fragment.view 添加到泄漏检测
    override fun onFragmentDestroyed(
      fm: FragmentManager,
      fragment: Fragment
    ) {
      reachabilityWatcher.expectWeaklyReachable(
        fragment, "${fragment::class.java.name} received Fragment#onDestroy() callback"
      )
    }
  }
  // 通过 registerFragmentLifecycleCallbacks 注册 Fragment 声明周期检测

  override fun invoke(activity: Activity) {
    if (activity is FragmentActivity) {
      val supportFragmentManager = activity.supportFragmentManager
      supportFragmentManager.registerFragmentLifecycleCallbacks(fragmentLifecycleCallbacks, true)
      // 这个地方注册了 Activity 的 ViewModel 的泄漏监听 
      ViewModelClearedWatcher.install(activity, reachabilityWatcher)
    }
  }
}
```

#### 3. RootViewWatcher(reachabilityWatcher)
```kotlin
val className = if (SDK_INT > 16) {
      "android.view.WindowManagerGlobal"
    } else {
      "android.view.WindowManagerImpl"
    }
class RootViewWatcher(
  private val reachabilityWatcher: ReachabilityWatcher
) : InstallableWatcher {

  private val listener = OnRootViewAddedListener { rootView ->
    val trackDetached = when(rootView.windowType) {
      //当监听到添加 PhoneWindow 的 rootView ,监听 Dialog 的 rootView
      PHONE_WINDOW -> {
        when (rootView.phoneWindow?.callback?.wrappedCallback) {
          // Activities are already tracked by ActivityWatcher
          is Activity -> false
          is Dialog -> rootView.resources.getBoolean(R.bool.leak_canary_watcher_watch_dismissed_dialogs)
          // Probably a DreamService
          else -> true
        }
      }
      // Android widgets keep detached popup window instances around.
      POPUP_WINDOW -> false
      TOOLTIP, TOAST, UNKNOWN -> true
    }
    if (trackDetached) {
      rootView.addOnAttachStateChangeListener(object : OnAttachStateChangeListener {
        // 监听到 onViewDetachedFromWindow 将 rootView 添加到泄漏检测
        val watchDetachedView = Runnable {
          reachabilityWatcher.expectWeaklyReachable(
            rootView, "${rootView::class.java.name} received View#onDetachedFromWindow() callback"
          )
        }

        override fun onViewAttachedToWindow(v: View) {
          mainHandler.removeCallbacks(watchDetachedView)
        }

        override fun onViewDetachedFromWindow(v: View) {
          // 这里为什么要用 Handler? 有可能是子线程？
          mainHandler.post(watchDetachedView)
        }
      })
    }
  }

  override fun install() {
    
    /* 通过 Curtains 监听 RootView 的创建，
    Curtains 也是 Square 开发的一个库，
    通过反射 "android.view.WindowManagerGlobal" 或者 "android.view.WindowManagerImpl" 
    的 mView 字段，给 mView 添加代理，监听 mView 的变化 */
    Curtains.onRootViewsChangedListeners += listener
  }

  override fun uninstall() {
    Curtains.onRootViewsChangedListeners -= listener
  }
}    
```

总结：通过代理 WindowManagerGlobal 的 mView ，监听到 rootView 的添加，再通过 rootView.addOnAttachStateChangeListener 
监听到 rootView 生命周期结束

#### 4. ServiceWatcher(reachabilityWatcher
```kotlin
class ServiceWatcher(private val reachabilityWatcher: ReachabilityWatcher) : InstallableWatcher {

  private val servicesToBeDestroyed = WeakHashMap<IBinder, WeakReference<Service>>()

  private val activityThreadClass by lazy { Class.forName("android.app.ActivityThread") }

  private val activityThreadInstance by lazy {
    activityThreadClass.getDeclaredMethod("currentActivityThread").invoke(null)!!
  }

  private val activityThreadServices by lazy {
    val mServicesField =
      activityThreadClass.getDeclaredField("mServices").apply { isAccessible = true }

    @Suppress("UNCHECKED_CAST")
    mServicesField[activityThreadInstance] as Map<IBinder, Service>
  }

  private var uninstallActivityThreadHandlerCallback: (() -> Unit)? = null
  private var uninstallActivityManager: (() -> Unit)? = null

  override fun install() {
    try {
      swapActivityThreadHandlerCallback { mCallback ->
        uninstallActivityThreadHandlerCallback = {
          swapActivityThreadHandlerCallback {
            mCallback
          }
        }
        Handler.Callback { msg ->
          // 通过给 ActivityThread.mH 的 callback 添加代理，监听 STOP_SERVICE 事件，再通过 msg.obj 找到对应的 Service,  用来监听 Service Destory 消息，这个时候还没有真正的关闭 Service 
          if (msg.what == STOP_SERVICE) {
            val key = msg.obj as IBinder
            activityThreadServices[key]?.let {
              onServicePreDestroy(key, it)
            }
          }
          mCallback?.handleMessage(msg) ?: false
        }
      }
      // 通过反射代理 android.app.IActivityManager 对象，监听 serviceDoneExecuting 方法调用，获取到 Service onDestory 方法调用时机,但是这个时候是拿不到 Service 对象的，所以需要上面代码拿到的 Service 实例对象
      swapActivityManager { activityManagerInterface, activityManagerInstance ->
        uninstallActivityManager = {
          swapActivityManager { _, _ ->
            activityManagerInstance
          }
        }
        Proxy.newProxyInstance(
          activityManagerInterface.classLoader, arrayOf(activityManagerInterface)
        ) { _, method, args ->
          if (METHOD_SERVICE_DONE_EXECUTING == method.name) {
            val token = args!![0] as IBinder
            if (servicesToBeDestroyed.containsKey(token)) {
              onServiceDestroyed(token)
            }
          }
          try {
            if (args == null) {
              method.invoke(activityManagerInstance)
            } else {
              method.invoke(activityManagerInstance, *args)
            }
          } catch (invocationException: InvocationTargetException) {
            throw invocationException.targetException
          }
        }
      }
    } catch (ignored: Throwable) {
      SharkLog.d(ignored) { "Could not watch destroyed services" }
    }
  }

  override fun uninstall() {
    checkMainThread()
    uninstallActivityManager?.invoke()
    uninstallActivityThreadHandlerCallback?.invoke()
    uninstallActivityManager = null
    uninstallActivityThreadHandlerCallback = null
  }

  private fun onServicePreDestroy(
    token: IBinder,
    service: Service
  ) {
    servicesToBeDestroyed[token] = WeakReference(service)
  }
  // 当监听到 service detroyed 将 service 添加到泄漏检测
  private fun onServiceDestroyed(token: IBinder) {
    servicesToBeDestroyed.remove(token)?.also { serviceWeakReference ->
      serviceWeakReference.get()?.let { service ->
        reachabilityWatcher.expectWeaklyReachable(
          service, "${service::class.java.name} received Service#onDestroy() callback"
        )
      }
    }
  }

  //通过反射 将 ActivityThread.H 的 Callback 添加一层代理 
  private fun swapActivityThreadHandlerCallback(swap: (Handler.Callback?) -> Handler.Callback?) {
    val mHField =
      activityThreadClass.getDeclaredField("mH").apply { isAccessible = true }
    val mH = mHField[activityThreadInstance] as Handler

    val mCallbackField =
      Handler::class.java.getDeclaredField("mCallback").apply { isAccessible = true }
    val mCallback = mCallbackField[mH] as Handler.Callback?
    mCallbackField[mH] = swap(mCallback)
  }

  @SuppressLint("PrivateApi")
  private fun swapActivityManager(swap: (Class<*>, Any) -> Any) {
    val singletonClass = Class.forName("android.util.Singleton")
    val mInstanceField =
      singletonClass.getDeclaredField("mInstance").apply { isAccessible = true }

    val singletonGetMethod = singletonClass.getDeclaredMethod("get")

    val (className, fieldName) = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
      "android.app.ActivityManager" to "IActivityManagerSingleton"
    } else {
      "android.app.ActivityManagerNative" to "gDefault"
    }

    val activityManagerClass = Class.forName(className)
    val activityManagerSingletonField =
      activityManagerClass.getDeclaredField(fieldName).apply { isAccessible = true }
    val activityManagerSingletonInstance = activityManagerSingletonField[activityManagerClass]

    val activityManagerInstance = singletonGetMethod.invoke(activityManagerSingletonInstance)

    val iActivityManagerInterface = Class.forName("android.app.IActivityManager")
    mInstanceField[activityManagerSingletonInstance] =
      swap(iActivityManagerInterface, activityManagerInstance!!)
  }

  companion object {
    private const val STOP_SERVICE = 116

    private const val METHOD_SERVICE_DONE_EXECUTING = "serviceDoneExecuting"
  }
}
```

总结：通过反射代理  ActivityThread.H 的 Callback ， 监听 STOP_SERVICE 事件获取到 Service 实例对象，再通过反射代理
android.app.IActivityManager 对象监听到  serviceDoneExecuting 方法调用，也就是 Service 生命结束，

#### 5. ViewModelClearedWatcher
```kotlin
internal class ViewModelClearedWatcher(
  storeOwner: ViewModelStoreOwner,
  private val reachabilityWatcher: ReachabilityWatcher
) : ViewModel() {

  private val viewModelMap: Map<String, ViewModel>?

  init {
    viewModelMap = try {
      val mMapField = ViewModelStore::class.java.getDeclaredField("mMap")
      mMapField.isAccessible = true
      @Suppress("UNCHECKED_CAST")
      mMapField[storeOwner.viewModelStore] as Map<String, ViewModel>
    } catch (ignored: Exception) {
      null
    }
  }
  // 当监听到 onCleard 事件，通过反射拿到 ViewModelStore 的所有 ViewModel ,将 ViewModel 添加到泄漏检测 
  override fun onCleared() {
    viewModelMap?.values?.forEach { viewModel ->
      reachabilityWatcher.expectWeaklyReachable(
        viewModel, "${viewModel::class.java.name} received ViewModel#onCleared() callback"
      )
    }
  }

  companion object {
    fun install(
      storeOwner: ViewModelStoreOwner,
      reachabilityWatcher: ReachabilityWatcher
    ) {
      // 为 ViewModelStoreOwner 添加一个ViewModel 用于监听 onCleard 事件
      val provider = ViewModelProvider(storeOwner, object : Factory {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel?> create(modelClass: Class<T>): T =
          ViewModelClearedWatcher(storeOwner, reachabilityWatcher) as T
      })
      provider.get(ViewModelClearedWatcher::class.java)
    }
  }
}
```

总结：因为同一个 ViewModelStoreOwner 下的所有 ViewModel 的 onCleared 调用时机相同，所以可以通过添加一个 ViewModel 的方式监听到同一个 ViewModelStoreOwner 下所有ViewModel.onCleared 的调用，然后通过反射获取到 ViewModelStoreOwner 下的所有 ViewModel 添加到泄漏检测


## 泄漏检测类 ObjectWatcher
从上面5中 Watcher 分析可知当对象生命结束时就会调用 ObjectWatcher.expectWeaklyReachable 方法，将对象添加到泄漏检测

```kotlin
 @Synchronized override fun ObjectWatcher.expectWeaklyReachable(
    watchedObject: Any,
    description: String
  ) {
    // 将 queue 队列中也就是已经被GC回收的对象从观察列表 watchedObjects 中移除
    removeWeaklyReachableObjects()
    // 生成一个 ID 用于关联 queue 和 watchedObjects 中关联的同一对象。方便查找删除

    val key = UUID.randomUUID()
      .toString()

    // 记录起始时间
    val watchUptimeMillis = clock.uptimeMillis()
    // KeyedWeakReference 继承自 WeakReference 只不过添加了watchUptimeMillis 和 key 字段
    val reference =
      KeyedWeakReference(watchedObject, key, description, watchUptimeMillis, queue)
    
    // 将生成的软引用添加到观察列表
    watchedObjects[key] = reference
    // checkRetainedExecutor 默认会延迟5秒执行，延迟5秒后检测对象是否已经被回收
    checkRetainedExecutor.execute {
      moveToRetained(key)
    }
  }

  // 清除已回收对象，如果未被回收记录保留时间，触发 onObjectRetainedListeners 监听，这个监听会间接触发 checkRetainedObjects
  @Synchronized private fun ObjectWatcher.moveToRetained(key: String) {
    removeWeaklyReachableObjects()
    val retainedRef = watchedObjects[key]
    if (retainedRef != null) {
      retainedRef.retainedUptimeMillis = clock.uptimeMillis()
      onObjectRetainedListeners.forEach { it.onObjectRetained() }
    }
  }  
```

```kotlin
 private fun checkRetainedObjects() {
    if (iCanHasHeap is Nope) { //如果这个时候正在 dump 内存，不用再触发 dump 内存
      var retainedReferenceCount = objectWatcher.retainedObjectCount
      // 如果当前有 objectWatcher 有未被回收的对象，主动触发触发 gc ，再查看是否有未回收对象
      if (retainedReferenceCount > 0) {
        gcTrigger.runGc()
        retainedReferenceCount = objectWatcher.retainedObjectCount
      }
      val nopeReason = iCanHasHeap.reason()
      val wouldDump = !checkRetainedCount(
        retainedReferenceCount, config.retainedVisibleThreshold, nopeReason

      // 检查是否需要提醒用户，并不是检测到有未被回收对象就会通知给用户，也有可能这个保留对象已经通知过了
      if (wouldDump) {
        val uppercaseReason = nopeReason[0].toUpperCase() + nopeReason.substring(1)
        onRetainInstanceListener.onEvent(DumpingDisabled(uppercaseReason))
        showRetainedCountNotification(
          objectCount = retainedReferenceCount,
          contentText = uppercaseReason
        )
      }
      return 
    }

    // 如果有未回收对象，触发 GC 查看是否被回收 
    var retainedReferenceCount = objectWatcher.retainedObjectCount

    if (retainedReferenceCount > 0) {
      gcTrigger.runGc()
      retainedReferenceCount = objectWatcher.retainedObjectCount
    }
    // 查看是否发生了变化，如果没变化，代表之前就已经执行过 通知和 dump 操作了
    if (checkRetainedCount(retainedReferenceCount, config.retainedVisibleThreshold)) return

    // 下面就是 执行通知和 dumHeap 操作了
    val now = SystemClock.uptimeMillis()
    val elapsedSinceLastDumpMillis = now - lastHeapDumpUptimeMillis
    if (elapsedSinceLastDumpMillis < WAIT_BETWEEN_HEAP_DUMPS_MILLIS) {
      onRetainInstanceListener.onEvent(DumpHappenedRecently)
      showRetainedCountNotification(
        objectCount = retainedReferenceCount,
        contentText = application.getString(R.string.leak_canary_notification_retained_dump_wait)
      )
      scheduleRetainedObjectCheck(
        delayMillis = WAIT_BETWEEN_HEAP_DUMPS_MILLIS - elapsedSinceLastDumpMillis
      )
      return
    }

    dismissRetainedCountNotification()
    val visibility = if (applicationVisible) "visible" else "not visible"
    dumpHeap(
      retainedReferenceCount = retainedReferenceCount,
      retry = true,
      reason = "$retainedReferenceCount retained objects, app is $visibility"
    )
  }
```

```kotlin
/**
   * Default implementation of [GcTrigger].
   */
  object Default : GcTrigger {
    override fun runGc() {
      // System.gc() 不是每次调用都执行 GC. Runtime.gc() 大概率会执行 GC 但也不是百分百
      Runtime.getRuntime()
        .gc()
      // 调用执行 GC 后，休眠 100 毫秒，然后查看 待回收列表是否还有未回收对象，如果有标记为可能泄漏  
      enqueueReferences()
      System.runFinalization()
    }

    private fun enqueueReferences() {
      try {
        Thread.sleep(100)
      } catch (e: InterruptedException) {
        throw AssertionError()
      }
    }
  }
```



