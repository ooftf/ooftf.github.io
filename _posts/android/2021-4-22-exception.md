#### UnsatisfiedLinkError
```java
   java.lang.UnsatisfiedLinkError: dlopen failed: cannot locate symbol "__emutls_get_address" referenced by "/data/app/~~aY8Bs7K2JnwFQnXfhxaKDA==/com.master.kit-Upy0wIUWX1vsEFKg3UAl8A==/lib/arm/libmmkv.so"...
        at java.lang.Runtime.loadLibrary0(Runtime.java:1087)
        at java.lang.Runtime.loadLibrary0(Runtime.java:1008)
        at java.lang.System.loadLibrary(System.java:1664)
        at com.tencent.mmkv.MMKV.initialize(MMKV.java:110)
        at com.tencent.mmkv.MMKV.initialize(MMKV.java:71)
        at com.ooftf.service.engine.typer.TyperFactory.init(TyperFactory.java:16)
        at com.ooftf.service.base.BaseApplication.onCreate(BaseApplication.kt:57)
        at com.master.kit.application.App.onCreate(App.java:31)
        at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1192)
        at android.app.ActivityThread.handleBindApplication(ActivityThread.java:6861)
        at android.app.ActivityThread.access$1400(ActivityThread.java:246)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1946)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:233)
        at android.app.ActivityThread.main(ActivityThread.java:7892)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:656)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:967)
```
#### SHA-256 digest error
```
SHA-256 digest error for org/bouncycastle/cert/AttributeCertificateHolder.class
```
org.bouncycastle:bcpkix-jdk15on:1.59

这种错误一般是因为AMS框架修改字节码导致的

#### ClassNotFoundException
```java
 Caused by: java.lang.ClassNotFoundException: Didn't find class "com.ooftf.service.base.BaseApplication" on path: DexPathList[[zip file "/data/app/~~9DN3HTtIOUN8T63LXWElbg==/com.master.kit-0T-Q11wnyQNjkvA1LmV6JQ==/base.apk"],nativeLibraryDirectories=[/data/app/~~9DN3HTtIOUN8T63LXWElbg==/com.master.kit-0T-Q11wnyQNjkvA1LmV6JQ==/lib/arm, /data/app/~~9DN3HTtIOUN8T63LXWElbg==/com.master.kit-0T-Q11wnyQNjkvA1LmV6JQ==/base.apk!/lib/armeabi-v7a, /system/lib, /system_ext/lib]]
        at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:207)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:379)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:312)
```
* 如果是第三方库的类  
有可能是因为库的版本升级导致
比如，使用了A库，A库使用了B库，同时C库也使用了B库，但是C库使用的B库版本比较高，就会导致最终使用的B库是高版本的，但是B库高版本比低版本少了某个类，这个类刚好被A库本使用
* 是自己编写的类
有可能是Transform API 导致转移Jar 出错导致的    

#### java.util.ConcurrentModificationException
```
1 java.util.ArrayList$Itr.next(ArrayList.java:860)
2 com.google.gson.internal.bind.CollectionTypeAdapterFactory$Adapter.write(CollectionTypeAdapterFactory.java:96)
3 com.google.gson.internal.bind.CollectionTypeAdapterFactory$Adapter.write(CollectionTypeAdapterFactory.java:61)
4 com.google.gson.Gson.toJson(Gson.java:704)
5 com.google.gson.Gson.toJson(Gson.java:683)
6 com.google.gson.Gson.toJson(Gson.java:638)
7 com.google.gson.Gson.toJson(Gson.java:618)
```
这种错误大概率是因为，在进行JSON转换的时候，还在修改List，大多是因为并发导致，可以使用线程安全的类，比如 CopyOnWriteArrayList

####   compatible with Java 11
```
   > Could not resolve com.github.ooftf:autoregister:1.4.3.
     Required by:
         project :
      > No matching variant of com.github.ooftf:autoregister:1.4.3 was found. The consumer was configured to find a runtime of a library compatible with Java 8, packaged as a jar, and its dependencies declared externally but:
          - Variant 'apiElements' capability com.github.ooftf:autoregister:1.4.3 declares a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares an API of a component compatible with Java 11 and the consumer needed a runtime of a component compatible with Java 8
          - Variant 'runtimeElements' capability com.github.ooftf:autoregister:1.4.3 declares a runtime of a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares a component compatible with Java 11 and the consumer needed a component compatible with Java 8
```
* 原因：引用库(一般是插件)要求Java 11 版本，但是当前项目是Java 8
* 解决方式
  1. 将引用库的要求版本版本改为 Java 8
     ```groovy
     sourceCompatibility = "8"
     targetCompatibility = "8"
     ```
  2. 将当前项目改为Java 11

## 你的主机中的软件中止了一个已建立的连接。
* 关闭电脑wifi热点

### plugin incompatible
plugin "ASM Bytecode Outline" is incompatible(supported only in intellij IDEA)
* 解决方式
将 C:/Users/{username}/AppData/Roaming/Google/AndroidStudio{version}/plugins 的 ASM-BO 文件夹删除

### Cannot fit requested classes in a single dex file (# methods: 65919 > 65536)
```groovy
android {
    defaultConfig {
       multiDexEnabled true
    }
}
```

### android.app.RemoteServiceException
```
android.app.RemoteServiceException
can't deliver broadcast
android.app.ActivityThread$H.handleMessage(ActivityThread.java:2171)
```

### dokit 使用 gradle-7.0.1 报错
Exception:
```
Querying the mapped value of property(org.gradle.api.file.RegularFile, property(org.gradle.api.file.RegularFile, property(org.gradle.api.file.RegularFile, map(org.gradle.api.file.RegularFile flatmap(provider(task 'processDebugMainManifest', class com.android.build.gradle.tasks.ProcessApplicationManifest)) check-type())))) before task ':app:processDebugMainManifest' has completed is not supported

```
解决方案：

修改 gradle 为 7 一下就可以


## reportSizeConfigurations: ActivityRecord not found for: Token{757a5d3 null}

[bugly 地址](https://bugly.qq.com/v2/crash-reporting/crashes/b26659df48/266312?pid=1)

[stackoverflow 相关讨论](https://stackoverflow.com/questions/46309428/android-activitythread-reportsizeconfigurations-causes-app-to-freeze-with-black)

[可能有用的解决方案](https://my.oschina.net/dtqq/blog/4512858)

[google 问题讨论](https://issuetracker.google.com/issues/62427912)


## Explicit termination method 'destroy' not called
```
java.lang.Throwable: Explicit termination method 'destroy' not called
	at dalvik.system.CloseGuard.open(CloseGuard.java:237)
```
这个错误长发生在 io 流没有 close，有些对象没有 destroy 之类的关闭操作。这些关闭动作需要放到 fianlly 才比较安全。