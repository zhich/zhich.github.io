---

title: Android Jetpack 之 WorkManager
date: 2020-04-12 16:13:00
categories: "Android"
tags:
     - Android
     - WorkManager
     - Jetpack
---

<meta name="referrer" content="no-referrer" />




### 概述

Android 在处理后台任务上，根据不同的需求给我们提供了 Service、JobScheduler、Loader 等。然而，大量的后台任务势必会过度消耗设备的电量。为了在设备电量和用户体验之间达到一个比较好的平衡，谷歌推出了 WorkManager。

WorkManager 是一个 Android 库，它在工作的触发器（如适当的网络状态和电池条件）满足时, 优雅地运行可推迟的后台工作。WorkManager 尽可能使用框架 JobScheduler , 以帮助优化电池寿命和批处理作业。在 Android 6.0（API 级 23）下面的设备上, 如果 WorkManager 已经包含了应用程序的依赖项，则尝试使用 Firebase JobDispatcher。否则，WorkManager 返回到自定义 AlarmManager 实现，以优雅地处理您的后台工作。

### 主要功能

- 最高向后兼容到 API 14
  - 在运行 API 23 及以上级别的设备上使用 JobScheduler
  - 在运行 API 14-22 的设备上结合使用 BroadcastReceiver 和 AlarmManager
- 添加网络可用性或充电状态等工作约束
- 调度一次性或周期性异步任务
- 监控和管理计划任务
- 将任务链接起来
- 确保任务执行，即使应用或设备重启也同样执行任务
- 遵循低电耗模式等省电功能

### 重要特点

- **针对不需要立即执行的任务：**比如向后端服务发送日志或分析数据，定期将应用数据与服务器同步等。从业务角度看，这些任务不需要立即执行。
- **保证任务一定会被执行：**即使应用程序当前不在运行中，哪怕彻底退出，或者设备重新启动，任务仍然会在适当的时候执行。这是因 WorkManager 有自己的数据库，关于任务的所有信息和数据都保存在这个数据库中。

