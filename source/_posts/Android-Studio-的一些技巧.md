---
title: Android Studio 的一些技巧
date: 2016-09-25 12:28:31
categories: "Android"
tags:
     - Android
     - Android Studio
---



[技巧](#技巧)

[插件](#插件)

[工具](#工具)



## 技巧
### 快速查找
`双击 Shift 键`。堪比 Alfred 的功能。对输入的内容进行模糊查询，若勾选上 Include none-project items 后，还可以搜索非项目中的内容，如引用的 jar 包中的内容。

### Search Action
`Ctrl + Shift + A`。类似搜索指令的入口。如输入 "Open Recent" 可以查找最近的工程；输入 "hier" 选中 Type Hierarchy ^H 即可以查看某个方法或类的调用栈。

### 演示模式
在菜单栏 `View 选项` 最下面可找到几种极为方便的演示模式。通过选择这几种模式可以在连接投影仪时非常方便地全屏显示代码区域。代码区在 mac 下可通过双指缩放进行代码区域的缩放。

### 显示最近操作、修改
`Ctrl + E` 和 `Ctrl + Shift + E` 快速显示最近文件操作和文件修改。同时可用 `Ctrl + Tab` 进行各个界面的切换。

### 操作记录前进和回退
`Ctrl + Alt + Left/Right`。

### 移动行
`Alt + Shift + 方向键上/方向键下`。整体移动也是类似的方法。

### 交换行
`Ctrl + Shift + 方向键上/方向键下`。

### Log 快捷模板
在 onCreate 中输入 `logi` ，按 Enter 即可生成如下一条 Log 日志。相应地，其它级别的日志也可类似地快速生成。
```Java
Log.i(TAG, "onCreate: ");
```

### 查看大纲
`(Fn) + Ctrl + F12 `。大纲界面显示方法和成员列表。输入关键字可模糊查询方法和成员。

### 附加调试
在 ADB 连接手机情况下，点击 `attach to debugger` 按钮并选择要调试的程序（只能调试 Debug 签名的 App），即可进入调试模式，无需通过 Debug 运行程序。对于大项目，这种方式可以以正常的方式进行程序运行，如果使用 Debug 模式运行会非常卡。

### 代码折叠
全局折叠、展开：`Ctrl + Shift + -` 、`Ctrl + Shift + +` 
局部折叠、展开：`Ctrl  + -` 、`Ctrl  + +` 

### 在文件系统中打开文件
按住 Ctrl 键并点击打开的代码的 Tab 页。

### 预览方法定义
`Ctrl + Shift + i (mac 为 Command + Y)`。在本页面预览方法的定义，无需跳转到方法定义的地方去。

### 拆分窗口
在编辑区域显示多个编辑界面：Window --> Editor Tabs --> Split vertical \ horizontal

### Extract 的妙用
**Extract 可以重构 Java 代码；抽取布局 XML 的一些属性作为 Style；抽取布局 Layout。**
在代码中，Extract 可提取各种变量、参数、常量。如将一个局部变量提取为类的成员变量，将一个字符串的常量提取为全局的常量（可选择提取到这个类本身或新的类中）。

### 方法调用栈
`Ctrl + Alt + H` 可以快速找到该方法的调用栈。

### Surround With
`Ctrl + Alt + T`。可快速对某段代码进行重构，如增加判空的 if 条件、增加 try catch 捕获异常。

### Image Asset && Vector Asset
可帮助快速创建不同分辨率的图像和 SVG 文件。要使用该功能，在 res 资源目录下右键选择 New。

### 断点
  - 条件断点
     满足某个条件时断住。在普通断点上右键，在弹出菜单的 Condition 中填入断点条件即可。如在循环里面需要 i == 5 时使用断点，则在 Condition 输入 i == 5。
  - 临时断点
    执行一次断点后该断点就会消失。在当前行使用快捷键 `(Fn) + Ctrl + Alt + F8`，即可生成一个临时断点，临时断点上有一个数字“1”。
  - 异常断点
    在 Run 菜单打开 View breakpoints 界面，点击右上角的 “+”，选择 Java Exception Breakpoints，并输入要监听的异常即可。如输入 NullPointerException，则在程序运行时不需设置任何断点，只要 App 因为 NullPointerException 异常而导致崩溃，系统就会在对应的地方自动断点并暂停。
  - 日志断点
    当代码写完了，突然出现一个 bug 需要加一行 Log 进行调试，但又不想为了这一行 Log 而重新编译一遍整个工程。此时可以使用日志断点解决这个问题。首先，在需要断点的地方打上一个普通断点，然后在断点处右键，选择 suspend 属性为 false，并在下方的 Log evaluated expression 中写入日志信息即可。如此设置后，在程序运行时则无需重新编译即可在断点处打出日志信息。

### 代码模板
  - 内置模板
  `Ctrl + J` 调出代码模板。这些模板在设置的 Live Templates 标签中。这里不仅提供了 Java 代码的快捷模板，连 Android 注释、Log、甚至 XML 都有非常多的快捷模板。

  - 后缀模板
 `Ctrl + J` 调出代码模板后。如需遍历一个 List 类型的变量 list，只需输入 list.for 快速生成遍历模板、输入 list.cast 快速生成类型转换模板。

  - 自定义模板
  
    **方法注释**

     ①、打开设置，选择 “Live Templates”；②、点击右栏的加号，选择增加一个 Template Group，并在该 Group 下新增一个 Template；③、选中自定义的注释模板，如 ma ，在下方的编辑区域中进行注释代码的编辑，如下代码模板；④、经过这样的配置后，在方法前输入 “ma” 即可弹出该模板，按 Enter 键后确认输入。

   ```Java
     /**
      * $desc$
      *
      * @author zch
      * create at $date$
      */
   ```

    **文件、类注释**
   
   ①、打开设置，选择 “File and Code Templates”；②、选择 Includes 标签，创建名称为 “ClassHeader” 的模板和名称为 “FileHeader” 的模板；③、有了这两个模板就可以组合这些模板来创建新的完整类、文件模板。如在 Files 标签中新建一个名称为 “MyActivity” 的模板文件，并设置代码模板。④、新建文件的时候选择 “MyActivity” 即可创建该种模板的文件。
   
ClassHeader 模板：
    
```Java
/**
 * class description here
 * @author ${USER}
 * @version 1.0.0
 * @since ${YEAR}-${MONTH}-${DAY}
 */
```
    
FileHeader模板：
    
```Java
/*
 * ${NAME}      ${YEAR}-${MONTH}-${DAY}
 * Copyright (c) ${YEAR} jufuns. All right reserved.
 *
 */
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
```
    
MyActivity模板：

```Java
#parse("FileHeader.java")

import android.app.Activity;
import android.os.Bundle;

#parse("ClassHeader.java")
public class ${NAME} extends Activity{

   @Override
    public void onCreate(Bundle savedInstanceState){
           super.onCreate(savedInstanceState);
    }

}
```
 
用 MyActivity 模板新建的 LoginActivity：
	
```Java
/*
 * LoginActivity      2016-10-07
 * Copyright (c) 2016 jufuns. All right reserved.
 *
 */
package com.jiejue.catwalk.ui.ac;

import android.app.Activity;
import android.os.Bundle;

/**
 * class description here
 *
 * @author zch
 * @version 1.0.0
 * @since 2016-10-07
 */
public class LoginActivity extends Activity {

    @Override
   	public void onCreate(Bundle savedInstanceState) {
       	super.onCreate(savedInstanceState);
       	
   	}
   	
}
```
> 类似地，我们也可以建立 Adapter、单例等等的模板代码。

### 立即停止 AndroidStudio 编译

  一个命令就可以停止它！

  只需进入项目文件夹（在 AS 的 terminal 窗口），然后输入以下命令即可。

  ```Java
  mac
  ./gradlew --stop

  window
  gradlew --stop
  ```

  就是这么简单，这个命令会杀死编译的守护进程，编译会立即停止。

## 插件
### .ignore
给 Git 项目生成最合适的 ignore 文件。

### ButterKnife Zelezny
在代码中的布局文件单击鼠标右键，选择 Generate-Generate ButterKnife 即可自动生成 ButterKnife 所需的注解文件。

### SelectorChapek
可将一个 drawable 文件夹下的图像自动生成对应的 drawable selector，只要文件名符合安装要求的规范即可。

### GsonFormat
可将一段 Json 生成所需的 Gson 实体。

### Android Parcelable code generator
可自动生成 Parcelable 接口所需的代码。

### AndroidCodeGenerator
可在 getView 方法中根据布局文件的 ID，快速生成对于的 ViewHolder。

### Prettify
可根据 Layout 自动生成该 Layout 中的 View 在 Java 中的 findViewById 代码。

### Exynap
[Exynap](http://exynap.com/) 一个帮助开发者自动生成样板代码的 AndroidStudio 插件。

### Android Methods Count
高效统计 Android 开源库的方法数。

### AndroidLocalizationer
可用于将项目中的 string 资源自动翻译为其他语言的 Android Studio/IntelliJ IDEA 插件。

### Key Promoter
当用鼠标点击 AS 的一个功能时，Key Promoter 插件会展示该功能的快捷键。

### FindBugs-IDEA
一个免费的 Android Studio 插件，可以在开发早期检测出常见的 Java bug .

### ADB Idea
[ADB Idea](https://github.com/pbreault/adb-idea) 一个开源的 Android Studio 插件，帮助你在 IDE 中实现 app 重启，杀死，清理数据，卸载。

### Codota
[Codota](https://www.codota.com/) 写代码经常会遇到需要从 github 或者 stackoverflow 上寻找代码示例的时候，这个插件可以在无需离开 IDE 就能做这件事情。

## 工具
### Stetho
Stetho 是 Facebook 开发的 Android 调试工具。它可以通过 Chrome 的开发者工具来辅助 Android 开发。提供有网络抓包、查看本地数据（比如 Sqlite 数据库，Sharepreference 等等）、Javascript 控制台、View Hierarchy 布局层级查看、Dump App 等功能。

### Gradle, please
[Gradle, please](http://gradleplease.appspot.com/) 帮助快速地找到第三方库 gradle 依赖的那行代码。比如我们要使用 glide , 只需在一个输入框中输入 glide , 下面就会显示 glide 的完整依赖。当我们搞不清楚库的拼写或者版本号这些细节时，就显得相当有用。

### LeakCanary
LeakCanary 是由 Square 开发的一个开源工具，让复杂的内存泄漏检测变得更简单。它可以在内存泄漏的时候显示通知，并提供一个完整的泄漏轨迹。

### Android Debug Database
Android Debug Database 是一个非常酷的开源工具，完全改变了 debug 数据库和 shared preferences 的方式。现在你可以在一个漂亮的界面上查看，编辑，删除数据，以及运行 sql 语句。

### Android WiFi ADB
[Android WiFi ADB](https://github.com/pedrovgs/AndroidWiFiADB) 可以通过 Wi-Fi  从 Android Studio 运行 app . 无需用数据线把设备和电脑连接起来。

### drawable-optimizer
[drawable-optimizer](https://github.com/fabiomsr/drawable-optimizer) 一个通过优化 PNG 图片来减小 APK 体积的 gradle 插件。

### DevKnox
app 中会有一些难以意识到的安全漏洞，要杜绝这些漏洞往往需要相当的经验和精力。但是这个工具可以帮助你检测安全漏洞，就像使用拼写检查一下简单。**使用方法：选中要查找问题的 java 文件或者文件夹，右键 Devknox Scan -> Devknox Scan 就会开始扫描。**

### ClassyShark
[ClassyShark](https://github.com/google/android-classyshark) 
可以帮助你窥探任何 apk 获得许多有用的信息，比如 classes , resources , manifest ,  dependencies , dex count 等等。它可以让你了解一个 app 时做什么的甚至是如何做到的 。ClassyShark 是开源的。

