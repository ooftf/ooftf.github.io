---
layout: post
author: "ooftf"
tags: [Android,Activity]
---

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

--------------------------------------------------------------------
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

###  Activity 启动流程 API 30

[参考文章](https://blog.csdn.net/u010921373/article/details/109253342)

![Activity 启动流程图](https://github.com/ooftf/ooftf.github.io/blob/master/images/android30_activity_start.png?raw=true)

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
 * ActivityStarter.executeRequest 调用了 ActivityStarter.startActivityUnchecked    
 * ActivityStarter.startActivityUnchecked  调用  ActivityStarter.startActivityInner  
 * ActivityStarter.startActivityInner 调用 RootWindowContainer.resumeFocusedStacksTopActivities()
 * RootWindowContainer.resumeFocusedStacksTopActivities() 调用 ActivityStack.resumeTopActivityUncheckedLocked
 * ActivityStack.resumeTopActivityUncheckedLocked 调用 ActivityStack.resumeTopActivityInnerLocked
 * ActivityStack.resumeTopActivityInnerLocked 调用 ActivityStackSupervisor.startSpecificActivity()
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
* Activity
* Instrumentation
* ActivityTaskManagerService
* A pplicationThread
* ActivityStarter  
  * Controller for interpreting how and then launching an activity.
  * This class collects all the logic for determining how an intent and flags should be turned into an activity and associated task and stack.
  * 如何将一个 Intent 转换为 Activity 并关联到对应的任务栈
* RootWindowContainer
* ActivityStack
* ActivityStackSupervisor
* ClientTransaction
* LaunchActivityItem
* ApplicationThread
* ActivityThread      
* Instrumentation 
* AppComponentFactory   

##### 猜想
从Activity的启动流程可知，有两次 Binder 调用 一次是在 Instrumentation.execStartActivity() 内调用  IActivityTaskManager.startActivity()
另一次是在 ClientTransaction.schedule() 内调用了 IApplicationThread.scheduleTransaction() 。猜想：第一次 Binder 调用应该是跳转到系统服务，第二次调用再跳转回 App 内





