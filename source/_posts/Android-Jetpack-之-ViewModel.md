---
title: Android Jetpack 之 ViewModel
date: 2018-105 8:38:00
categories: "Android"
tags:
     - Android
     - ViewModel
---






### 前言

在 Android 中，ViewModel 的作用就是在 **UI 控制器**（ 如 Activity、Fragment）的生命周期中保存和管理 UI 相关的数据。ViewModel 保存的数据在配置更改（如屏幕旋转）后会依然存在，不会丢失。

如屏幕旋转的时候，Activity 会重建，为了不让数据丢失，我们通常的做法是在 `onSaveInstanceState()` 方法中通过 bundle 保存数据，然后在 `onCreate()` 或 `onRestoreInstanceState()` 方法中取出 bundle 数据来恢复。然而，这种方式是有一定的局限性的，它只适用于**可序列化然后反序列化**的少量数据，对于 Bitmap 等比较大的数据就不适用了。

另一方面，UI 控制器通常需要做一些耗时的异步调用操作，并且需要去管理这些调用。UI 控制器需要确保系统在销毁后去清理掉这些异步调用，以避免潜在的内存泄漏，这种管理方式需要大量的维护工作。而且，在配置更改后重建对象是很浪费资源的，因为该对象可能必须重新发出之前已经发出过的调用。

UI 控制器一般只负责显示和处理用户操作，加载数据库数据或网络数据的工作应该委托给其它类，这样会让测试工作更加容易地进行。因此，**将视图数据相关操作从 UI 控制器逻辑中分离出来是很有必要。**

### ViewModel 使用

比如，一个 ViewModelActivity 需要展示一个 User 的列表数据，那么可以定义一个 UserViewModel 来获取数据，然后传给 ViewModelActivity 来展示。

```Kotlin
class UserViewModel : ViewModel() {

    private lateinit var users: MutableLiveData<List<User>>

    fun getUsers(): LiveData<List<User>> {
        if (!::users.isInitialized) {
            users = MutableLiveData()
            loadUsers()
        }
        return users
    }

    private fun loadUsers() {
        // Do an asynchronous operation to fetch users .
        Thread(Runnable {
            Thread.sleep(3000)
            // 由于在子线程发送值需要用 postValue , 否则用 setValue 就可以了。
            users.postValue(listOf(User("1", "AA"), User("2", "BB")))
        }).start()
    }
}
```

```Kotlin
class ViewModelActivity : AppCompatActivity() {

    private val TAG = "ViewModelActivity"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_view_model)

        // 就算配置更改（如屏幕旋转）了，获取到的 userViewModel 对象还会是上一次的 UserViewModel 对象
        val userViewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)

        // 这里的 this 需要用实现了 LifecycleOwner 的类的 this . 如 AppCompatActivity、FragmentActivity
        userViewModel.getUsers().observe(this, Observer {
            Log.e(TAG, it.toString())
            // 打印结果：[User(id=1, name=AA), User(id=2, name=BB)]
        })
    }
}
```

查看源码可知，ViewModelProviders.of(this) 获取了一个全新的 ViewModelProvider 对象，

```Kotlin
public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @Nullable Factory factory) {
        Application application = checkApplication(activity);
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }
```

ViewModelProvider 对象调用 get() 方法获取到我们需要的 ViewModel 对象。追踪一下 get() 方法可以知道，ViewModel 对象是存储在一个 ViewModelStore 类的对象中的，该类里面使用 HashMap 来保存和获取 ViewModel . 

```Kotlin
ViewModel viewModel = mViewModelStore.get(key);
```

获取 ViewModel 使用的 key 相对具体的 ViewModel 类是不会变化的，因此从 ViewModelStore 中取出的 ViewModel 对象也不会变。包括在配置更改后也可以获取到之前的 ViewModel .

当宿主 Activity 调用了 finish() 方法，系统会调用 ViewModel 对象的 onCleared() 方法来让它清理掉资源，到这里之后 ViewModel 才会被释放掉。

> ViewModel 里面不要引用 View、或者任何持有 Activity 类的 context , 否则会引发内存泄漏问题。

当 ViewModel 需要 Application 类的 context 来获取资源、查找系统服务等，可以继承 **AndroidViewModel **类。 

```Kotlin
class MyAndroidViewModel(application: Application) : AndroidViewModel(application) {

    private val app
        get() = getApplication<Application>()

    fun getStatus(code: Int): String {
        return when (code) {
            1 -> app.resources.getString(R.string.be_late) // 迟到
            2 -> app.resources.getString(R.string.leave_early) // 早退
            else -> app.resources.getString(R.string.absenteeism) // 旷工
        }
    }
}
```

```Kotlin
val myAndroidViewModel = ViewModelProviders.of(this).get(MyAndroidViewModel::class.java)
Log.e(TAG, myAndroidViewModel.getStatus(2))
// 打印结果：早退
```

### ViewModel 的生命周期

ViewModel 会一直保留在内存中，直到 Activity / Fragment 在以下情况下才会销毁：

- 宿主 Activity 被 finish 后调用 onDestroy 方法。
- 宿主 Fragment 被 detached 后调用 onDetach 方法。

下图展示了一个 Activity 经历了旋转然后调用 finish 的各种生命周期状态，同时展示了关联了该 Activity 的 ViewModel 的生命周期。（UI 控制器是 Fragment 的情况也类似。）

![Mou icon](http://pcckwdbix.bkt.clouddn.com/viewmodel-lifecycle.png)

### Fragment 之间共享数据

