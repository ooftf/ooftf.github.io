## [startup](https://developer.android.google.cn/topic/libraries/app-startup?hl=zh-cn)
应用程序和库通常依赖于在应用程序启动时立即初始化组件。您可以通过使用 ContentProviders 来初始化每个依赖项来满足此需求，但是内容提供程序的实例化成本很高，并且可能会不必要地减慢启动顺序。此外，Android 以不确定的顺序初始化ContentProviders。 App Startup 提供了一种在应用程序启动时初始化组件并显式定义它们的依赖项的更高效的方法。
## 使用指南
```groovy
dependencies {
    implementation "androidx.startup:startup-runtime:1.0.0"
}
```
#### 自动初始化
```kotlin
// Initializes WorkManager.
class WorkManagerInitializer : Initializer<WorkManager> {
    override fun create(context: Context): WorkManager {
        val configuration = Configuration.Builder().build()
        WorkManager.initialize(context, configuration)
        return WorkManager.getInstance(context)
    }
    override fun dependencies(): List<Class<out Initializer<*>>> {
        // No dependencies on other libraries.
        return emptyList()
    }
}

// Initializes ExampleLogger.
class ExampleLoggerInitializer : Initializer<ExampleLogger> {
    override fun create(context: Context): ExampleLogger {
        // WorkManager.getInstance() is non-null only after
        // WorkManager is initialized.
        return ExampleLogger(WorkManager.getInstance(context))
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {
        // Defines a dependency on WorkManagerInitializer so it can be
        // initialized after WorkManager is initialized.
        return listOf(WorkManagerInitializer::class.java)
    }
}


```
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes ExampleLoggerInitializer discoverable. -->
    <meta-data  android:name="com.example.ExampleLoggerInitializer"
          android:value="androidx.startup" />
</provider>
```

#### 手动初始化组件

首先为要手动初始化的组件禁用自动初始化。

```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <meta-data android:name="com.example.ExampleLoggerInitializer"
              tools:node="remove" />
</provider>
```
禁用所有组件自动初始化。
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    tools:node="remove" />
```
手动初始化组件
```java
AppInitializer.getInstance(context)
    .initializeComponent(ExampleLoggerInitializer::class.java)
```
