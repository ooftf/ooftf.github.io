## 获取当前Service 列表
```kotlin
  fun listServices(mContext: Context): MutableList<ActivityManager.RunningServiceInfo> {
        val activityManager = mContext.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
        return activityManager.getRunningServices(100)
    }
```