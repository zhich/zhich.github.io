---
title: Android 代码优化小提示
date: 2018-01-09 22:35:29
categories: "Android"
tags:
     - Android
---

## 代码逻辑相关

### 遍历一个List集合

bad:

```Java
List<User> userList = new ArrayList<>();
for (int i = 0; i < userList.size(); i++) {
    User user = new User();
   	 //省略 n 行代码...
    userList.add(user);
}
```

better:

```Java
List<User> userList = new ArrayList<>();
User user = null;
for (int i = 0, size = userList.size(); i < size; i++) {
    user = new User();
    //省略 n 行代码...
    userList.add(user);
}
```

> ①、第二种方式把 userList.size() 用变量 size 存值避免了每次迭代都去调用该方法（不要在 for 循环的第二个表达式的判 断调用对象的方法或字段）；②、把 user 变量的声明放到循环外边，避免每次使用都去声明一下 User 对象。 ③、getCount() 方法的处理也类似，如用 int count = xxx.getCount() 缓存起来。

### 遍历HashMap的最佳方法

```Java
Map<String, User> userMap = new HashMap<>();
Iterator it = userMap.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry entry = (Map.Entry) it.next();
    System.out.println(entry.getKey() + " = " + entry.getValue());
    //迭代器方法遍历过程中可以通过 it.remove(); 删除当前遍历的元素
    it.remove(); // 避免抛 ConcurrentModificationException 异常
}
```

### 字符串

#### 判断字符串str是否为null或空串

bad:

```Java
if (null == str || "".equals(str)) {
}
```

better:

```Java
if (TextUtils.isEmpty(str)) {
}
```

> TextUtils 是一个非常好用的工具类，把 List 转成字符串，逗号分隔；逗号分隔的 String 字符串，切割成 List ，分别可以	用 TextUtils 的 join 和 split 方法。

### 容易报空指针的情况

#### 判断一个字符串的内容是否为某值

bad:

```Java
if (str.equals("hello")) {
}
```

better:

```Java
// 避免空指针异常，应该把常量写在前面
if ("hello".equals(str)) {
}
```

#### 判断集合某个元素对象的某个字段是否为空

bad:

```Java
// 如果元素对象为 null 这里就挂了吧
if (null != userList.get(i).name) {
}
```

better:

```Java
// 使用对象的方法或字段时，考虑下对象本身是否可能为 null
if (null != userList.get(i) && null != userList.get(i).name) {
}
```

### 常用资源释放

#### Cursor

```Java
try {
    cursor = context.getContentResolver().query(Uri.parse(PROVIDER_SETTINGFILE), null, null, null, null);
    if (null == cursor) { // cursor 为 null，下面没法玩
        return;
    }
    //省略 n 行代码...
} catch (Exception e) {
    e.printStackTrace();
} finally {
    try {
        cursor.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

#### 流文件

```Java
public class Test {

    private static final int BUFFER_SIZE = 1024;

    public static void main(String[] args) {
        FileReader fr = null;
        FileWriter fw = null;
        try {
            fr = new FileReader("origin.txt");
            fw = new FileWriter("destination.txt");
            char[] buf = new char[BUFFER_SIZE];
            int len = 0;
            while ((len = fr.read(buf)) != -1) {
                fw.write(buf, 0, len);
            }
        } catch (Exception e) {
            throw new RuntimeException("读写失败");
        } finally {
            if (fw != null)
                try {
                    fw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            if (fr != null)
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
        }
    }
}
```

#### WebView



#### Handler

- 使用弱引用

```Java
public class NoLeakActivity extends Activity {

    private NoLeakHandler mNoLeakHandler;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mNoLeakHandler = new NoLeakHandler(this);
        Message message = Message.obtain();
        mNoLeakHandler.sendMessageDelayed(message, 2000);
    }

    private static class NoLeakHandler extends Handler {

        private WeakReference<NoLeakActivity> mActivity;

        public NoLeakHandler(NoLeakActivity activity) {
            mActivity = new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    }
}
```

- 及时清除消息

```Java
public class NoLeakActivity extends Activity {

