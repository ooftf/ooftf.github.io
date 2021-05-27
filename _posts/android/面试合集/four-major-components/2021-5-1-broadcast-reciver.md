## 特性
1. 监听特定事件
2. 分为有序广播和无序广播
3. 有序广播可拦截


## 两种注册方式
1. 静态注册  
    * 无序广播
        ```xml
        <receiver
            android:name=".service.MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true"
            <intent-filter>
                <action android:name="com.function.luo.MyBroadcastReceiver" />
            </intent-filter>
        </receiver>
        ```
    * 有序广播
        ```xml
        <receiver
            android:name=".service.MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter android:priority="100">
                <action android:name="com.function.luo.MyBroadcastReceiver" />
            </intent-filter>
        </receiver>
        ```
2. 动态注册  
    动态注册无需在AndroidManifest.xml中声明即可直接使用  
    动态注册的广播和注册的Context的生命周期相关，如果是Activity，那么Activity销毁后也就无法接受广播。如果是Application，只要App存活就能接收到广播
    ```java
        IntentFilter  intentFilter = new IntentFilter();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        NetWorkChangeReceiver netWorkChangeReceiver = new NetWorkChangeReceiver();
        context.registerReceiver(netWorkChangeReceiver,intentFilter);
    ```

## 发送广播
* 发送无序广播
    ```java
      Intent intent = new Intent("com.function.luo.MyBroadcastReceiver");
                     sendBroadcast(intent);
    ```
* 发送有序广播
    ```java
     Intent intent = new Intent("com.function.luo.MyBroadcastReceiver");
                     sendOrderedBroadcast(intent,null);
    ```

## 接受广播
```java
public class MyBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context,"发送标准广播",Toast.LENGTH_LONG).show();
        //拦截广播
        abortBroadcast();
    }
}
```    

## 本地广播
前面我们发送和接受的广播全部属于系统全局广播，即发出的广播可以被其它任何应用程序接收到，并且我们也可以接受来自于其它任何应用程序的广播。
为了解决广播安全性问题，Android 引入了一套本地广播机制，使用这个机制发出的广播只能在应用程序内部进行传递，并且广播接受器也只能接受来自本应用程序发出的广播，这样所有的安全性问题就都不存在了。
1. 注册广播
    ```java
        LocalBroadcastManager localBroadcastManager = LocalBroadcastManager.getInstance(this);
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("om.function.luo.LOCAL_BROADCAST");
        LocalReceiver localReceiver = new LocalReceiver();
        localBroadcastManager.registerReceiver(localReceiver, intentFilter);

    ```
2. 发送广播
    ```java
        Intent intent = new Intent("om.function.luo.LOCAL_BROADCAST");
        localBroadcastManager.sendBroadcast(intent);
    ```
3. 接受广播
    ```java
     private class LocalReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "本地广播.....", Toast.LENGTH_LONG).show();
        }
    }
    ```
* 本地广播是无法通过静态注册的方式来接收的，其实这也完全可以理解，因为静态注册主要是为了让程序在未启动的情况下也能接受到广播，而发送本地广播时，我们的程序已经启动了，因此也完全不需要使用静态注册的功能。