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

在日常的开发中，我们通常需要在 Activity / Fragment 的生命周期方法中进行一些繁重的操作，这样使代码看起来十分臃肿。Lifecycle 的引入主要是用来管理和响应 Activity / Fragment 的生命周期的变化，帮助我们编写出更易于组织且通常更加轻量级的代码，让代码变得更易于维护。

Lifecycle 是一个类，它持有 Activity / Fragment 生命周期状态的信息，并允许其它对象观察此状态。

### Lifecycle 使用

场景：让 MVP 中的 Presenter 观察 Activity 的 onCreate 和 onDestroy 状态。

- Presenter 继承 LifecycleObserver 接口

```Kotlin
interface IPresenter : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate(owner: LifecycleOwner)

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy(owner: LifecycleOwner)

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY) // ON_ANY 注解能观察到其它所有的生命周期方法
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
//        Log.e(javaClass.simpleName, "onLifecycleChanged")
    }
}
```

- 在 Activity 中添加 LifecycleObserver

```Kotlin
class MyLifecycleActivity : AppCompatActivity() {

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

启动 Activity 会打印：

```Kotlin
MyLifecycleActivity: onCreate
MyPresenter: onCreate
```

finish Activity 会打印：

```Kotlin
MyPresenter: onDestroy
MyLifecycleActivity: onDestroy
```

以上 Presenter 对象只观察了 Activity 的 onCreate 方法和 onDestroy 方法，我们还可以观察其它的生命周期方法。在 Lifecycle 内部有个枚举类 Event , 它包含了 LifecycleObserver 能够观察到的所有生命周期方法，只需要添加上相应的注解即可。

```Kotlin
enum class Event {
    /**
     * Constant for onCreate event of the [LifecycleOwner].
     */
    ON_CREATE,
    /**
     * Constant for onStart event of the [LifecycleOwner].
     */
    ON_START,
    /**
     * Constant for onResume event of the [LifecycleOwner].
     */
    ON_RESUME,
    /**
     * Constant for onPause event of the [LifecycleOwner].
     */
    ON_PAUSE,
    /**
     * Constant for onStop event of the [LifecycleOwner].
     */
    ON_STOP,
    /**
     * Constant for onDestroy event of the [LifecycleOwner].
     */
    ON_DESTROY,
    /**
     * An [Event] constant that can be used to match all events.
     */
    ON_ANY
}
```

同时 Lifecycle 内部还有代表了各个**生命周期所处状态**的枚举类 State

```Kotlin
enum class State {
    /**
     * Destroyed state for a LifecycleOwner. After this event, this Lifecycle will not dispatch
     * any more events. For instance, for an [android.app.Activity], this state is reached
     * **right before** Activity's [onDestroy][android.app.Activity.onDestroy] call.
     */
    DESTROYED,

    /**
     * Initialized state for a LifecycleOwner. For an [android.app.Activity], this is
     * the state when it is constructed but has not received
     * [onCreate][android.app.Activity.onCreate] yet.
     */
    INITIALIZED,

    /**
     * Created state for a LifecycleOwner. For an [android.app.Activity], this state
     * is reached in two cases:
     *
     *  * after [onCreate][android.app.Activity.onCreate] call;
     *  * **right before** [onStop][android.app.Activity.onStop] call.
     *
     */
    CREATED,

    /**
     * Started state for a LifecycleOwner. For an [android.app.Activity], this state
     * is reached in two cases:
     *
     *  * after [onStart][android.app.Activity.onStart] call;
     *  * **right before** [onPause][android.app.Activity.onPause] call.
     *
     */
    STARTED,

    /**
     * Resumed state for a LifecycleOwner. For an [android.app.Activity], this state
     * is reached after [onResume][android.app.Activity.onResume] is called.
     */
    RESUMED;

    /**
     * Compares if this State is greater or equal to the given `state`.
     *
     * @param state State to compare with
     * @return true if this State is greater or equal to the given `state`
     */
    fun isAtLeast(state: State): Boolean {
        return compareTo(state) >= 0
    }
}
```

在一般开发中，当 Activity 拥有多个 Presenter 并需要在各个生命周期做一些特殊逻辑时，代码可能是：

```Kotlin
override fun onStop() {
    presenter1.onStop()
    presenter2.onStop()
    presenter3.onStop()
    super.onStop()
}

override fun onDestroy() {
    presenter1.onDestroy()
    presenter2.onDestroy()
    presenter3.onDestroy()
    super.onDestroy()
}
```

这样会使 Activity 的代码变得很臃肿。

如果用 Lifecycle , 只需将持有 Lifecycle 对象的 Activity 的生命周期的响应分发到各个 LifecycleObserver 观察者中即可。

```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_my_lifecycle)
    
    lifecycle.addObserver(presenter1) // 添加 LifecycleObserver
    lifecycle.addObserver(presenter2) // 添加 LifecycleObserver
    lifecycle.addObserver(presenter3) // 添加 LifecycleObserver
}
```

### 基本原理

**几个概念**

- **LifecycleObserver 接口**

  Lifecycle 观察者。实现了该接口的类，被 LifecycleOwner 类的 addObserver 方法注册后，通过注解的方式即可观察到 LifecycleOwner 的生命周期方法。

- **LifecycleOwner 接口**

  Lifecycle 持有者。实现了该接口的类持有生命周期（Lifecycle 对象），该接口生命周期（Lifecycle 对象）的改变会被其注册的观察者 LifecycleObserver 观察到并触发其对应的事件。

- **Lifecycle 类**

  生命周期。和 LifecycleOwner 不同，LifecycleOwner 通过 getLifecycle() 方法获取到内部的 Lifecycle 对象。

- **State**
  
  当前生命周期所处状态。Lifecycle 将 Activity 的生命周期函数对应成 State .

- **Event**

  当前生命周期改变对应的事件。State 变化将触发 Event 事件，从而被已注册的 LifecycleObserver 接收。

**实现原理**

- AppCompatActivity（LifecycleOwner）

  AppCompatActivity 的父类 `SupportActivity` 和 `Fragment` 一样，实现了 LifecycleOwner 接口，因此它们都拥有 Lifecycle 对象。

```Java
public class SupportActivity extends Activity implements LifecycleOwner, Component {
   
    // ... 
   
    private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

    public Lifecycle getLifecycle() {
        return this.mLifecycleRegistry;
    }
    
    // ...
}
```

```Java
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    @NonNull
    Lifecycle getLifecycle();
}
```

从源码可知 getLifecycle() 方法返回的是 `LifecycleRegistry` 对象，而 LifecycleRegistry 是 Lifecycle 的子类，所有对LifecycleObserver 的操作都是由 LifecycleRegistry 完成的。

- LifecycleRegistry

  生命周期登记处。作为 Lifecycle 的子类，它的作用是添加观察者、响应生命周期事件和分发生命周期事件。

```Java
public class LifecycleRegistry extends Lifecycle {

    // LifecycleObserver Map , 每一个 Observer 都有一个 State
    private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap =
            new FastSafeIterableMap<>();

    // 当前的状态
    private State mState;

    // Lifecycle 持有者，如继承了 LifecycleOwner 的 SupportActivity
    private final WeakReference<LifecycleOwner> mLifecycleOwner;

    public LifecycleRegistry(@NonNull LifecycleOwner provider) {
        mLifecycleOwner = new WeakReference<>(provider);
        mState = INITIALIZED;
    }
}
```


[文中 Demo GitHub 地址](https://github.com/zhich/AndroidJetpackDemo)