    private Handler mHandler = new Handler();

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
//                startMainActivity();
            }
        }, 2000);
    }

    @Override
    protected void onDestroy() {
        // 把所有的消息和回调移除
        mHandler.removeCallbacksAndMessages(null);
        super.onDestroy();
    }

    @Override
    public void onBackPressed() {
        // 把所有的消息和回调移除（onDestroy 执行不确定，因此这里需执行一遍）
        mHandler.removeCallbacksAndMessages(null);
        super.onBackPressed();
    }
}
```

### 其它

#### 反面判断条件

bad:

```Java
public void testMethod(ArrayList<User> userList) {
	if (null != userList && userList.size() > 0) {
		for (int i = 0, size = userList.size(); i < size; i++) {
			// ...
		}
	}
}
```

better:

```Java
public void testMethod(ArrayList<User> userList) {
	if (null == userList || userList.isEmpty()) {
		return;
	}
	for (int i = 0, size = userList.size(); i < size; i++) {
		// ...
	}
}
```

> 很多比较复杂的层级判断都可以从这些判断的反面出发，来降低程序的复杂性，从而提高可读性。

#### if与return搭配

bad:

```Java
public int testIfElse(String cmd) {
    if ("1".equals(cmd)) {
   	 return 1;
    } else if ("2".equals(cmd)) {
   	 return 2;
    } else if ("3".equals(cmd)) {
   	 return 3;
    } else {
   	 return 4;
    }
}
```

better:

```Java
public int testIfElse(String cmd) {
   if ("1".equals(cmd)) {
       return 1;
   }
   if ("2".equals(cmd)) {
       return 2;
   }
   if ("3".equals(cmd)) {
       return 3;
   }
   return 4;
}
```

#### 对象序列化

Android 中的序列化官方推荐 Parceble , 其实 Parceble 最好用于内存之间数据的交换，如果要把数据写入硬盘的话，推荐实现 Serializable .

#### SharedPreferences

SharedPreferences.Editor.commit 这个方法是同步的，一直到把数据同步到 Flash 上面之后才会返回，由于 IO 操作的不可控，尽量使用 apply 方法代替。apply 只在 API Level >= 9 才会支持，需要做兼容。不过，最新的 support v4 包已经为我们做好了处理，使用  SharedPreferencesCompat.EditorCompat.getInstance().apply(editor) 即可。

#### 其它优化

- 静态变量不要直接或者间接引用 Activity、Service 等。这会使 Activity 以及它所引用的所有对象无法释放，然后，用户操作时间一长，内存就会狂升。

- Handler 机制有一个特点是不会随着 Activity、Service 的生命周期结束而结束。也就是说，如果你 Post 了一个 Delay 的 Runnable , 然后在 Runnable 执行之前退出了 Activity , Runnable 到时间之后还是要执行的。如果 Runnable 里面包含更新 View 的操作，程序崩溃了。

- 不少人在子线程中更新 View 时喜欢使用 Context.runOnUiThread , 这个方法有个缺点，就是一但 Context 生命周期结束，比如 Activity 已经销毁时，一调用就会崩溃。

- Application 的生命周期就是进程的生命周期。只有进程被干掉时，Application 才会销毁。哪怕是没有 Activity、Service 在运行，Application 也会存在。所以，为了减少内存压力，尽量不要在 Application 里面引用大对象、Context 等。

- Activity 的 onDestroy 方法调用时机是不确定的（有时候离开界面很久之后才会调用 onDestroy 方法），应该避免指望通过 onDestroy 方法去释放与 Activity 相关的资源，否则会导致一些随机 bug .

- 如果使用 Context.startActivity 启动外部应用，最好做一下异常预防，因为寻找不到对应的应用时，会抛出异常。如果你要打开的是应用内的 Activity , 不防使用显式 Intent , 这样能提高系统搜索目标 Activity 的效率。

- 如果子类实现 Serializable 接口而父类未实现时，父类不会被序列化，但此时父类必须有个无参构造方法，否则会抛 InvalidClassException 异常。

- transient 关键字修饰变量可以限制序列化。

- View.getContext 获取控件上下文，写 RecyclerView 的 Adapter 的时候，为了使用 LayoutInflater , 不用在构造函数中传入一个外部的 context .

## UI相关

### Space

Space 经常用于组件之间的缝隙，其 draw()为空，减少了绘制渲染的过程。组件之间的距离使用 Space 会提高了绘制效率，特别是对于动态设置间距会很方便高效。正是因为 draw()为空，对该 view 没有做任务绘制渲染，所以不能对 Space 设置背景色。

### tools标签

tools 标签可以很好的帮助开发者实时预览 xml 的效果，并且运行以后 tools 标签的内容不会展示出来。例如：

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    tools:text="这段话只在预览时能看到,运行以后就看不到了" />
```

### ContextCompat

6.0 之后 getResources().getColor() 方法被废弃了，可以用 ContextCompat.getColor(context, R.color.color_name) 替换，ContextCompat 是 v4 包里的，请放心使用，另外还有 getDrawable() 等方法。
