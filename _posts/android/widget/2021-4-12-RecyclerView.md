---
layout: post
author: "ooftf"
tags: Android
---

### ScrollView嵌套RecyclerView自动滚动
    相关文章https://blog.csdn.net/beibaokongming/article/details/79581232
- 解决方案一： ScrollView的子viewGroup中增加android:descendantFocusability="blocksDescendants" 属性
- 解决方案二： ScrollView的直接子view 上添加focusableInTouchMode=true
### ScrollView 嵌套RecyclerView + GridLayoutManager 只显示一行或者不显示，需要滑动RecyclerView
    将ScrollView换位NestScrollView
### RecyclerView 中EditText singleLine模式下，作为最后一个Item，输入换行崩溃，某些机型（华为）
```
java.lang.IllegalStateException: focus search returned a view that wasn't able to take focus!
    at android.widget.TextView.onKeyUp(TextView.java:8643)
    at android.widget.EditText.onKeyUp(EditText.java:289)
    at android.view.KeyEvent.dispatch(KeyEvent.java:3094)
    at android.view.View.dispatchKeyEvent(View.java:13537)
    at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1973)
    at com.ooftf.layout.kv.KvLayout.dispatchKeyEvent(KvLayout.kt:423)
    at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1973)
    at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1973)
    at com.android.internal.policy.DecorView.superDispatchKeyEvent(DecorView.java:668)
    at com.android.internal.policy.PhoneWindow.superDispatchKeyEvent(PhoneWindow.java:1993)
    at android.app.Activity.dispatchKeyEvent(Activity.java:4107)
    at androidx.core.app.ComponentActivity.superDispatchKeyEvent(ComponentActivity.java:122)
    at androidx.core.view.KeyEventDispatcher.dispatchKeyEvent(KeyEventDispatcher.java:84)
    at androidx.core.app.ComponentActivity.dispatchKeyEvent(ComponentActivity.java:140)
    at androidx.appcompat.app.AppCompatActivity.dispatchKeyEvent(AppCompatActivity.java:569)
    at androidx.appcompat.view.WindowCallbackWrapper.dispatchKeyEvent(WindowCallbackWrapper.java:59)
    at androidx.appcompat.app.AppCompatDelegateImpl$AppCompatWindowCallback.dispatchKeyEvent(AppCompatDelegateImpl.java
    at com.baidu.mobstat.ak.dispatchKeyEvent(SourceFile:50)
    at com.baidu.mobstat.ak.dispatchKeyEvent(SourceFile:50)
    at com.android.internal.policy.DecorView.dispatchKeyEvent(DecorView.java:516)
    at android.view.ViewRootImpl$ViewPostImeInputStage.processKeyEvent(ViewRootImpl.java:6038)
    at android.view.ViewRootImpl$ViewPostImeInputStage.onProcess(ViewRootImpl.java:5906)
    at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:5350)
    at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:5403)
    at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:5369)
    at android.view.ViewRootImpl$AsyncInputStage.forward(ViewRootImpl.java:5527)
    at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:5377)
    at android.view.ViewRootImpl$AsyncInputStage.apply(ViewRootImpl.java:5584)
    at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:5350)
    at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:5403)
    at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:5369)
    at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:5377)
    at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:5350)
    at android.view.ViewRootImpl.deliverInputEvent(ViewRootImpl.java:8325)
    at android.view.ViewRootImpl.doProcessInputEvents(ViewRootImpl.java:8245)
    at android.view.ViewRootImpl.enqueueInputEvent(ViewRootImpl.java:8198)
    at android.view.ViewRootImpl$ViewRootHandler.handleMessage(ViewRootImpl.java:5111)
    at android.os.Handler.dispatchMessage(Handler.java:110)
    at android.os.Looper.loop(Looper.java:219)
    at android.app.ActivityThread.main(ActivityThread.java:8387)
    at java.lang.reflect.Method.invoke(Native Method)
    at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:513)
    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1055)
```
* 方案一
```
  override fun dispatchKeyEvent(event: KeyEvent?): Boolean {
        //解决EditText作为RecyclerView的最后一行时，输入换行，在某些机型（华为）崩溃的问题
        try {
            return super.dispatchKeyEvent(event)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return true

    }
```
* 方案二
   把EditText的android:imeOptions="actionNone" 即可
### RecyclerView clear再添加数据，界面有闪烁
    是因为RecyclerView有默认动画， itemAnimator = null 可去除默认动画

