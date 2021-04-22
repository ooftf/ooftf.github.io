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