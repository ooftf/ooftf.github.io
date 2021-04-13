# 版本变化
### KitKat Android 4.4
* 全屏沉浸模式
* 用于动画场景的转场框架
* 透明系统 UI 样式
* 新的 Chromium WebView 采用更新版本的 JavaScript引擎 (V8),支持使用 Chrome DevTools进行远程调试
* Android 4.4 中已引入处于实验阶段的 ART 运行时

### Lollipop Android 5.0 API 21
* [Material Design](https://developer.android.com/about/versions/lollipop#Material)
* 新增悬浮通知
* 新增的 Job Scheduling API 允许您通过将作业推迟到稍后或指定条件下
* 在 Android 5.0 中，ART 运行时取代 Dalvik 成为平台默认设置
* NDK64 位系统的支持
* 如果用户尝试安装的应用具有重复自定义权限且签名密钥不同于定义此权限的驻留应用，则系统将阻止安装。

### Marshmallow Android 6.0 API 23
* 运行时权限  
    此版本引入了一种新的权限模式，如今，用户可直接在运行时管理应用权限。这种模式让用户能够更好地了解和控制权限，同时为应用开发者精简了安装和自动更新过程。用户可为所安装的各个应用分别授予或撤销权限。  对于以 Android 6.0（API 级别 23）或更高版本为目标平台的应用，请务必在运行时检查和请求权限。要确定您的应用是否已被授予权限，请调用新增的 checkSelfPermission() 方法。要请求权限，请调用新增的 requestPermissions() 方法。即使您的应用并不以 Android 6.0（API 级别 23）为目标平台，您也应该在新权限模式下测试您的应用。
* 低电耗模式和应用待机模式  
   **此版本引入了针对空闲设备和应用的最新节能优化技术。这些功能会影响所有应用，因此请务必在这些新模式下测试您的应用。**
  * **低电耗模式：**

  如果用户拔下设备的电源插头，并在屏幕关闭后的一段时间内使其保持不活动状态，设备会进入低电耗模式，在该模式下设备会尝试让系统保持休眠状态。在该模式下，设备会定期短时间恢复正常工作，以便进行应用同步，还可让系统执行任何挂起的操作。

  * **应用待机模式：**

  应用待机模式允许系统判定应用在用户未主动使用它时处于空闲状态。当用户有一段时间未触摸应用时，系统便会作出此判定。如果拔下了设备电源插头，系统会为其视为空闲的应用停用网络访问以及暂停同步和作业。
* 取消支持 Apache HTTP 客户端  
    Android 6.0 版移除了对 Apache HTTP 客户端的支持。如果您的应用使用该客户端，并以 Android 2.3（API 级别 9）或更高版本为目标平台，请改用 HttpURLConnection 类。此 API 效率更高，因为它可以通过透明压缩和响应缓存减少网络使用，并可最大限度降低耗电量。要继续使用 Apache HTTP API，您必须先在 build.gradle 文件中声明以下编译时依赖项：
    ```groovy
    android {
        useLibrary 'org.apache.http.legacy'
    }
    ```
* 硬件标识符访问权
为给用户提供更严格的数据保护，从此版本开始，对于使用 WLAN API 和 Bluetooth API 的应用，Android 移除了对设备本地硬件标识符的编程访问权。WifiInfo.getMacAddress() 方法和 BluetoothAdapter.getAddress() 方法现在会返回常量值 02:00:00:00:00:00。

现在，要通过蓝牙和 WLAN 扫描访问附近外部设备的硬件标识符，您的应用必须拥有 ACCESS_FINE_LOCATION 或 ACCESS_COARSE_LOCATION 权限。


### Nougat Android 7 API 24(Android 7.1 API  25)
* 多窗口支持
* SurfaceView

Android 7.0 可同步移动到 SurfaceView 类，此类在某些情况下提供的电池性能优于 TextureView：在渲染视频或 3D 内容时，包含滚动和动画视频位置的应用在使用 SurfaceView 时比 TextureView 耗电更少。

* 快速设置

Android 7.0 还添加了一个新的 API，从而让您可以定义自己的“快速设置”图块，使用户可以轻松访问您应用中的关键控件和操作
如需了解有关创建应用图块的信息请参阅 android.service.quicksettings.Tile

* APK signature scheme v2

Android 7.0 引入一项新的应用签名方案 APK Signature Scheme v2，它能提供更快的应用安装时间和更多针对未授权 APK 文件更改的保护。在默认情况下，Android Studio 2.2 和 Android Plugin for Gradle 2.2 会使用 APK Signature Scheme v2 和传统签名方案来签署您的应用。

虽然我们建议您对您的应用采用 APK Signature Scheme v2，但这项新方案并非强制性的。如果您的应用在使用 APK Signature Scheme v2 时不能正确开发，您可以停用这项新方案。禁用过程会导致 Android Studio 2.2 和 Android Plugin for Gradle 2.2 仅使用传统签名方案来签署您的应用。要仅用传统方案签署，打开模块级 build.gradle 文件，然后将行 v2SigningEnabled false 添加到您的版本签名配置中：
```groovy
  android {
    ...
    defaultConfig { ... }
    signingConfigs {
      release {
        storeFile file("myreleasekey.keystore")
        storePassword "password"
        keyAlias "MyReleaseKey"
        keyPassword "password"
        v2SigningEnabled false
      }
    }
  }
```

### Oreo Android 8.0
* 通知
* 自动填充框架
* 画中画模式
* 自动调整 TextView 的大小
* 自适应图标
* 固定快捷方式和小部件

### Pie
### Android 10
### Android 11
### Android 12