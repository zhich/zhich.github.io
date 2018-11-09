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

Lifecycle 内部还有代表了各个**生命周期所处状态**的枚举类 State

```Kotlin
enum class State {

    /**
     * Destroyed state for a LifecycleOwner. After this event, this Lifecycle will not dispatch
     * any more events. For instance, for an [android.app.Activity], this state is reached
     * before Activity's [onDestroy] call.
     */
    DESTROYED,

    /**
     * Initialized state for a LifecycleOwner. For an [android.app.Activity], this is
     * the state when it is constructed but has not received
     * [onCreate] yet.
     */
    INITIALIZED,

    /**
     * Created state for a LifecycleOwner. For an [android.app.Activity], this state
     * is reached in two cases:
     *
     * after [onCreate] call;
     * before [onStop] call.
     */
    CREATED,

    /**
     * Started state for a LifecycleOwner. For an [android.app.Activity], this state
     * is reached in two cases:
     *
     * after [onStart] call;
     * before [onPause] call.
     */
    STARTED,

    /**
     * Resumed state for a LifecycleOwner. For an [android.app.Activity], this state
     * is reached after [onResume] is called.
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

   /**
    * 添加 LifecycleObserver 观察者，并将之前的状态分发给这个 Observer , 例如我们在 onResume 之后注册这个 Observer , 
    * 该 Observer 依然能收到 ON_CREATE 事件
    */
    @Override
    public void addObserver(@NonNull LifecycleObserver observer) {
        // ...
        // 例如：Observer 初始状态是 INITIALIZED , 当前状态是 RESUMED , 需要将 INITIALIZED 到 RESUMED 之间的
        // 所有事件分发给 Observer
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }
        // ...
    }

    /**
     * 处理生命周期事件
     */
    public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        State next = getStateAfter(event);
        moveToState(next);
    }

    /**
     * 改变状态
     */
    private void moveToState(State next) {
        if (mState == next) {
            return;
        }
        mState = next;
        // ...
        sync();
        // ...
    }

    /**
     * 同步 Observer 状态，并分发事件
     */
    private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            Log.w(LOG_TAG, "LifecycleOwner is garbage collected, you shouldn't try dispatch "
                    + "new events from it.");
            return;
        }
        while (!isSynced()) {
            mNewEventOccurred = false;
            // State中，状态值是从 DESTROYED - INITIALIZED - CREATED - STARTED - RESUMED 增大
            // 如果当前状态值 < Observer 状态值，需要通知 Observer 减小状态值，直到等于当前状态值
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            // 如果当前状态值 > Observer 状态值，需要通知 Observer 增大状态值，直到等于当前状态值
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }

    /**
     * 向前传递事件。
     * 增加 Observer 的状态值，直到状态值等于当前状态值
     */
    private void forwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                pushParentState(observer.mState);
                // 分发状态改变事件
                observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                popParentState();
            }
        }
    }

    /**
     * 向后传递事件。
     * 减小 Observer 的状态值，直到状态值等于当前状态值
     */
    private void backwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                mObserverMap.descendingIterator();
        while (descendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                Event event = downEvent(observer.mState);
                pushParentState(getStateAfter(event));
                observer.dispatchEvent(lifecycleOwner, event);
                popParentState();
            }
        }
    }
}
```
根据上面的分析，我们知道 LifecycleRegistry 才是真正替 Lifecycle 去埋头干粗活的类！

接下来继续来看看实现了 LifecycleOwner 接口的 SupportActivity 类是如何将事件分发给 LifecycleRegistry 的。

```Java
public class SupportActivity extends Activity implements LifecycleOwner, Component {

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
    }
}
```

注意到 SupportActivity 的 onCreate() 方法里面有行 `ReportFragment.injectIfNeededIn(this)` 代码，再进入 ReportFragment 类分析。

- ReportFragment

```Java
public class ReportFragment extends Fragment {

    public static void injectIfNeededIn(Activity activity) {
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            manager.executePendingTransactions();
        }
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }

    @Override
    public void onPause() {
        super.onPause();
        dispatch(Lifecycle.Event.ON_PAUSE);
    }

    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
        // just want to be sure that we won't leak reference to an activity
        mProcessListener = null;
    }

    /**
     * 分发事件
     */
    private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }

        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }
}
```

不难看出这是一个没有 UI 的后台 Fragment , 一般可以为 Activity 提供一些后台行为。在 ReportFragment 的各个生命周期中都调用了 LifecycleRegistry.handleLifecycleEvent() 方法来分发生命周期事件。

**为什么不直接在 SupportActivity 的生命周期函数中给 Lifecycle 分发生命周期事件，而是要加一个 Fragment 呢？**

在 ReportFragment 的 injectIfNeededIn() 方法中找到答案：

```Java
public static void injectIfNeededIn(Activity activity) {
    // ProcessLifecycleOwner should always correctly work and some activities may not extend
    // FragmentActivity from support lib, so we use framework fragments for activities
    android.app.FragmentManager manager = activity.getFragmentManager();
    if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
        manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
        // Hopefully, we are the first to make a transaction.
        manager.executePendingTransactions();
    }
}
```

有两个原因：为了能让 ProcessLifecycleOwner 正确地工作；②、并非所有的 Activity 都是继承来自 support 包的 FragmentActivity 类的。因此封装一个同样具有生命周期的后台 Fragment 来给 Lifecycle 分发生命周期事件。

**另一方面，假如我们的 Activity 没有继承自 SupportActivity , 那我们如何分发生命周期事件呢？**

鼠标停在 ReportFragment 类，同时按下 `Ctrl + Shift + Alt + F7` 在 Project and Libraries 的范围下搜索 ReportFragment 被引用的地方。我们发现还有 LifecycleDispatcher 和 ProcessLifecycleOwner 两个类有使用到 ReportFragment .


[文中 Demo GitHub 地址](https://github.com/zhich/AndroidJetpackDemo)