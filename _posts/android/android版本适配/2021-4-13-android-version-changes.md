---
layout: post
author: "ooftf"
tags: Android
top: true
---
# [版本变化](https://developer.android.com/about/versions)
### KitKat Android 4.4
* **全屏沉浸模式**
* 用于动画场景的转场框架
* 透明系统 UI 样式
* 新的 Chromium WebView 采用更新版本的 JavaScript引擎 (V8),支持使用 Chrome DevTools进行远程调试
* **Android 4.4 中已引入处于实验阶段的 ART 运行时**

### Lollipop Android 5.0 API 21
* **[Material Design](https://developer.android.com/about/versions/lollipop#Material)**
* **新增悬浮通知**
* 新增的 Job Scheduling API 允许您通过将作业推迟到稍后或指定条件下
* **在 Android 5.0 中，ART 运行时取代 Dalvik 成为平台默认设置**
* NDK64 位系统的支持
* 如果用户尝试安装的应用具有重复自定义权限且签名密钥不同于定义此权限的驻留应用，则系统将阻止安装。

### Marshmallow Android 6.0 API 23
* **运行时权限**  
    此版本引入了一种新的权限模式，如今，用户可直接在运行时管理应用权限。这种模式让用户能够更好地了解和控制权限，同时为应用开发者精简了安装和自动更新过程。用户可为所安装的各个应用分别授予或撤销权限。  对于以 Android 6.0（API 级别 23）或更高版本为目标平台的应用，请务必在运行时检查和请求权限。要确定您的应用是否已被授予权限，请调用新增的 checkSelfPermission() 方法。要请求权限，请调用新增的 requestPermissions() 方法。即使您的应用并不以 Android 6.0（API 级别 23）为目标平台，您也应该在新权限模式下测试您的应用。
* **低电耗模式和应用待机模式**  
   **此版本引入了针对空闲设备和应用的最新节能优化技术。这些功能会影响所有应用，因此请务必在这些新模式下测试您的应用。**
  * **低电耗模式：**
    如果用户拔下设备的电源插头，并在屏幕关闭后的一段时间内使其保持不活动状态，设备会进入低电耗模式，在该模式下设备会尝试让系统保持休眠状态。在该模式下，设备会定期短时间恢复正常工作，以便进行应用同步，还可让系统执行任何挂起的操作。
  * **应用待机模式：**
    应用待机模式允许系统判定应用在用户未主动使用它时处于空闲状态。当用户有一段时间未触摸应用时，系统便会作出此判定。如果拔下了设备电源插头，系统会为其视为空闲的应用停用网络访问以及暂停同步和作业。
* **取消支持 Apache HTTP 客户端**  
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
[官方文档](https://developer.android.com/about/versions/nougat/android-7.0-changes.html)

#### **多窗口支持**
#### SurfaceView
  Android 7.0 可同步移动到 SurfaceView 类，此类在某些情况下提供的电池性能优于 TextureView：在渲染视频或 3D 内容时，包含滚动和动画视频位置的应用在使用 SurfaceView 时比 TextureView 耗电更少。
#### 快速设置
    Android 7.0 还添加了一个新的 API，从而让您可以定义自己的“快速设置”图块，使用户可以轻松访问您应用中的关键控件和操作
    如需了解有关创建应用图块的信息请参阅 android.service.quicksettings.Tile
#### **APK signature scheme v2**
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
#### 广播优化
  面向 Android 7.0 开发的应用不会收到 CONNECTIVITY_ACTION 广播，即使它们已有清单条目来请求接受这些事件的通知。在前台运行的应用如果使用 BroadcastReceiver 请求接收通知，则仍可以在主线程中侦听 CONNECTIVITY_CHANGE。  
  应用无法发送或接收 ACTION_NEW_PICTURE 或 ACTION_NEW_VIDEO 广播。此项优化会影响所有应用，而不仅仅是面向 Android 7.0 的应用。
#### 文件权限更改
为了提高私有文件的安全性，面向 Android 7.0 或更高版本的应用私有目录被限制访问　(0700)。此设置可防止私有文件的元数据泄漏，如它们的大小或存在性。此权限更改有多重副作用：

私有文件的文件权限不应再由所有者放宽，为使用 MODE_WORLD_READABLE 和/或 MODE_WORLD_WRITEABLE 而进行的此类尝试将触发 SecurityException。
注：迄今为止，这种限制尚不能完全执行。应用仍可能使用原生 API 或 File API 来修改它们的私有目录权限。但是，我们强烈反对放宽私有目录的权限。

传递软件包网域外的 file:// URI 可能给接收器留下无法访问的路径。因此，尝试传递 file:// URI 会触发 FileUriExposedException。分享私有文件内容的推荐方法是使用 FileProvider。
DownloadManager 不再按文件名分享私人存储的文件。旧版应用在访问 COLUMN_LOCAL_FILENAME 时可能出现无法访问的路径。面向 Android 7.0 或更高版本的应用在尝试访问 COLUMN_LOCAL_FILENAME 时会触发 SecurityException。通过使用 DownloadManager.Request.setDestinationInExternalFilesDir() 或 DownloadManager.Request.setDestinationInExternalPublicDir() 将下载位置设置为公共位置的旧版应用仍可以访问 COLUMN_LOCAL_FILENAME 中的路径，但是我们强烈反对使用这种方法。对于由 DownloadManager 公开的文件，首选的访问方式是使用ContentResolver.openFileDescriptor()。    

#### 在应用间共享文件
对于面向 Android 7.0 的应用，Android 框架执行的 StrictMode API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException 异常。

要在应用间共享文件，您应发送一项 content:// URI，并授予 URI 临时访问权限。进行此授权的最简单方式是使用 FileProvider 类。如需了解有关权限和共享文件的详细信息，请参阅共享文件。

### Oreo Android 8.0 API 26 (Android 8.1 API 27)
* [通知](https://developer.android.com/guide/topics/ui/notifiers/notifications#ManageChannels)
  * [通知渠道](https://developer.android.com/guide/topics/ui/notifiers/notifications#ManageChannels)  
    通知渠道：Android 8.0 引入了通知渠道，其允许您为要显示的每种通知类型创建用户可自定义的渠道。用户界面将通知渠道称之为通知类别
  * 通知标志  
    Android 8.0 引入了对在应用启动器图标上显示通知标志的支持。通知标志可反映某个应用是否存在与其关联、并且用户尚未予以清除也未对其采取行动的通知。通知标志也称为通知点
  * 休眠
  * 通知超时
  * 通知设置
  * 通知清除
  * 背景颜色
  * 消息样式
* [自动填充框架](https://developer.android.com/guide/topics/text/autofill)  
  帐号创建、登录和信用卡交易需要时间并且容易出错。在使用要求执行此类重复性任务的应用时，用户很容易遭受挫折。
  Android 8.0 通过引入自动填充框架，简化了登录和信用卡表单之类表单的填写工作。在用户选择接受自动填充之后，新老应用都可使用自动填充框架。
* [自动调整 TextView 的大小](https://developer.android.com/guide/topics/ui/look-and-feel/autosizing-textview)  
  Android 8.0 允许您根据 TextView 的大小自动设置文本展开或收缩的大小。这意味着，在不同屏幕上优化文本大小或者优化包含动态内容的文本大小比以往简单多了。如需了解有关如何在 Android 8.0 中自动调整 TextView 的大小的详细信息，请参阅自动调整 TextView 的大小。
* [自适应图标](https://developer.android.com/guide/practices/ui_guidelines/icon_design_adaptive)  
  Android 8.0 引入自适应启动器图标。自适应图标支持视觉效果，可在不同设备型号上显示为各种不同的形状。
* [固定快捷方式和小部件](https://developer.android.com/guide/topics/ui/shortcuts)  
  Android 8.0 引入了快捷方式和微件的应用内固定功能。在您的应用中，您可以根据用户权限为支持的启动器创建固定的快捷方式和小部件。

### Pie Android 9.0 API 28
* **默认不支持 HTTP 明文请求**
* 对使用非 SDK 接口的限制
* 前台服务  
  如果应用以 Android 9 或更高版本为目标平台并使用前台服务，则必须请求 FOREGROUND_SERVICE 权限。这是普通权限，因此，系统会自动为请求权限的应用授予此权限。
  如果以 Android 9 或更高版本为目标平台的应用尝试创建前台服务且未请求 FOREGROUND_SERVICE，则系统会抛出 SecurityException。
* 构建序列号弃用  
  在 Android 9 中，Build.SERIAL 始终设为 "UNKNOWN"，以保护用户隐私。  
  如果您的应用需要访问设备的硬件序列号，您应改为请求 READ_PHONE_STATE 权限，然后调用 getSerial()。

### [Android 10 API 29](https://developer.android.com/about/versions/10/privacy/changes)
* 外部存储访问权限范围限定为应用文件和媒体  
    默认情况下，对于以 Android 10 及更高版本为目标平台的应用，其访问权限范围限定为外部存储，即分区存储。此类应用可以查看外部存储设备内以下类型的文件，无需请求任何与存储相关的用户权限：
    特定于应用的目录中的文件（使用 getExternalFilesDir() 访问）。
    应用创建的照片、视频和音频片段（通过媒体库访问）。
    要详细了解分区存储以及如何共享、访问和修改在外部存储设备上保存的文件，请参阅有关如何管理外部存储设备中的文件以及如何访问和修改媒体文件的指南。
* 在后台运行时访问设备位置信息需要权限
* 为了让用户更好地控制应用对位置信息的访问权限，Android 10 引入了 ACCESS_BACKGROUND_LOCATION 权限。
    与 ACCESS_FINE_LOCATION 和 ACCESS_COARSE_LOCATION 权限不同，ACCESS_BACKGROUND_LOCATION 权限仅会影响应用在后台运行时对位置信息的访问权限。除非符合以下条件之一，否则应用将被视为在后台访问位置信息：

    * 属于该应用的 Activity 可见。
    * 该应用运行的某个前台设备已声明前台服务类型为 location。  
      要声明您的应用中的某个服务的前台服务类型，请将应用的 targetSdkVersion 或 compileSdkVersion 设置为 29 或更高版本。详细了解前台服务如何继续执行用户发起的需要访问位置信息的操作。

    以 Android 9 或更低版本为目标平台时自动授予访问权限
    如果您的应用在 Android 10 或更高版本上运行，但其目标平台是 Android 9（API 级别 28）或更低版本，则该平台具有以下行为：

    * 如果您的应用为 ACCESS_FINE_LOCATION 或 ACCESS_COARSE_LOCATION 声明了 <uses-permission> 元素，则系统会在安装期间自动为 ACCESS_BACKGROUND_LOCATION 添加 <uses-permission> 元素。
    * 如果您的应用请求了 ACCESS_FINE_LOCATION 或 ACCESS_COARSE_LOCATION，系统会自动将 ACCESS_BACKGROUND_LOCATION 添加到请求中。
    在设备升级到 Android 10 后访问
    如果用户向您的应用授予对设备位置信息的访问权限（ACCESS_COARSE_LOCATION 或 ACCESS_FINE_LOCATION），然后将其设备从 Android 9 升级到 Android 10，则系统会自动更新应用已获取的基于位置信息的那组权限。您的应用在设备升级后接收的那组权限取决于应用的目标 SDK 版本及其定义的权限，如下表所示：
* 针对从后台启动 Activity 的限制  
  从 Android 10 开始，系统会增加针对从后台启动 Activity 的限制。此项行为变更有助于最大限度地减少对用户造成的中断，并且可以让用户更好地控制其屏幕上显示的内容。只要您的应用启动 Activity 是因用户互动直接引发的，该应用就极有可能不会受到这些限制的影响。
* 对不可重置的设备标识符实施了限制  
    从 Android 10 开始，应用必须具有 READ_PRIVILEGED_PHONE_STATE 特许权限才能访问设备的不可重置标识符（包含 IMEI 和序列号）。
* 限制了对剪贴板数据的访问权限  
  除非您的应用是默认输入法 (IME) 或是目前处于焦点的应用，否则它无法访问 Android 10 或更高版本平台上的剪贴板数据。

### Android 11
* 隐私设置
  * 单次授权：让用户可以选择授予更多对位置信息、麦克风和摄像头的临时访问权限。
  * 权限对话框的可见性：一再拒绝某项权限表示用户希望“不再询问”。
  * 数据访问审核：深入了解您的应用在何处访问私密数据，无论是在您的应用自己的代码中，还是在依赖库的代码中。
  * 系统提醒窗口权限：根据请求自动向某些类型的应用授予 SYSTEM_ALERT_WINDOW 权限。此外，包含 ACTION_MANAGE_OVERLAY_PERMISSION intent 操作的 intent 始终会将用户转至系统设置中的屏幕。
  * 永久 SIM 卡标识符：在 Android 11 及更高版本中，使用 getIccId() 方法访问不可重置的 ICCID 受到限制。该方法会返回一个非 null 的空字符串。如需唯一标识设备上安装的 SIM 卡，请改用 getSubscriptionId() 方法。订阅 ID 会提供一个索引值（从 1 开始），用于唯一识别已安装的 SIM 卡（包括实体 SIM 卡和电子 SIM 卡）。除非设备恢复出厂设置，否则此标识符的值对于给定 SIM 卡是保持不变的。

### Android 12


### 兼容性问题

#### layer-list 兼容问题
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingDefaultResource">
    <item>
        <color android:color="@color/white">
        </color>
    </item>
    <item
        android:bottom="24dp"
        android:drawable="@mipmap/entrance_launcher_logo"
        android:gravity="bottom|center_horizontal" />
</layer-list>
```
像上面代码 <android:gravity="bottom|center_horizontal"> 这个属性在 Android 5.0 版本中不起作用，这个图片 entrance_launcher_logo 会铺满整个 layer-list，Android 6.0 可以正常显示，在 6.0 以下可以使用 bitmap 达到相同的效果
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingDefaultResource">
    <item>
        <color android:color="@color/white">
        </color>
    </item>
    <item  android:bottom="24dp">
        <bitmap
            android:src="@mipmap/entrance_launcher_logo"
            android:gravity="bottom|center_horizontal" />
    </item>
</layer-list>
```

如果使用的使 drawable 如下，drawable 不推荐使用 layer-list 在 5.0 的手机上无法做到精准布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingDefaultResource">
    <item>
        <color android:color="@color/white">
        </color>
    </item>
    <item
        android:drawable="@drawable/user_top_bg"
        android:gravity="top" />
</layer-list>
```
可以用下面方式兼容，代码如下
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingDefaultResource">
    <item>
        <shape>
            <size
                android:width="400dp"
                android:height="850dp" />
            <solid android:color="@color/white" />
        </shape>
    </item>
    <item android:gravity="top" android:bottom="750dp"  android:drawable="@drawable/user_top_bg">
    </item>
</layer-list>
```