WorkManager 不适用于应用进程结束时能够安全终止的运行中后台工作，也不适用于需要立即执行的任务。请查看[后台处理指南](https://developer.android.com/guide/background?hl=zh-cn)，了解哪种解决方案符合您的需求。

### 使用

**1、添加相关依赖**

使用 Java 或 Kotlin 语言将 WorkManager 依赖项添加到您的 Android 项目中。[依赖链接](https://developer.android.com/jetpack/androidx/releases/work?hl=zh-cn#declaring_dependencies)

```Kotlin
dependencies {
   def work_version = "2.3.1"

   // (Java only)
   implementation "androidx.work:work-runtime:$work_version"

   // Kotlin + coroutines
   implementation "androidx.work:work-runtime-ktx:$work_version"
}
```

**2、使用 Worker 定义任务**

Worker 是一个抽象类，用来指定需要执行的具体任务。我们需要继承 Worker 类，并实现它的 doWork 方法，所有需要在任务中执行的代码都在该方法中编写。

```Kotlin
class UploadLogWorker(context: Context, workerParams: WorkerParameters) : Worker(context, workerParams) {

    override fun doWork(): Result {
        LogUtil.d("UploadLogWorker", "doWork()")
        return Result.success()
    }

    override fun onStopped() {
        super.onStopped()
        // 当任务结束时会回调这里
    }
}
```

doWork 方法最后返回一个 Result，这个 Result 是一个枚举，它有几个固定的值：

- **Result.success()** 任务成功。
- **Result.Failure()** 任务失败。
- **Result.Retry()** 遇到暂时性失败，此时可使用 WorkRequest.Builder.setBackoffCriteria(BackoffPolicy, long, TimeUnit) 来重试。

**3、使用 WorkRequest 配置任务**

通过 WorkRequest 配置我们的任务**何时运行**以及**如何运行**。

- **设置任务触发条件** 。比如，我们可以设置设备处于充电中，网络已连接，并且电量充足的情况下才执行我们的任务（完整的触发条件列表，请参阅 [Constraints.Builder 参考文档](https://developer.android.com/reference/androidx/work/Constraints.Builder?hl=zh-cn)）。

  ```kotlin
  val constraints = Constraints.Builder()
        .setRequiresCharging(true) // 充电中
        .setRequiredNetworkType(NetworkType.CONNECTED) // 网络已连接
        .setRequiresBatteryNotLow(true) // 电量充足
        .build()
  ```

- **将 constraints 设置到 WorkRequest 中**。WorkRequest 是抽象类，它有两个子类，OneTimeWorkRequest 和 PeriodicWorkRequest，分别对应一次性任务和周期性任务。

  ```kotlin
  val oneTimeWorkRequest = OneTimeWorkRequest.Builder(UploadLogWorker::class.java)
          .setConstraints(constraints) // 设置触发条件
          .build()
  ```

- **设置延迟执行任务**。如果任务没有设置触发条件，或者所有触发条件都符合了，系统可能立刻执行任务，如果你希望再延后执行，则可以使用 setInitialDelay 方法。以下示例设置符合触发条件后，**至少**经过 5 分钟后再执行。

  ```Kotlin
  val oneTimeWorkRequest = OneTimeWorkRequest.Builder(UploadLogWorker::class.java)
        .setConstraints(constraints)
        .setInitialDelay(5, TimeUnit.MINUTES) // 符合触发条件后，至少经过 5 分钟后再执行
        .build()
  ```

    > 任务执行的确切时间还取决于 WorkRequest 中使用的触发条件和系统优化。WorkManager 经过设计，能够在满足这些触发条件的情况下提供可能的最佳行为。

- **设置指数退避策略**。如果需要 WorkManager 重新尝试执行任务，可以让 Worker 的 doWork 方法返回 Result.retry()。系统有默认的指数退避策略来帮助我们重新执行任务，我们也可以使用 setBackoffCriteria 方法来自定义指数退避策略。

  ```Kotlin
  val oneTimeWorkRequest = OneTimeWorkRequest.Builder(UploadLogWorker::class.java)
        .setBackoffCriteria(BackoffPolicy.LINEAR, OneTimeWorkRequest.MIN_BACKOFF_MILLIS, TimeUnit.MILLISECONDS)
        .build()
  ```

  > 比如 Worker 线程的执行出现异常，比如服务器宕机，那么我们可能就希望过一段时间再重新执行任务。

- **任务的输入/输出**。输入和输出值以键值对的形式存储在 Data 对象中。

  在 WorkRequest 中设置输入数据。

  ```Kotlin
val inputData = workDataOf("name" to "张三", "id" to 112134)
val uploadLogRequest = OneTimeWorkRequest.Builder(UploadLogWorker::class.java)
        .setInputData(inputData)
        .build()
  ```
  
  在 Worker 的 doWork 方法中取出输入数据。类似地，Data 类可用于输出返回值。要返回 Data 对象，请将它包含到 Result 的 Result.success() 或 Result.failure() 中。
  
  ```kotlin
  class UploadLogWorker(context: Context, workerParams: WorkerParameters) : Worker(context, workerParams) {
  
      override fun doWork(): Result {
          val name = inputData.getString("name")
          val id = inputData.getInt("id", 0)
          LogUtil.d("name-->$name, id--->$id")
  
          val outputData = workDataOf("name" to name, "id" to id)
          return Result.success(outputData)
      }
  }
  ```
  
  > 按照设计，Data 对象应该很小，值可以是字符串、基元类型或数组变体。如果需要将更多数据传入和传出工作器，应该将数据放在其它位置，例如 Room 数据库。Data 对象的大小上限为 10KB。

- **为任务设置标签**。设置了 Tag 后，可以通过 WorkManager.cancelAllWorkByTag(String) 取消使用特定 Tag 的所有任务，通过 WorkManager.getWorkInfosByTagLiveData(String) 返回 LiveData 和具有该 Tag 的所有任务的状态列表。

  ```Kotlin
val oneTimeWorkRequest1 = OneTimeWorkRequest.Builder(UploadLogWorker::class.java)
        .addTag("A")
        .build()
val oneTimeWorkRequest2 = OneTimeWorkRequest.Builder(UploadLogWorker::class.java)
        .addTag("A")
        .build()
  ```
  
  > 以上两个任务的 Tag 都设置为 A , 使这两个任务成为了一个组：A 组，这样的好处是以后可以操作整个组。

**4、将 WorkRequest 提交给系统**

```Kotlin
WorkManager.getInstance(applicationContext).enqueue(uploadLogRequest)
```

**5、观察任务的状态**

将任务提交给系统后，可通过 `WorkInfo` 获知任务的状态，WorkInfo 包含了任务的 id、tag、Worker 对象传递过来的 outputData，以及当前的状态。以下三种方式可以得到 WorkInfo 对象。

```Kotlin
WorkManager.getInstance(applicationContext).getWorkInfosByTag()
WorkManager.getInstance(applicationContext).getWorkInfoById()
WorkManager.getInstance(applicationContext).getWorkInfosForUniqueWork()
```

如果希望实时获知任务的状态，上面三个方法还有对于的 LiveData 方法。

```Kotlin
WorkManager.getInstance(applicationContext).getWorkInfosByTagLiveData()
WorkManager.getInstance(applicationContext).getWorkInfoByIdLiveData()
WorkManager.getInstance(applicationContext).getWorkInfosForUniqueWorkLiveData()
```

通过 LiveData，我们便可在任务状态发生变化的时候收到通知。

```Kotlin
WorkManager.getInstance(applicationContext).getWorkInfoByIdLiveData(uploadLogRequest.id).observe(this, Observer { workInfo ->
    if (workInfo != null && workInfo.state == WorkInfo.State.SUCCEEDED) {
        val name = workInfo.outputData.getString("name")
        val id = workInfo.outputData.getInt("id", 0)
        LogUtil.d("name-->$name, id-->$id")
    }
})
```

**6、取消任务**

可根据 tag、id 取消任务，也可以取消全部任务。

```Kotlin
WorkManager.getInstance(applicationContext).cancelAllWorkByTag()
WorkManager.getInstance(applicationContext).cancelWorkById()
WorkManager.getInstance(applicationContext).cancelUniqueWork()
WorkManager.getInstance(applicationContext).cancelAllWork()
```

**7、周期任务 PeriodicWorkRequest**

WorkRequest 有两种实现，OneTimeWorkRequest（一次性任务）和 PeriodicWorkRequest（周期性任务）。OneTimeWorkRequest 在任务成功完成后就结束了，而 PeriodicWorkRequest 会按设定的时间周期执行。二者使用起来无太大差别。

```Kotlin
val uploadWorkRequest = PeriodicWorkRequest.Builder(UploadLogWorker::class.java, 15, TimeUnit.MINUTES)
        .setConstraints(constraints)
        .build()
```

> 周期性任务的时间间隔不能少于 **15 分钟**。

**8、任务链**


- **并发任务**。使用 **WorkManager.getInstance().beginWith(...).enqueue()**。

  ```Kotlin
  // request1，request2 同时执行
  WorkManager.getInstance(applicationContext).beginWith(request1,request2).enqueue()
  ```

- **串发任务**。使用 **WorkManager.getInstance().beginWith().then().then()...enqueue()**。

  ```kotlin
  // 先执行 request1，然后执行 request2
  WorkManager.getInstance(applicationContext).beginWith(request1).then(request2).enqueue()
  ```

- **组合任务**。使用 **WorkContinuation.combine()** 方法。

  ```Kotlin
  // AB 串发，CD 串发，这两个串之间并发执行后，把汇总结果给到 E
  val chuan1 = WorkManager.getInstance(applicationContext)
        .beginWith(A)
        .then(B)
  val chuan2 = WorkManager.getInstance(applicationContext)
        .beginWith(C)
        .then(D)
  WorkContinuation.combine(listOf(chuan1, chuan2))
        .then(E)
        .enqueue()
  
  
  ```

### 注意点

- WorkManager 是根据系统版本决定用 JobScheduler 或者 BroadcastReceiver + AlarmManager 的组合，如果某个系统不允许 AlarmManager 自动唤起，那么 WorkManager 就可能无法正常工作。
- 实际操作中，周期任务的执行与所设定的时间可能有差别，执行时间可能也没有太明显的规律。



[文中 Demo GitHub 地址](https://github.com/zhich/AndroidJetpackDemo)

参考资料：

- [Android Developers](https://developer.android.com/topic/libraries/architecture/workmanager?hl=zh-cn)
- [WorkManager的基本使用](https://zhuanlan.zhihu.com/p/78599394)