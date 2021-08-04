---
layout: post
author: "ooftf"
top: true
tags: [Android,Activity]
---

## [Activity](https://developer.android.google.cn/guide/components/activities/intro-activities?hl=zh_cn)
![生命周期](https://raw.githubusercontent.com/ooftf/Material/master/img/blogactivity_lifecycle.png)
## 关于Application.activityLifecycleCallbacks.onCreate  Lifecycle.onCreate 和 MyAcitivity.onCreate 的调用顺序

查看 Activity.performCreate 方法
```java
final void performCreate(Bundle icicle, PersistableBundle persistentState) {
    dispatchActivityPreCreated(icicle);
    if (persistentState != null) {
        onCreate(icicle, persistentState);
    } else {
        onCreate(icicle);
    }
    /*
     * 从这里可知执行完 onCreate 方法又执行的 mFragments.dispatchActivityCreated()
     * 而 Activity Lifecycle 的实现是通过 ReportFragment 监听 onActivityCreate 方法
     * 所以mFragments.dispatchActivityCreated() 调用了 Activity Lifecycle.onCreate 方法
     * 因此结论是，onCreate 方法调用完成，才执行 Lifecycle.onCreate
     */
    
    mFragments.dispatchActivityCreated();
    dispatchActivityPostCreated(icicle);
}
```
查看 Activity.onCreate 方法
```java
@MainThread
@CallSuper
protected void Activity.onCreate(@Nullable Bundle savedInstanceState) {
    dispatchActivityCreated(savedInstanceState);
}
```
查看 Activity.dispatchActivityCreated 方法
```java
private void dispatchActivityCreated(@Nullable Bundle savedInstanceState) {
    // 这里将 onCreate 方法通知给 Application.activityLifecycleCallbacks.onCreate 
    getApplication().dispatchActivityCreated(this, savedInstanceState);
}
```

由上面分析可知 onCreate 的执行顺序为
```java
Activity.performCreate{
    MyAcitivity.onCreate{
        Activity.onCreate{
            Application.activityLifecycleCallbacks.onCreate
        }
         // doSoming
    }
    Lifecycle.onCreate
}

```

## Activity 视图层级结构

![Activity 视图层](https://raw.githubusercontent.com/ooftf/Material/master/img/blogAndroidViewLayer.png)

#### Android 10 1440*3200 小米手机 展示 NavigationBar 状态下

|名字|坐标|描述|
|--|--|--|
|ContentFrameLayout|[0,333][1440,3048]|android.R.id.content|
|ActionBarOverlayLayout|[0,137][1440,3048]|ActionBar 主题|
|FitWindowsLinearLayout|[0,137][1440,3048]|NoActionBar 主题|
|FrameLayout|[0,137][1440,3048]||
|View|[0,3048][1440,3200]|与 LinearLayout 同级都是 DecorView 的 children ,android.R.id.navigationBarBackground|
|LinearLayout|[0,0]  [1440,3048]||
|DecorView|[0,0]  [1440,3200]||
|android.view.ViewRootImpl|||

window.decorView.getWindowVisibleDisplayFrame(outRect) 获取到的是 去除 StatusBar、NavigationBar、键盘 等系统界面之后的区域
#### 启动模式
1. standard
2. singleTop
3. singleTask
4. singleInstance
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

* SingleTask或singleInstance 如果在A启动A    虽然不会走大部分生命周期，但是会导致本身和位于其中的Fragment走onResume方法

```
    [HomeActivity, onActivityPaused]
    [HomeFragment, onFragmentPaused]
    [SupportRequestManagerFragment, onFragmentPaused]
    [HomeActivity, onActivityResumed]
    [HomeFragment, onFragmentResumed]
    [SupportRequestManagerFragment, onFragmentResumed]
```

### startActivityForResult
A 通过 startActivityForResult 启动B ，B 启动 C ，在C展示过程中关闭 B 并不会回调 A 的 onActivityResult   
onActivityResult 当 A 回到前台的时候才会回调
###  Activity 启动流程 API 30

[参考文章](https://blog.csdn.net/u010921373/article/details/109253342)

![Activity 启动流程图](https://raw.githubusercontent.com/ooftf/Material/master/img/blogandroid30_activity_start.png)

* Activity.startActivity() 调用了 Activity.startActivityForResult()
* Activity.startActivityForResult() 调用了 Instrumentation.execStartActivity()
* Instrumentation.execStartActivity() 调用了 ActivityTaskManager.getService().startActivity()
* ActivityTaskManager.getService().startActivity()  
  ActivityTaskManager.getService() 获取到的是 IActivityTaskManager 的一个单例
  IActivityTaskManager 是一个Binder类型的对象 创建方式如下
  
  ```java
  final IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
                      return IActivityTaskManager.Stub.asInterface(b);
  ```
 * IActivityTaskManager 的具体实现为 ActivityTaskManagerService
 * ActivityTaskManagerService.startActivity() 调用 ActivityTaskManagerService.startActivityAsUser
 * ActivityTaskManagerService.startActivityAsUser() 调用 ActivityStarter.execute()   
 * ActivityStarter.execute() 调用 ActivityStarter.executeRequest  
 * ActivityStarter.executeRequest 调用 ActivityStarter.startActivityUnchecked    
 * ActivityStarter.startActivityUnchecked  调用  ActivityStarter.startActivityInner  
 * ActivityStarter.startActivityInner 调用 RootWindowContainer.resumeFocusedStacksTopActivities()
 * RootWindowContainer.resumeFocusedStacksTopActivities() 调用 ActivityStack.resumeTopActivityUncheckedLocked
 * ActivityStack.resumeTopActivityUncheckedLocked 调用 ActivityStack.resumeTopActivityInnerLocked
 * ActivityStack.resumeTopActivityInnerLocked 调用 ActivityStackSupervisor.startSpecificActivity()
   ```java
       void startSpecificActivity(ActivityRecord r, boolean andResume, boolean checkConfig) {
        // Is this activity's application already running?
        final WindowProcessController wpc =
                mService.getProcessController(r.processName, r.info.applicationInfo.uid);

        boolean knownToBeDead = false;
        if (wpc != null && wpc.hasThread()) {
            try {
                realStartActivityLocked(r, wpc, andResume, checkConfig);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Exception when starting activity "
                        + r.intent.getComponent().flattenToShortString(), e);
            }

            // If a dead object exception was thrown -- fall through to
            // restart the application.
            knownToBeDead = true;
        }

        r.notifyUnknownVisibilityLaunchedForKeyguardTransition();

        final boolean isTop = andResume && r.isTopRunningActivity();
        mService.startProcessAsync(r, knownToBeDead, isTop, isTop ? "top-activity" : "activity");
    }

   ```
   检查目标线程是否存在，如果不存在调用 ActivityTaskManagerService.startProcessAsync() 启动进程
 * ActivityStackSupervisor.startSpecificActivity() 调用 ActivityStackSupervisor.realStartActivityLocked
   ```java
    boolean realStartActivityLocked(ActivityRecord r, WindowProcessController proc,
              boolean andResume, boolean checkConfig) throws RemoteException {
                ...
                  // Create activity launch transaction.
                  final ClientTransaction clientTransaction = ClientTransaction.obtain(
                          proc.getThread(), r.appToken);
  
                  final DisplayContent dc = r.getDisplay().mDisplayContent;
                  clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
                          System.identityHashCode(r), r.info,
                          // TODO: Have this take the merged configuration instead of separate global
                          // and override configs.
                          mergedConfiguration.getGlobalConfiguration(),
                          mergedConfiguration.getOverrideConfiguration(), r.compat,
                          r.launchedFromPackage, task.voiceInteractor, proc.getReportedProcState(),
                          r.getSavedState(), r.getPersistentSavedState(), results, newIntents,
                          dc.isNextTransitionForward(), proc.createProfilerInfoIfNeeded(),
                          r.assistToken, r.createFixedRotationAdjustmentsIfNeeded()));
  
                  // Set desired final state.
                  final ActivityLifecycleItem lifecycleItem;
                  if (andResume) {
                      lifecycleItem = ResumeActivityItem.obtain(dc.isNextTransitionForward());
                  } else {
                      lifecycleItem = PauseActivityItem.obtain();
                  }
                  clientTransaction.setLifecycleStateRequest(lifecycleItem);
  
                  // Schedule transaction.
                  mService.getLifecycleManager().scheduleTransaction(clientTransaction);
                ...
              }
  
   ```
  mService.getLifecycleManager().scheduleTransaction(clientTransaction); 向 ClientLifecycleManager 添加了 ClientTransaction 计划任务，ClientTransaction 又添加了LaunchActivityItem 作为 CallBack 所以最终会调用  LaunchActivityItem.execute
* clientTransaction 是如何被调用的呢
  * ClientLifecycleManager.scheduleTransaction() 调用了 ClientTransaction.schedule()
  * ClientTransaction.schedule() 调用了 IApplicationThread.scheduleTransaction()
  * 已知 IApplicationThread 的实现类是 ApplicationThread ；查看 ApplicationThread.scheduleTransaction()
  * ApplicationThread.scheduleTransaction() 调用 ActivityThread.scheduleTransaction
    ```java
     void scheduleTransaction(ClientTransaction transaction) {
          transaction.preExecute(this);
          sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
      }
    ```
  * ClientTransactionHandler.sendMessage 的实现是 ActivityThread.sendMessage
    ```java
     private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
           if (DEBUG_MESSAGES) {
               Slog.v(TAG,
                       "SCHEDULE " + what + " " + mH.codeToString(what) + ": " + arg1 + " / " + obj);
           }
           Message msg = Message.obtain();
           msg.what = what;
           msg.obj = obj;
           msg.arg1 = arg1;
           msg.arg2 = arg2;
           if (async) {
               msg.setAsynchronous(true);
           }
           mH.sendMessage(msg);
       }
    ```
  * 接受 Message 是在 ActivityThread.H.handleMessage 内
    ```java
    case EXECUTE_TRANSACTION:
                       final ClientTransaction transaction = (ClientTransaction) msg.obj;
                       mTransactionExecutor.execute(transaction);
                       if (isSystem()) {
                           // Client transactions inside system process are recycled on the client side
                           // instead of ClientLifecycleManager to avoid being cleared before this
                           // message is handled.
                           transaction.recycle();
                       }
                       // TODO(lifecycler): Recycle locally scheduled transactions.
                       break;
    ```
* LaunchActivityItem.execute 调用 ClientTransactionHandler.handleLaunchActivity
* ClientTransactionHandler 的实现类是 ActivityThread 所以最终走向了 ActivityThread.handleLaunchActivity
* ActivityThread.performLaunchActivity
  ```java
   /**  Core implementation of activity launch. */
      private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
          ...
          ContextImpl appContext = createBaseContextForActivity(r);
          Activity activity = null;
          try {
              java.lang.ClassLoader cl = appContext.getClassLoader();
              activity = mInstrumentation.newActivity(
                      cl, component.getClassName(), r.intent);
              StrictMode.incrementExpectedActivityCount(activity.getClass());
              r.intent.setExtrasClassLoader(cl);
              r.intent.prepareToEnterProcess();
              if (r.state != null) {
                  r.state.setClassLoader(cl);
              }
          } catch (Exception e) {
              if (!mInstrumentation.onException(activity, e)) {
                  throw new RuntimeException(
                      "Unable to instantiate activity " + component
                      + ": " + e.toString(), e);
              }
          }
  
          ...
          Application app = r.packageInfo.makeApplication(false, mInstrumentation);
          ...
          if (activity != null) {
             
              if (r.isPersistable()) {
                  mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
              } else {
                  mInstrumentation.callActivityOnCreate(activity, r.state);
              }
              
          }
          ...
          r.activity = activity;
          ...
          r.setState(ON_CREATE)
          // updatePendingActivityConfiguration() reads from mActivities to update
          // ActivityClientRecord which runs in a different thread. Protect modifications to
          // mActivities to avoid race.
          synchronized (mResourcesManager) {
              mActivities.put(r.token, r);
          }
          ...
          return activity;
      }
  ```
* Activity 的实例是由 Instrumentation.newActivity 创建的
* Instrumentation.newActivity 调用了 AppComonentFactory.instantiateActivity
* AppComponentFactory.instantiateActivity
  ```java
    public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
              @Nullable Intent intent)
              throws InstantiationException, IllegalAccessException, ClassNotFoundException {
          return (Activity) cl.loadClass(className).newInstance();
      }
  ```

#### 创建 Activity 类的调用顺序
* Activity.startActivity() ->startActivityForResult()
* Instrumentation.execStartActivity()
* ActivityTaskManager.getService().startActivity
* ActivityTaskManagerService.startActivity ->  startActivityAsUser()
  * 系统服务
* ActivityStarter.execute() -> startActivityInner()
  * Controller for interpreting how and then launching an activity.
  * This class collects all the logic for determining how an intent and flags should be turned into an activity and associated task and stack.
  * 将一个 Intent 转换为 Activity 并关联到对应的任务栈
  * 通过 PMS.resolveIntent 获取到 Intent 对应的具体信息
  * 检查开启权限 
* RootWindowContainer.resumeFocusedStacksTopActivities()
* ActivityStack.resumeTopActivityUncheckedLocked ->   resumeTopActivityInnerLocked()
  应该就是我们平常所说的 Activity 栈
* ActivityStackSupervisor.startSpecificActivity() -> realStartActivityLocked()
* ClientLifecycleManager.scheduleTransaction()
* ClientTransaction.schedule()
* IApplicationThread.scheduleTransaction()
* ApplicationThread.scheduleTransaction()  
  用于和 AMS 交互
* ActivityThread.scheduleTransaction()      
* ActivityThread.H.handleMessage
* ClientTransaction.getCallbacks().forEach->execute()
* LaunchActivityItem.excute 
  * LaunchActivityItem 就是一个ClientTransaction的 callback ClientTransactionItem 的一个实例
* ActivityThread.handleLaunchActivity() -> performLaunchActivity()
* Instrumentation.newActivity()
* AppComponentFactory.instantiateActivity()   

##### 两次跨进程

从Activity的启动流程可知，有两次 Binder 调用 一次是在 Instrumentation.execStartActivity() 内调用  IActivityTaskManager.startActivity()
另一次是在 ClientTransaction.schedule() 内调用了 IApplicationThread.scheduleTransaction() 。猜想：第一次 Binder 调用应该是跳转到系统服务，第二次调用再跳转回 App 内


## ActivityThraed.performLaunchActivity
```java
    /**  Core implementation of activity launch. */
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

        Activity activity = null;
        activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
        Application app = r.packageInfo.makeApplication(false, mInstrumentation);
        activity.attach(appContext, this, getInstrumentation(), r.token,
                              r.ident, app, r.intent, r.activityInfo, title, r.parent,
                              r.embeddedID, r.lastNonConfigurationInstances, config,
                              r.referrer, r.voiceInteractor, window, r.configCallback,
                              r.assistToken);
        if (r.isPersistable()) {
            mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
        } else {
             mInstrumentation.callActivityOnCreate(activity, r.state);
        }
           
        return activity;
    }
```
这个方法一共做了四件比较重要的事按照顺序分别是
1. 创建 Activity
2. 如果 Application 还没有创建（这个时候，已经创建好了），创建 Application 并执行Application.onCreate 方法
3. 调用 Activity.attach 方法
4. 调用 Activity.onCreate 方法

## Activity window view ViewRootImpl WindowManagerImpl

Activity 在 attach 方法中中创建并添加 window(PhoneWindow)
window 在setContentView 时创建并添加 DecorView 并将 contentView添加到 DecorView 中

# 疑问
* Window 什么时候创建的
* WindowManger 什么时候创建的
* DecorView 什么时候创建的
* ViewRootImpl 什么时候创建的

#### Activity.attach

ActivityThread.performLaunchActivity->Activity attach 方法，创建PhoneWindow，为PhoneWindow 创建 WindowManagerImpl
```java
//Activity
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
        attachBaseContext(context);
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
    }
```
```java
  //Window
  public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
            boolean hardwareAccelerated) {
        mAppToken = appToken;
        mAppName = appName;
        mHardwareAccelerated = hardwareAccelerated;
        if (wm == null) {
            wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        }
        mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    }
```
```java
    //WindowManagerImpl
    public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
        return new WindowManagerImpl(mContext, parentWindow);
    }
     private WindowManagerImpl(Context context, Window parentWindow) {
        mContext = context;
        mParentWindow = parentWindow;
    }
```
#### Activity.setContentView
Activity.setContentView  调用 Window.setContentView
```java
// Window
    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }
      mLayoutInflater.inflate(layoutResID, mContentParent);
    }

    private void installDecor() {
       if (mDecor == null) {
           mDecor = generateDecor(-1);
       } else {
           mDecor.setWindow(this);
       }
       if (mContentParent == null) {
           mContentParent = generateLayout(mDecor);
       }
    }

     protected ViewGroup generateLayout(DecorView decor) {
        mDecor.startChanging();
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        mDecor.finishChanging()
        return contentParent;
    }
    
```

#### Activity.makeVisible
```java
    void makeVisible() {
        if (!mWindowAdded) {
            ViewManager wm = getWindowManager();
            wm.addView(mDecor, getWindow().getAttributes());
            mWindowAdded = true;
        }
        mDecor.setVisibility(View.VISIBLE);
    }
```

## ViewRootImpl
有很多布局相关的事件，都需要从根布局开始操作，而 ViewRootImpl 就是用来做这些事情的
```java
    //WindowManagerGlobal
    public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
      final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
      ViewRootImpl root;
      root = new ViewRootImpl(view.getContext(), display);
      root.setView(view, wparams, panelParentView, userId);
  
    }
```
Activity 视图渲染调用堆栈
```
ActivityThread.handleResunmeActivity()
    Activity.makeVisible 
        WindwoManagerImpl.addView
            WindowManagerGlobal.addView // #创建ViewRootImpl# 
                ViewRootImpl.setView 
                    ViewRootImpl.requestLayout() 
                        scheduleTraversals() 
                            mHandler.getLooper().getQueue().postSyncBarrier()
                            Choreographer.postCallback(TraversalRunnable)
                    //  WindowlessWindowManager 是一个 Bindler 这是一个跨进程操作，调用系统服务
                    WindowlessWindowManager.addToDisplayAsUser()
                        WindowlessWindowManager.addToDisplay
                      

TraversalRunnable.run 
    ViewRootImpl.doTraversal
        mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier); 
        ViewRootImpl.performTraversals
            DecoerView.dispatchAttachedToWindow
            ViewRootImpl.performMeasure
                View.measure
                    View.onMeasure
            ViewRootImpl.performLayout
                View.layout
                    View.onLayout
            ViewRootImpl.performDraw
                ViewRootImpl.draw
                    ThreadedRenderer.draw
                        ThreadedRenderer.updateRootDisplayList
                            ThreadedRenderer.updateViewTreeDisplayList
                                view.updateDisplayListIfDirty
                                    RecordingCanvas canvas = renderNode.beginRecording(width, height);
                                    View.draw(canvas);

```
## Activity 属性
### 如何在 Recent Screen 显示多个 
启动Activity的时候添加 Flag Intent.FLAG_ACTIVITY_NEW_DOCUMENT 会在任务列表中单独显示
```kotlin
startActivity(Intent(this,MainActivity2::class.java).apply {addFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT)})
```
### 如何不显示在 Recent Screen
```xml
<activity
    android:name=".MainActivity"
    android:excludeFromRecents="true" />
```
* 在 Android 系统中，如果我们不想某个 Activity 出现在 “Recent screens” 中，可以设置 <activity> 属性 android:excludeFromRecents 为 true。
* android:excludeFromRecents 属性并不会仅仅影响被设置的 Activity。由此该 Activity 启动的后续同属一个 “Task” 的一系列 Activity 都不会出现在 Recent screens。
也就是说该属性是对 Task 起作用的，而不仅仅是某个 Activity。所以想要后续的 Activity 能够出现在 Recent screens 中，就必须让后续 Activity 在新的 Task 中。
* 如果设置上面属性的 Activity 正是当前正在使用的，切换到 Recent screens 也是可以看到的。但是退到后台运行后，比如按下 Home 键，属性就会发生作用。

### taskAffinity

## 问答

### Android onSaveInstanceState()和onRestoreInstanceState() 调用时机
在没有 finish  Activity 的时候，onStart 被调用前就会调用 onSaveInstanceState
例如
1. 当用户按下HOME键时。
2. 从最近应用中选择运行其他的程序时。
3. 按下电源按键（关闭屏幕显示）时。
4. 从当前activity启动一个新的activity时。
5. 屏幕方向切换时(无论竖屏切横屏还是横屏切竖屏都会调用)。



### Activity A跳转Activity B，再按返回键，生命周期执行的顺序
当A跳转到B的时候

A先执行onPause，然后居然是B再执行onCreate -> onStart -> onResume，最后才执行A的onStop

当B按下返回键

B先执行onPause，然后居然是A再执行onRestart -> onStart -> onResume，最后才是B执行onStop  -> onDestroy!!!
