---
title: Android Jetpack 之 LiveData
date: 2018-11-22 8:28:00
categories: "Android"
tags:
     - Android
     - LiveData
     - Jetpack
---






### 概述

- LiveData 是一个持有数据的类，它持有的数据是可以被观察者订阅的，当数据被修改时就会通知观察者。观察者可以是 Activity、Fragment、Service 等。
- LiveData 能够感知观察者的生命周期，只有当观察者处于激活状态（STARTED、RESUMED）才会接收到数据更新的通知，在未激活时会自动解注册观察者，以减少内存泄漏。
- 使用 LiveData 保存数据时，由于数据和组件是分离的，当组件重建时可以保证数据不会丢失。

### 使用

LiveData 有两种使用方式，结合 ViewModel 使用以及直接继承 LiveData 类。

**结合 ViewModel 使用**

以下代码场景：点击按钮提示一个名字。

```Kotlin
class MyViewModel : ViewModel() {

    // 创建一个 String 类型的 LiveData
    private lateinit var name: MutableLiveData<String>

    fun getName(): MutableLiveData<String> {
        if (!::name.isInitialized) {
            name = MutableLiveData()
        }
        return name
    }
}
```

```Kotlin
class LiveDataActivity : AppCompatActivity() {

    private lateinit var myViewModel: MyViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_live_data)

        // 创建并注册观察者
        myViewModel = ViewModelProviders.of(this).get(MyViewModel::class.java)
        myViewModel.getName().observe(this, Observer {
            // LiveData 数据更新回调，it 代表被观察对象的数据，此处为 name
            Toast.makeText(baseContext, it, Toast.LENGTH_SHORT).show()
        })

        btnSetName.setOnClickListener {
            // 使用 setValue 更新 LiveData 数据
            myViewModel.getName().value = "张三"
        }
    }
}
```

> 让数据（name）和组件（LiveDataActivity）分离，当 Activity 重建时，数据（name）不会丢失。

**直接继承 LiveData 类**

以下代码场景：在 Activity 中监听 Wifi 信号强度。

```Kotlin
class WifiLiveData private constructor(context: Context) : LiveData<Int>() {

    private var mContext: WeakReference<Context> = WeakReference(context)

    companion object {

        private var instance: WifiLiveData? = null

        fun getInstance(context: Context): WifiLiveData {
            if (instance == null) {
                instance = WifiLiveData(context)
            }
            return instance!!
        }
    }

    override fun onActive() {
        super.onActive()
        registerReceiver()
    }

    override fun onInactive() {
        super.onInactive()
        unregisterReceiver()
    }

    /**
     * 注册广播，监听 Wifi 信号强度
     */
    private fun registerReceiver() {
        val intentFilter = IntentFilter()
        intentFilter.addAction(WifiManager.RSSI_CHANGED_ACTION)
        mContext.get()!!.registerReceiver(mReceiver, intentFilter)
    }

    /**
     * 注销广播
     */
    private fun unregisterReceiver() {
        mContext.get()!!.unregisterReceiver(mReceiver)
    }

    private val mReceiver = object : BroadcastReceiver() {

        override fun onReceive(context: Context?, intent: Intent) {
            when (intent.action) {
                WifiManager.RSSI_CHANGED_ACTION -> getWifiLevel()
            }
        }
    }

    private fun getWifiLevel() {
        val wifiManager = mContext.get()!!.applicationContext.getSystemService(android.content.Context.WIFI_SERVICE) as WifiManager
        val wifiInfo = wifiManager.connectionInfo
        val level = wifiInfo.rssi

        instance!!.value = level // 发送 Wifi 的信号强度给观察者
    }
}
```

```Kotlin
class LiveDataActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_live_data)

        withExtendsLiveDataTest()
    }

    /**
     * 直接继承 LiveData 类
     */
    private fun withExtendsLiveDataTest() {
        WifiLiveData.getInstance(this).observe(this, Observer {
            Log.e("LiveDataActivity", it.toString()) // 打印 Wifi 信号强度
        })
    }
}
```

> 当 Activity 处于激活状态（onActive）时注册广播，处于非激活状态时注销广播（onInactive）。