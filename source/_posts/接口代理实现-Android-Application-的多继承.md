---
title: 接口代理实现 Android Application 的多继承
date: 2018-08-03 11:57:00
categories: "Android"
tags:
     - Android
---

<meta name="referrer" content="no-referrer">





在项目开发中，当我们接入第三方 SDK 时，可能会要求我们自己的 Application 继承它们的 Application , 但是 Java 只能是单继承的，这时我们就可以使用**接口代理**的方式来间接地实现“多继承”了。

假设我们的 MyApplication 需要继承两个第三方 SDK 的类 XApplication 与 YApplication .

```Java
package com.zch.demo.app;

import android.app.Application;
import android.content.Context;
import android.content.res.Configuration;
import android.util.Log;

/**
 * Created by zch on 2018/8/3.
 */
public class XApplication extends Application {

    @Override
    public void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        Log.d("info-->", "XApplication attachBaseContext");
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("info-->", "XApplication onCreate");
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        Log.d("info-->", "XApplication onConfigurationChanged");
    }
}
```

```Java
package com.zch.demo.app;

import android.app.Application;
import android.content.Context;
import android.content.res.Configuration;
import android.util.Log;

/**
 * Created by zch on 2018/8/3.
 */
public class YApplication extends Application {

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        Log.d("info-->", "YApplication attachBaseContext");
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("info-->", "YApplication onCreate");
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        Log.d("info-->", "YApplication onConfigurationChanged");
    }
}
```

显然，我们自家的 MyApplication 是不可能同时直接继承上面的两个 Application 的。我们可以让 MyApplication 继承一个代理类 ProxyApplication , 然后在 ProxyApplication 中通过**反射和接口回调**的方式调用组合实现类 ApplicationImpl（组合了多个 Application 接口）对应的方法。

```Java
package com.zch.demo.app;

import android.content.Context;
import android.content.res.Configuration;
import android.util.Log;

/**
 * Created by zch on 2018/8/3.
 */
public class MyApplication extends ProxyApplication {

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);

        Log.d("info-->", "MyApplication attachBaseContext");
    }

    @Override
    public void onCreate() {
        super.onCreate();

        Log.d("info-->", "MyApplication onCreate");
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);

        Log.d("info-->", "MyApplication onConfigurationChanged");
    }
}
```

```Java
package com.zch.demo.app;

import android.app.Application;
import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public class ProxyApplication extends Application {

    private IApplicationListener iApplicationListener;

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);

        iApplicationListener = getProxyApplication();

        if (iApplicationListener != null) {
            iApplicationListener.onProxyAttachBaseContext(base);
        }
    }

    @Override
    public void onCreate() {
        super.onCreate();

        if (iApplicationListener != null) {
            iApplicationListener.onProxyCreate();
        }
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);

        if (iApplicationListener != null) {
            iApplicationListener.onProxyConfigurationChanged(newConfig);
        }
    }

    private IApplicationListener getProxyApplication() {
        try {
            Class clazz = Class.forName("com.zch.demo.app.ApplicationImpl");
            return (IApplicationListener) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

在组合实现类 ApplicationImpl 中，我们通过反射代理的方式**调用多个 Application 的生命周期方法**。

```Java
package com.zch.demo.app;

import android.app.Application;
import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public class ApplicationImpl extends Application implements IApplicationListener {

    private IXApplicationListener ixApplicationListener;
    private IYApplicationListener iyApplicationListener;

    @Override
    public void onProxyAttachBaseContext(Context base) {
        super.attachBaseContext(base);

        ixApplicationListener = getXApplication();
        iyApplicationListener = getYApplication();

        if (ixApplicationListener != null) {
            ixApplicationListener.onXAttachBaseContext(base);
        }
        if (iyApplicationListener != null) {
            iyApplicationListener.onYAttachBaseContext(base);
        }
    }

    @Override
    public void onProxyCreate() {
        super.onCreate();

        if (ixApplicationListener != null) {
            ixApplicationListener.onXCreate();
        }
        if (iyApplicationListener != null) {
            iyApplicationListener.onYCreate();
        }
    }

    @Override
    public void onProxyConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);

        if (ixApplicationListener != null) {
            ixApplicationListener.onXConfigurationChanged(newConfig);
        }
        if (iyApplicationListener != null) {
            iyApplicationListener.onYConfigurationChanged(newConfig);
        }
    }

    private IXApplicationListener getXApplication() {
        Class clazz = null;
        try {
            clazz = Class.forName("com.zch.demo.app.XApplicationImpl");
            return (IXApplicationListener) clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return null;
    }

    private IYApplicationListener getYApplication() {
        Class clazz = null;
        try {
            clazz = Class.forName("com.zch.demo.app.YApplicationImpl");
            return (IYApplicationListener) clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

接口 IApplicationListener

```Java
package com.zch.demo.app;

import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public interface IApplicationListener {

    void onProxyAttachBaseContext(Context base);

    void onProxyCreate();

    void onProxyConfigurationChanged(Configuration newConfig);
}
```

实现类 XApplicationImpl

```Java
package com.zch.demo.app;

import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public class XApplicationImpl extends XApplication implements IXApplicationListener {

    @Override
    public void onXAttachBaseContext(Context base) {
        super.attachBaseContext(base);
    }

    @Override
    public void onXCreate() {
        super.onCreate();
    }

    @Override
    public void onXConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
}
```

接口 IXApplicationListener

```Java
package com.zch.demo.app;

import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public interface IXApplicationListener {

    void onProxyAttachBaseContext(Context base);

    void onProxyCreate();

    void onProxyConfigurationChanged(Configuration newConfig);
}
```

实现类 YApplicationImpl

```Java
package com.zch.demo.app;

import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public class YApplicationImpl extends YApplication implements IYApplicationListener {

    @Override
    public void onYAttachBaseContext(Context base) {
        super.attachBaseContext(base);
    }

    @Override
    public void onYCreate() {
        super.onCreate();
    }

    @Override
    public void onYConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
}
```

接口 IYApplicationListener

```Java
package com.zch.demo.app;

import android.content.Context;
import android.content.res.Configuration;

/**
 * Created by zch on 2018/8/3.
 */
public interface IYApplicationListener {

    void onYAttachBaseContext(Context base);

    void onYCreate();

    void onYConfigurationChanged(Configuration newConfig);
}
```

当我们在清单文件中声明了 MyApplication 并跑起项目时，会打印以下日志：

```
08-03 13:56:33.830 9314-9314/? D/info-->: XApplication attachBaseContext
08-03 13:56:33.830 9314-9314/? D/info-->: YApplication attachBaseContext
08-03 13:56:33.830 9314-9314/? D/info-->: MyApplication attachBaseContext
08-03 13:56:33.830 9314-9314/? D/info-->: XApplication onCreate
08-03 13:56:33.830 9314-9314/? D/info-->: YApplication onCreate
08-03 13:56:33.830 9314-9314/? D/info-->: MyApplication onCreate
```

说明我们在 MyApplication 中成功地调用了 XApplication 与 YApplication 中的生命周期方法。