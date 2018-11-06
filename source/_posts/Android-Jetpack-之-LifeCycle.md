---
title: Android Jetpack 之 Lifecycle
date: 2018-11 6 21:15:00
categories: "Android"
tags:
     - Android
     - Lifecycle
     - Jetpack
---






### 前言

在日常的开发中，我们通常需要在 Activity / Fragment 的生命周期方法中进行一些繁重的操作，这样使代码看起来十分臃肿。LifeCycle 的引入主要是用来管理和响应 Activity / Fragment 的生命周期的变化，帮助我们编写出更易于组织且通常更加轻量级的代码，让代码变得更易于维护。

LifeCycle 是一个类，它持有 Activity / Fragment 生命周期状态的信息，并允许其它对象观察此状态。

### Lifecycle使用

场景：让 MVP 中的 Presenter 观察 Activity 的 onCreate 和 onDestroy 状态。

- Presenter 继承 LifecycleObserver 接口

```Kotlin
interface IPresenter : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate(owner: LifecycleOwner)

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy(owner: LifecycleOwner)

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    fun onLifecycleChanged(owner: LifecycleOwner, event: Lifecycle.Event)
}
```

```Kotlin
class MyPresenter : IPresenter {

    override fun onCreate(owner: LifecycleOwner) {
        Log.e(javaClass.simpleName, "onCreate")
    }

    override fun onDestroy(owner: LifecycleOwner) {
        Log.e(javaClass.simpleName, "onDestroy")
    }

    override fun onLifecycleChanged(owner: LifecycleOwner, event: Lifecycle.Event) {
        Log.e(javaClass.simpleName, "onLifecycleChanged")
    }
}
```

- 在 Activity 中添加 Observer

```Kotlin
class MyLifeCycleActivity : AppCompatActivity() {

    private lateinit var myPresenter: MyPresenter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my_lifecycle)

        Log.e(javaClass.simpleName, "onCreate")

        myPresenter = MyPresenter()
        lifecycle.addObserver(myPresenter) // 添加 LifecycleObserver
    }

    override fun onDestroy() {
        Log.e(javaClass.simpleName, "onDestroy")
        super.onDestroy()
    }
}
```




[文中 Demo GitHub 地址](https://github.com/zhich/AndroidJetpackDemo)