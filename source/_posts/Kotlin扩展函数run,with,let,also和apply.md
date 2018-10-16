---
title: Kotlin扩展函数run,with,let,also和apply
date: 2018-10-16 21:41:00
categories: "Android"
tags:
     - Android
---



[使用](#使用)





## 使用
### run()

**定义**

```Java
/**
 * Calls the specified function [block] and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

**示例**

```Java
fun runTest1() {
    var name = "AA"
    run {
        val name = "BB"
        Log.e(TAG, name) // AA
    }
    Log.d(TAG, name) // BB
}
```

> run 函数在 runTest1 函数中又提供了自己的作用域，并且 run 函数中可以重新定义一个 name 变量，该变量只存在于 run 函数中。

```Java
fun runTest2() {
    var success = true
    var result = run {
        if (success) {
            "200"
        } else {
            "404"
        }
    }
    Log.d(TAG, result) // 200
}
```

> run 函数还有返回值，它**返回作用域中的最后一个对象**。

### T.run()

**定义**

```Java
/**
 * Calls the specified function [block] with `this` value as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

**示例**

```Java
fun tRunTest() {
    val runStr = "ABCDEF".run {
        Log.d(TAG, "字符串的长度为$length") // 字符串的长度为6
        substring(2)
    }
    Log.d(TAG, runStr) // CDEF
}
```

> T.run() 函数中通过 this 来获取 "ABCDEF" 对象，然后输出 length . 该函数也有返回值，它**返回作用域中的最后一个对象**。

### with()

**定义**

```Java
/**
 * Calls the specified function [block] with the given [receiver] as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

**示例**.

```Java
fun withTest() {
    val with = with("ABCDEF") {
        substring(2)
    }
    Log.d(TAG, with) // CDEF
}
```

> with 也是存在一个返回值，它**返回作用域中的最后一个对象**。

### let()

**定义**

```Java
fun letTest() {
    val let = "ABCDEF".let {
        it.substring(2) // 这里需要使用 it
    }
    Log.d(TAG, let) // CDEF
}
```

> 

