---
layout: post
author: "ooftf"
tags: Android
---

# RemoteView
## 用途
    在其它进程中显示并更新View
    主要用在通知栏和桌面小部件的开发过程中
##  支持的View
1. Layout
   * FrameLayout
   * LinearLayout
   * RelativeLayout
   * GridLayout
2. View
   * AnalogClock
   * Button
   * Chronometer
   * ImageButton
   * ImageView
   * ProgressBar
   * TextView
   * ViewFlipper
   * ListView
   * GridView
   * StackView
   * AdapterViewFlipper
   * ViewStub
---
RemoteViews不支持他们的子类以及其他View类型，也就是说RemoteViews中无法使用除了上述列表中以外的View,也无法使用自定义View
如果我们在RemoteViews中使用EditText那么就会抛出如下异常
```
        E/StatusBar(765): android.view.InflateException: Binary XML file line #25:
        Error inflating class android.widget.EditText
        E/StatusBar(765): Caused by: android.view.InflateException: Binary XML file
        line #25: Class not allowed to be inflated android.widget.EditText
        E/StatusBar(765):      at android.view.LayoutInflater.failNotAllowed
        (LayoutInflater.java:695)
        E/StatusBar(765):      at android.view.LayoutInflater.createView
        (LayoutInflater.java:628)
        E/StatusBar(765):      ... 21 more
```
-----
系统并没有通过Binder去直接支持View的跨进程，而是提供了一个Action的概念，  
Action代表一个View的操作，Action同样实现了Parcelable接口。  
系统首先将View操作封装到Action对象并将这些对象跨进程传输到远程，  
接着在远程进程中通过反射执行Action对象中的具体操作