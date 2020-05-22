# 问题
* 为什么要考虑移动端跨平台开发
* 为什么选择Flutter
* Flutter如果做到跨平台
* Flutter内部架构
* Flutter对比RN和小程序有什么优缺点
* Flutter对于热更新的支持
* 现有项目如何引入Flutter
* Flutter和本地通讯
* Android控件和Flutter控件的对应关系
* Flutter布局
* Flutter和Android通讯
* Flutter现状（现在和展望）
# Flutter初时
![Flutter架构图](https://user-gold-cdn.xitu.io/2019/9/6/16d04a2dde0b0705?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![Android架构图](https://user-gold-cdn.xitu.io/2019/9/6/16d04a5b5351d574?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![Web架构图](https://user-gold-cdn.xitu.io/2019/9/6/16d04a74e96060ac?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![React Native](https://user-gold-cdn.xitu.io/2018/7/2/1645819e8bbe78f2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
和前端开发不同，react native 所有的标签都不是真实控件，JS代码中所写控件的作用，类似 Map 中的 key 值。JS端通过这个 key 组合的 Dom ，最后Native端会解析这个 Dom ，得到对应的Native控件渲染，如 Android 中<view> 标签对应 ViewGroup 控件。

双方的通讯通过C++层会被转为String字符串传输

Flutter是谷歌2018年发布的跨平台移动UI框架
## Fluttter热更新
webview、rn/weex，都有一个特点，可以远程动态载入js代码，可以更新本地的js代码。前端开发者认为动态性是天经地义的，但其实flutter并不支持。
flutter是有编译优化概念的，如果它提供动态性支持，会影响它的性能。
网上有个方案是：改造flutter，用一个独立的v8/jscore来加载动态js代码，去操作flutter布局引擎的渲染，flutter本来没有跨环境通信的问题，结果又弄了一个js引擎进来搞出了通信问题，造成性能下降，还把包体积增加了很大，还不如直接用rn/weex。
## 暂存区
其他Hot Reload (热重载)
Flutter 采用的是类似 React 的响应式编程模型。UI 在运行时视觉上的变化是由应用的状态来驱动的
## 库介绍

| 库                          | 功能             |
| -------------------------- | -------------- |
| **dio**                    | **网络框架**       |
| **shared_preferences**     | **本地数据缓存**     |
| **fluttertoast**           | **toast**      |
| **flutter_redux**          | **redux**      |
| **device_info**            | **设备信息**       |
| **connectivity**           | **网络链接**       |
| **flutter_markdown**       | **markdown解析** |
| **json_annotation**        | **json模板**     |
| **json_serializable**      | **json模板**     |
| **url_launcher**           | **启动外部浏览器**    |
| **iconfont**               | **字库图标**       |
| **share**                  | **系统分享**       |
| **flutter_spinkit**        | **加载框样式**      |
| **get_version**            | **版本信息**       |
| **flutter_webview_plugin** | **全屏的webview** |
| **sqflite**                | **数据库**        |
| **flutter_statusbar**      | **状态栏**        |
| **flutter_svg**            | **svg**        |
| **photo_view**             | **图片预览**       |
| **flutter_slidable**       | **侧滑**         |
| **flutter_cache_manager**  | **缓存管理**       |
| **path_provider**          | **本地路径**       |
| **permission_handler**     | **权限**         |
| **scope_model**            | **状态管理和共享**    |
| **lottie**                 | **svg动画**    |
| **flare**                  | **路径动画**    |
# 名词解析
* Skia是一个 2D的绘图引擎库，跨平台(支持Android 和 iOS)
# 网站
[Flutter Go 教学帮助的App](https://flutter-go.pub/website/)

[咸鱼Flutter](https://c.tb.cn/I3.ZZpRl)

[移动端跨平台开发深度解析](https://juejin.im/post/5b395eb96fb9a00e556123ef)

[Dart第三方库搜索](https://pub.flutter-io.cn/)

[UI渲染](https://juejin.im/post/5dac428af265da5ba838f476)