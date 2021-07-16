---
title: WorkManager
---

# [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager) 
调度在退出应用或重启设备后仍应运行的**可延期**异步任务
#### 使用底层作业来调度服务
![workmanager](https://raw.githubusercontent.com/ooftf/Material/master/img/blogWorkManager.png)
#### 示例
1. 创建一个Worker
   ```kotlin
   class UploadWorker(context: Context, workerParams: WorkerParameters) : Worker(context, workerParams) {
       override fun doWork(): Result {
           inputData.getString("data")
           if(){
               Result.success()
           }else if(){
               Result.retry()
           }else{
               Result.failure()
           }
           
       }
   }
   ```kotlin
2. 执行Worker

   ```kotlin
    val data = Data.Builder()
               .putString("data", jsonParser.object2Json(request))
               .build()
    val uploadWorkRequest: WorkRequest =
        OneTimeWorkRequestBuilder<UploadWorker>()
            .setInputData(data)
            .build()
    WorkManager
        .getInstance(AppHolder.app)
        .enqueue(uploadWorkRequest)
   ```

