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

















