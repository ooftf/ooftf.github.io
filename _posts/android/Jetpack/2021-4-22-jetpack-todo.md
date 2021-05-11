### 需要接触的Jetpack组件
#### 目的
1. 遵顼最佳做法  
   Android Jetpack 组件采用最新的设计方法构建，具有向后兼容性，可以减少崩溃和内存泄露。
2. 消除样板代码  
   Android Jetpack 可以管理各种繁琐的 Activity（如后台任务、导航和生命周期管理），以便您可以专注于打造出色的应用。
3. 减少不一致  
   这些库可在各种 Android 版本和设备中以一致的方式运作，助您降低复杂性。
#### 组件
* compose   声明式UI
* databinding 在现有的框架下编写声明式UI
* hilt 依赖注入框架
* lifecycle 感知 Activity Fragment Service Application 组件生命周期
* navigation Fragment导航组件，适合用于单一Activity + 多Fragment形式的Activity
* pagging 列表分页加载组件，在分页异步数据读取过程中，展示默认Item，管理调用时机
* room 数据库
* [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager) 调度在退出应用或重启设备后仍应运行的可延期异步任务
* asynclayoutinflater	异步膨胀布局以避免界面出现卡顿。
* cardview	用圆角和阴影实现 Material Design 卡片模式。