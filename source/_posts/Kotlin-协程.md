---
title: Kotlin 协程
date: 2023-08-08 22:25:00
categories: "Kotlin"
tags:
     - Android
     - Kotlin
---

<meta name="referrer" content="no-referrer" />





### 协程是什么

协程基于线程，它是轻量级线程。

- 协程让**异步逻辑同步化**，杜绝回调地狱。
- 协程最核心的点就是，函数或者一段程序能够被**挂起**，稍后再在挂起的位置**恢复**。

### 在 Android 中协程用来解决什么问题

- **处理耗时任务**，这种任务常常会阻塞主线程。
- **保证主线程安全**，即安全地从主线程调用任务 suspend 函数。

### 协程的挂起和恢复

常规函数基础操作包括：invoke（或 call）和 return，协程增加了 suspend 和 resume。

- suspend：也称为挂起或暂停，用于暂停执行当前协程，并保存所有局部变量。
- resume：用于让已暂停的协程从其暂停处继续执行。

### 挂起与阻塞的区别

```kotlin
GlobalScope.launch(Dispatchers.Main) {
    // 挂起
    delay(5000)
    Log.d("tag", "${Thread.currentThread().name}:after delay.")
}

// 阻塞
Thread.sleep(5000)
Log.d("tag", "${Thread.currentThread().name}:after sleep.")
```

### 挂起函数

- 使用 suspend 关键字修饰的函数叫做挂起函数。
- 挂起函数只能在**协程体内**或**其它挂起函数内**调用。

### 协程的两部分

Kotlin 的协程实现分为两个层次：

- 基础设施层，标准库的协程 API，主要对协程提供了概念和语义上最基本的支持。
- 业务框架层，协程的上层框架支持。

基础设施层的一个 demo：

```kotlin
import kotlin.coroutines.*

// 协程体
val continuation = suspend {
    5
}.createCoroutine(object : Continuation<Int> {

    override val context: CoroutineContext = EmptyCoroutineContext

    override fun resumeWith(result: Result<Int>) {
        println("Coroutine End: $result")
    }
})

continuation.resume(Unit)
```

基础设施层使用的是 `kotlin.coroutines.*` 而业务框架层使用的是 `kotlinx.coroutines.*`。

### 调度器

所有协程必须在调度器中运行，即使它们在主线程上运行也是如此。

- **Dispatchers.Main**。Android 上的主线程，用来处理 UI 交互和一些轻量级任务（调用 suspend 函数；调用 UI 函数；更新 LiveData）。

- **Dispatchers.IO**。非主线程，专为磁盘和网络 IO 进行了优化（数据库；文件读写；网络处理）。

- **Dispatchers.Default**。非主线程，专为 CPU 密集型任务进行了优化（数组排序；JSON 数据解析；处理差异判断）。

### 任务泄漏

- 当某个协程任务丢失，无法追踪，会导致内存、CPU、磁盘等资源浪费，甚至发送一个无用的网络请求，这种情况称为任务泄漏。
- 为了能够避免协程泄漏，Kotlin 引入了结构化并发机制。

### 结构化并发

使用结构化并发可以做到：

- 取消任务，当某项任务不再需要时，取消它。
- 追踪任务，当任务正在执行时，追踪它。
- 发出错误信号，当协程失败时，发出错误信号表明有错误发生。

### CoroutineScope

1、定义协程必须指定其 CoroutineScope，它会跟踪所有协程，同样它还可以**取消由它所启动的所有协程**。

2、常用的相关 API 有：

- **GlobalScope**，生命周期是 process 级别的，即使 Activity 或 Fragment 已经被销毁，协程仍然在执行。
- **MainScope**，在 Activity 中使用，可以在 onDestroy() 中取消协程。
- **viewModelScope**，只能在 ViewModel 中使用，绑定 ViewModel 生命周期。
- **lifecycleScope**，只能在 Activity、Fragment 中使用，会绑定 Activity 和 Fragment 的生命周期。

### MainScope 使用

```kotlin
class CoroutineActivity : AppCompatActivity(), CoroutineScope by MainScope() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_coroutine)

        submit()
    }

    private fun submit() {
        btnSubmit?.setOnClickListener {
            launch {
                try {
                    delay(5000)
                } catch (e: Exception) {
                    /**
                     * 取消协程会抛出异常
                     * kotlinx.coroutines.JobCancellationException: Job was cancelled; job=SupervisorJobImpl{Cancelling}@7e7375b
                     */
                    e.printStackTrace()
                }
            }
        }
    }

    override fun onDestroy() {
        cancel()
        super.onDestroy()
    }
}
```

### 协程构建器

launch 和 async 构建器都可以用来启动新协程：

- **launch**，返回一个 Job 并且不附带任何结果值。
- **async**，返回一个 Deferred，Deferred 也是一个 Job，可以使用 .await() 在一个延期的值上得到它的最终结果。

等待一个作业：

- join 和 await。
- 组合并发。

测试构建器：

```kotlin
@Test
fun `coroutine builder`() = runBlocking {
    val job1 = launch {
        delay(200)
        println("job1 finished.")
    }

    val job2 = async {
        delay(200)
        println("job2 finished.")
        "job2 result"
    }
    println(job2.await())
}

// 打印结果
job1 finished.
job2 finished.
job2 result
```

> runBlocking 把当前线程包装成一个协程，它会阻塞当前线程等待子协程执行完再结束。

### join 和 await 等待协程作业

测试 join：

```kotlin
@Test
fun `coroutine join`() = runBlocking {
    val job1 = launch {
        delay(2000)
        println("One")
    }
    job1.join()
    val job2 = launch {
        delay(200)
        println("Two")
    }
    val job3 = launch {
        delay(200)
        println("Three")
    }
}

// 打印结果
One
Two
Three
```

测试 await：

```kotlin
@Test
fun `coroutine await`() = runBlocking {
    val job1 = async {
        delay(2000)
        println("One")
    }
    job1.await()
    val job2 = async {
        delay(200)
        println("Two")
    }
    val job3 = async {
        delay(200)
        println("Three")
    }
}

// 打印结果
One
Two
Three
```

### async 组合并发

测试同步 sync：

```kotlin
@Test
fun `coroutine sync`() = runBlocking {
    val time = measureTimeMillis {
        val one = doOne()
        val two = doTwo()
        println("The result:${one + two}")
    }
    println("Completed in $time ms")
}

private suspend fun doOne(): Int {
    delay(1000)
    return 1
}

private suspend fun doTwo(): Int {
    delay(1000)
    return 1
}

// 打印结果
The result:2
Completed in 2018 ms
```

测试异步 async：

```kotlin
@Test
fun `coroutine async`() = runBlocking {
    val time = measureTimeMillis {
        val one = async {
            doOne()
        }
        val two = async {
            doTwo()
        }
        println("The result:${one.await() + two.await()}")

        /* 以下形式，结果是 2m。
        val one = async {
            doOne()
        }.await()
        val two = async {
            doTwo()
        }.await()
        println("The result:${one + two}")*/
    }
    println("Completed in $time ms")
}

// 打印结果
The result:2
Completed in 1034 ms
```

### 协程的启动模式

- **DEFAULT**：协程创建后，立即开始调度，在调度前如果协程被取消，其将直接进入取消响应的状态。

  ```kotlin
  @Test
  fun `start mode DEFAULT`() = runBlocking {
      val job = launch(start = CoroutineStart.DEFAULT) {
          delay(5000)
          println("Job finished.")
      }
      delay(1000)
      job.cancel()
  }
  
  // 什么也不打印
  ```

- **ATOMIC**：协程创建后，立即开始调度，协程执行到第一个挂起点之前不响应取消。

  ```kotlin
  @Test
  fun `start mode ATOMIC`() = runBlocking {
      val job = launch(start = CoroutineStart.ATOMIC) {
          var count = 0
          val time = measureTimeMillis {
              for (i in 0 until 2100000000) {
                  if (i % 2 == 0) {
                      count++
                  }
              }
          }
          println("Before delay $time ms. count=$count")
          delay(5000)
          println("Job finished.")
      }
      delay(1000)
      job.cancel()
  }
  
  // 打印结果
  Before delay 1513 ms, count=1050000000.
  ```

- **LAZY**：只有协程被需要时，包括主动调用协程的 start、join 或者 await 等函数时才会开始调度，如果调度前就被取消，那么该协程将直接进入异常结束状态。

  ```kotlin
  @Test
  fun `start mode LAZY`() = runBlocking {
      val job = async (start = CoroutineStart.LAZY) {
          delay(2000)
          println("Job finished.")
      }
      // 调度前就取消，会抛出异常 kotlinx.coroutines.JobCancellationException
      job.cancel()
      job.await()
  }
  ```

- **UNDISPATCHED**：协程创建后立即在**当前函数调用栈**中执行，直到遇到第一个真正挂起的点。

  ```kotlin
  @Test
  fun `start mode UNDISPATCHED`() = runBlocking {
      val job = async(context = Dispatchers.IO, start = CoroutineStart.UNDISPATCHED) {
          println("thread1: ${Thread.currentThread().name}")
          delay(1000)
          println("thread2: ${Thread.currentThread().name}")
      }
  }
  
  // 打印结果
  thread1: Test worker @coroutine#2
  thread2: DefaultDispatcher-worker-1 @coroutine#2
  ```


### 协程的作用域构建器

#### coroutineScope 与 runBlocking

- runBlocking 是常规函数，而 coroutineScope 是挂起函数。
- 它们都会等待其协程体结束，主要区别在于 runBlocking 会阻塞当前线程来等待，而 coroutineScope 只是挂起，会释放底层线程用于其它用途。

```kotlin
@Test
fun `coroutine coroutineScope builder`() = runBlocking {
    coroutineScope {
        val job1 = launch {
            delay(400)
            println("job1 finished.")
        }
        val job2 = async {
            delay(200)
            println("job2 finished.")
            throw IllegalArgumentException()
        }
    }
}

// 打印结果
job2 finished.

java.lang.IllegalArgumentException...
```

#### coroutineScope 与 supervisorScope

- coroutineScope：一个协程失败了，所有其它兄弟协程也会被取消。
- supervisorScope：一个协程失败了，不会影响其它兄弟协程。

```kotlin
@Test
fun `coroutine supervisorScope builder`() = runBlocking {
    supervisorScope {
        val job1 = launch {
            delay(400)
            println("job1 finished.")
        }
        val job2 = async {
            delay(200)
            println("job2 finished.")
            throw IllegalArgumentException()
        }
    }
}

// 打印结果
job2 finished.
job1 finished.
```

### Job 对象

- 对于每一个创建的协程（通过 launch 或者 async），会返回一个 Job 实例，该实例是协程的唯一标示，并且负责管理协程的生命周期。
- 一个任务可以包含一系列状态：新创建（**New**）、活跃（**Active**）、完成中（**Completing**）、已完成（**Completed**）、取消中（**Cancelling**）和已取消（**Cancelled**）。虽然我们无法直接访问这些状态，但是我们可以访问 Job 的属性：isActive、isCancelled 和 isCompleted。

### Job 的生命周期

如果协程处于活跃状态，协程运行出错或者调用 job.cancel() 都会将当前任务置为取消中（Cancelling）状态（isActive = false，isCancelled = true）。当所有的子协程都完成后，协程会进入已取消（Cancelled）状态，此时 isCompleted = true。

![](https://gitee.com/zch0304/images/raw/master/note/676350c3f83e92cda2f87c2b90471451.png) 

### 协程的取消

- 取消作用域会取消它的子协程。

  ```kotlin
  @Test
  fun `scope cancel`() = runBlocking {
      val scope = CoroutineScope(Dispatchers.Default)
      scope.launch {
          delay(1000)
          println("job1.")
      }
      scope.launch {
          delay(1000)
          println("job2.")
      }
      delay(100)
      scope.cancel()
      delay(2000)
  }
  
  // 无打印结果
  ```

- 被取消的子协程不会影响其余兄弟协程。

  ```kotlin
  @Test
  fun `brother cancel`() = runBlocking {
      val scope = CoroutineScope(Dispatchers.Default)
      val job1 = scope.launch {
          delay(1000)
          println("job1.")
      }
      val job2 = scope.launch {
          delay(1000)
          println("job2.")
      }
      delay(100)
      job1.cancel()
      delay(2000)
  }
  
  // 打印结果
  job2.
  ```

- 协程通过抛出一个特殊的异常 CancellationException 来处理取消操作。

  ```kotlin
  @Test
  fun `CancellationException`() = runBlocking {
      val job1 = GlobalScope.launch {
          try {
              delay(1000)
              println("job1.")
          } catch (e: Exception) {
              e.printStackTrace()
          }
      }
      delay(100)
      /* job1.cancel(CancellationException("取消"))
       job1.join()*/
      job1.cancelAndJoin()
  }
  
  // 打印结果
  kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled;
  ```

- 所有 kotlinx.coroutines 中的挂起函数（withContext, delay 等）都是可取消的。

### CPU 密集型任务取消

- isActive 是一个可以被使用在 CoroutineScope 中的扩展属性，检查 Job 是否处于活跃状态。

  ```kotlin
  @Test
  fun `cancel cpu task by isActive`() = runBlocking {
      val job = launch(Dispatchers.Default) {
          var nextPrintTime = System.currentTimeMillis()
          var i = 0
          while (i < 5 && isActive) {
              if (System.currentTimeMillis() >= nextPrintTime) {
                  println("job: I'm sleeping ${i++} ...")
                  nextPrintTime += 500
              }
          }
      }
      delay(1300)
      println("main: I'm tired of waiting!")
      job.cancelAndJoin()
      println("main: Now I can quit.")
  }
  
  // 打印结果
  job: I'm sleeping 0 ...
  job: I'm sleeping 1 ...
  job: I'm sleeping 2 ...
  main: I'm tired of waiting!
  main: Now I can quit.
  ```

- ensureActive()，如果 Job 处于非活跃状态，这个方法会抛出异常。

  ```kotlin
  @Test
  fun `cancel cpu task by ensureActive`() = runBlocking {
      val job = launch(Dispatchers.Default) {
          var nextPrintTime = System.currentTimeMillis()
          var i = 0
          while (i < 5) {
              ensureActive() // 抛出 CancellationException 异常，但被静默处理掉了。
              if (System.currentTimeMillis() >= nextPrintTime) {
                  println("job: I'm sleeping ${i++} ...")
                  nextPrintTime += 500
              }
          }
      }
      delay(1300)
      println("main: I'm tired of waiting!")
      job.cancelAndJoin()
      println("main: Now I can quit.")
  }
  
  // 打印结果
  job: I'm sleeping 0 ...
  job: I'm sleeping 1 ...
  job: I'm sleeping 2 ...
  main: I'm tired of waiting!
  main: Now I can quit.
  ```

- yield 函数会检查所在协程的状态，如果已经取消，则抛出 CancellationException 予以响应。此外，它还会尝试出让线程的执行权，给其它协程提供执行机会。

  ```kotlin
  @Test
  fun `cancel cpu task by yield`() = runBlocking {
      val job = launch(Dispatchers.Default) {
          var nextPrintTime = System.currentTimeMillis()
          var i = 0
          while (i < 5) {
              yield()
              if (System.currentTimeMillis() >= nextPrintTime) {
                  println("job: I'm sleeping ${i++} ...")
                  nextPrintTime += 500
              }
          }
      }
      delay(1300)
      println("main: I'm tired of waiting!")
      job.cancelAndJoin()
      println("main: Now I can quit.")
  }
  
  // 打印结果
  job: I'm sleeping 0 ...
  job: I'm sleeping 1 ...
  job: I'm sleeping 2 ...
  main: I'm tired of waiting!
  main: Now I can quit.
  ```

### 协程取消的副作用

- 在 finally 中释放资源。

  ```kotlin
  @Test
  fun `release resources`() = runBlocking {
      val job = launch {
          try {
              repeat(1000) { i ->
                  println("job: I'm sleeping $i ...")
                  delay(500)
              }
          } finally {
              println("job: I'm running finally.")
          }
      }
      delay(1300)
      println("main: I'm tired of waiting!")
      job.cancelAndJoin()
      println("main: Now I can quit.")
  }
  
  // 打印结果
  job: I'm sleeping 0 ...
  job: I'm sleeping 1 ...
  job: I'm sleeping 2 ...
  main: I'm tired of waiting!
  job: I'm running finally.
  main: Now I can quit.
  ```

- use 函数：该函数只能被实现了 Closeable 的对象使用，程序结束的时候会自动调用 close 方法，适合文件对象。

  ```kotlin
  @Test
  fun `use function`() = runBlocking {
      /*val br = BufferedReader(FileReader("E:\\Hello.txt"))
      var line: String?
      try {
          while (true) {
              line = br.readLine() ?: break
              println(line)
          }
      } catch (e: Exception) {
      } finally {
          try {
              br.close()
          } catch (e: Exception) {
          }
      }*/
  
      BufferedReader(FileReader("E:\\Hello.txt")).use {
          var line: String?
          while (true) {
              line = it.readLine() ?: break
              println(line)
          }
      }
  }
  ```

### 不能取消的任务

处于取消中状态的协程不能够挂起（运行不能取消的代码），当协程被取消后需要调用挂起函数，我们需要将清理任务的代码放置于 NonCancellable CoroutineContext 中。这样会挂起运行中的代码，并保持协程取消中状态直到任务处理完成。

```kotlin
@Test
fun `cancel with NonCancelable`() = runBlocking {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500)
            }
        } finally {
            withContext(NonCancellable) {
                println("job: I'm running finally.")
                delay(1000)
                println("job: And I've just delayed for 1 sec because I'm non-cancellable.")
            }
        }
    }
    delay(1300)
    println("main: I'm tired of waiting!")
    job.cancelAndJoin()
    println("main: Now I can quit.")
}

// 打印结果
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
job: I'm running finally.
job: And I've just delayed for 1 sec because I'm non-cancellable.
main: Now I can quit.
```

> NonCancellable 可用于常驻任务。

### 超时任务

- 很多情况下取消一个协程的理由是它有可能超时。用 withTimeout 进行超时操作，如果规定时间内未完成任务则会抛出异常。

  ```kotlin
  @Test
  fun `deal with withTimeout`() = runBlocking {
      withTimeout(1300) {
          repeat(1000) { i ->
              println("job: I'm sleeping $i ...")
              delay(500)
          }
      }
  }
  
  // 打印结果
  job: I'm sleeping 0 ...
  job: I'm sleeping 1 ...
  job: I'm sleeping 2 ...
  
  Timed out waiting for 1300 ms
  kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1300 ms
  ```

- withTimeoutOrNull 通过返回 null 来进行超时操作，从而替代抛出一个异常。

  ```kotlin
  @Test
  fun `deal with withTimeoutOrNull`() = runBlocking {
      val result = withTimeoutOrNull(1300) {
          repeat(1000) { i ->
              println("job: I'm sleeping $i ...")
              delay(500)
          }
          "Done"
      } ?: "Undone"
      println("Result is $result.")
  }
  
  // 打印结果
  job: I'm sleeping 0 ...
  job: I'm sleeping 1 ...
  job: I'm sleeping 2 ...
  Result is Undone.
  ```


### 协程的上下文

CoroutineContext 是一组用于定义协程行为的元素，它由如下几项构成：

- **Job**：控制协程的生命周期。
- **CoroutineDispatcher**：向合适的线程分发任务。
- **CoroutineName**：协程的名称，调试的时候很有用。
- **CoroutineExceptionHandler**：处理未被捕获的异常。

#### 组合上下文中的元素

有时我们需要在协程上下文中定义多个元素，我们可以使用 "+" 操作符来实现。比如，我们可以显示指定一个调度器来启动协程并且同时显示指定一个命名：

```kotlin
@Test
fun `CoroutineContext`() = runBlocking<Unit> {
    launch(Dispatchers.Default + CoroutineName("hello")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}

// 打印结果
I'm working in thread DefaultDispatcher-worker-1 @hello#2
```

#### 协程上下文的继承

对于新创建的协程，它的 CoroutineContext 会包含一个全新的 Job 实例，它会帮助我们控制协程的生命周期。而**剩下的元素会从 CoroutineContext 的父类继承**，该父类可能是另外一个协程或者创建该协程的 CoroutineScope。

```kotlin
@Test
fun `CoroutineContext extend`() = runBlocking<Unit> {
    val scope = CoroutineScope(Job() + Dispatchers.IO + CoroutineName("hello"))
    val job = scope.launch {
        // 新的协程会将 CoroutineScope 作为父级
        println("${coroutineContext[Job]} ${Thread.currentThread().name}")
        val result = async {
             // 通过 async 创建的新协程会将当前协程作为父级
            println("${coroutineContext[Job]} ${Thread.currentThread().name}")
            "OK"
        }.await()
    }
    job.join()
}

// 打印结果
"hello#2":StandaloneCoroutine{Active}@55b8c583 DefaultDispatcher-worker-1 @hello#2
"hello#3":DeferredCoroutine{Active}@64e55ef0 DefaultDispatcher-worker-3 @hello#3
```

#### 协程上下文的公式

协程上下文 = **默认值** + **继承的 CoroutineContext** + **参数**。

- 一些元素包含默认值：Dispatchers.Default 是默认的 CoroutineDispatcher，以及 "coroutine" 作为默认的 CoroutineName。
- 继承的 CoroutineContext 是 CoroutineScope 或者其父协程的 CoroutineContext。
- 传入协程构建器的参数的优先级高于继承的上下文，隐藏会覆盖对应参数值。

```kotlin
@Test
fun `CoroutineContext extend2`() = runBlocking<Unit> {
    val coroutineExceptionHandler = CoroutineExceptionHandler { _, excption ->
        println("CoroutineExceptionHandler got $excption")
    }
    val scope = CoroutineScope(Job() + Dispatchers.Main + coroutineExceptionHandler)
    // 新的 CoroutineContext = 父级 CoroutineContext + Job()
    val job = scope.launch(Dispatchers.IO) {
        // 新协程
        println("${coroutineContext[Job]} ${Thread.currentThread().name}")
    }
    job.join()
}

// 打印结果
"coroutine#2":StandaloneCoroutine{Active}@78ec7afa DefaultDispatcher-worker-1 @coroutine#2
```

### 协程的异常处理

#### 异常处理的必要性

当应用出现一些意外情况时，给用户提供合适的体验非常重要。一方面，目睹应用崩溃是个很糟糕的体验，另一方面，当用户操作失败时，也必须要能给出正确的提示信息。

#### 异常的传播

协程构建器有两种形式：**自动传播异常**（launch 与 actor），**向用户暴露异常**（async 与 produce）。当这些构建器用于创建一个**根协程**时（该协程不是另一个协程的子协程），前者这类构建器，异常会在它发生的第一时间被抛出，而后者则依赖用户来最终消费异常，例如通过 await 或 receive。

```kotlin
@Test
fun `exception propagation`() = runBlocking<Unit> {
    val job = GlobalScope.launch {
        try {
            throw IndexOutOfBoundsException()
        } catch (e: Exception) {
            println("Caught IndexOutOfBoundsException")
        }
    }
    job.join()

    val deferred = GlobalScope.async {
        throw ArithmeticException()
    }
    try {
        deferred.await()
    } catch (e: Exception) {
        println("Caught ArithmeticException")
    }
}

// 打印结果
Caught IndexOutOfBoundsException
Caught ArithmeticException
```

#### 非根协程的异常

其它协程所创建的协程中，产生的异常总是会被传播。

```kotlin
@Test
fun `exception propagation2`() = runBlocking<Unit> {
    val scope = CoroutineScope(Job())
    val job = scope.launch {
        async {
            // 如果 async 抛出异常，launch 就会立刻抛出异常，而不会调用 .await()
            throw IllegalArgumentException()
        }
    }
    job.join()
}

// 打印结果
Exception in thread "DefaultDispatcher-worker-1 @coroutine#3" java.lang.IllegalArgumentException
```

#### 异常的传播特性

当一个协程由于一个异常而运行失败时，它会传播这个异常并传递给它的父级。接下来，父级会进行下面几步操作：

- 取消它的子级。
- 取消它自己。
- 将异常传播并传递给它的父级。

#### SupervisorJob

- 使用 SupervisorJob 时，一个子协程的运行失败不会影响到其它子协程。SupervisorJob 不会传播异常给它的父级，它会**让子协程自己处理异常**。
- 这种需求常见于在作用域内定义作业的 UI 组件，如果任一个 UI 的子作业执行失败了，它并不总是有必要取消整个 UI 组件，但是如果 UI 组件被销毁了，由于它的结果不再被需要了，它就有必要使所有的子作业执行失败。

```kotlin
@Test
fun `test SupervisorJob`() = runBlocking<Unit> {
    val supervisor = CoroutineScope(SupervisorJob())
    val job1 = supervisor.launch {
        delay(100)
        println("child 1.")
        throw IllegalArgumentException()
    }
    val job2 = supervisor.launch {
        try {
            delay(Long.MAX_VALUE)
        } finally {
            println("child 2 finished.")
        }
    }
//        delay(200)
//        supervisor.cancel()
    joinAll(job1, job2)
}

// 打印结果
child 1.
Exception in thread "DefaultDispatcher-worker-2 @coroutine#2" java.lang.IllegalArgumentException
一直运行中......
```

#### supervisorScope

当作业自身执行失败的时候，所有子作业将会全部被取消。

```kotlin
@Test
fun `test supervisorScope`() = runBlocking<Unit> {
    supervisorScope {
        launch {
            delay(100)
            println("child 1.")
            throw IllegalArgumentException()
        }
        try {
            delay(Long.MAX_VALUE)
        } finally {
            println("child 2 finished.")
        }
    }
}

// 打印结果
child 1.
Exception in thread "Test worker @coroutine#2" java.lang.IllegalArgumentException
一直运行中......
```

```kotlin
@Test
fun `test supervisorScope2`() = runBlocking<Unit> {
    supervisorScope {
        launch {
            try {
                println("The child is sleeping.")
                delay(Long.MAX_VALUE)
            } finally {
                println("The child is cancelled.")
            }
        }
        yield()
        println("Throwing an exception from the scope.")
        throw AssertionError()
    }
}

// 打印结果
The child is sleeping.
Throwing an exception from the scope.
The child is cancelled.

java.lang.AssertionError
```

#### 异常的捕获

- 使用 CoroutineExceptionHandler 对协程的异常进行捕获。
- 以下的条件被满足时，异常就会被捕获。
  - **时机**：异常是被自动抛出异常的协程所抛出的（使用 launch，而不是 async 时）。
  - **位置**：在 CoroutineScope 的 CoroutineContext 中或在一个根协程（CoroutineScope 或者 supervisorScope 的直接子协程）中。

```kotlin
@Test
fun `test coroutineExceptionHandler`() = runBlocking<Unit> {
    val handler = CoroutineExceptionHandler { _, excption ->
        println("Caught $excption")
    }
    val job = GlobalScope.launch(handler) {
        throw AssertionError()
    }
    val deferred = GlobalScope.async(handler) {
        throw ArithmeticException()
    }
    job.join()
    deferred.await()
}

// 打印结果
Caught java.lang.AssertionError

java.lang.ArithmeticException
```

```kotlin
@Test
fun `test coroutineExceptionHandler2`() = runBlocking<Unit> {
    val handler = CoroutineExceptionHandler { _, excption ->
        println("Caught $excption")
    }
    val scope = CoroutineScope(Job())
    val job = scope.launch(handler) {
        launch {
            throw ArithmeticException()
        }
    }
    job.join()
}

// 打印结果
Caught java.lang.ArithmeticException
```

```kotlin
@Test
fun `test coroutineExceptionHandler3`() = runBlocking<Unit> {
    val handler = CoroutineExceptionHandler { _, excption ->
        println("Caught $excption")
    }
    val scope = CoroutineScope(Job())
    val job = scope.launch {
        // handler 放这里捕获不到异常
        launch(handler) {
            throw ArithmeticException()
        }
    }
    job.join()
}

// 打印结果
Exception in thread "DefaultDispatcher-worker-1 @coroutine#3" java.lang.ArithmeticException
```

#### Android 中全局异常处理

- 全局异常处理器可以获取到所有协程未处理的未捕获异常，不过它并不能对异常进行捕获，虽然**不能阻止程序崩溃**，全局异常处理器在程序调试和异常上报等场景中仍然有非常大的用处。
- 我们需要在 classpath 下面创建 META-INF/services 目录，并在其中创建一个名为 kotlinx.coroutines.CoroutineExceptionHandler 的文件，文件内容就是我们的全局异常处理器的全类名。

正常情况捕获异常（程序不崩溃）：

```kotlin
val handler = CoroutineExceptionHandler { _, excption ->
    Log.d("tag", "Caught $excption")
}

btnGlobalCoroutineExceptionHandler?.setOnClickListener {
    GlobalScope.launch(handler) {
        "abc".substring(10)
    }
}

// 打印结果
D/tag: Caught java.lang.StringIndexOutOfBoundsException: length=3; index=10
```

全局异常处理器获取未被捕获异常（程序崩溃）：

```kotlin
btnGlobalCoroutineExceptionHandler?.setOnClickListener {
    GlobalScope.launch {
        "abc".substring(10)
    }
}

// 打印结果
D/tag: Unhandled Coroutine Exception: java.lang.StringIndexOutOfBoundsException: length=3; index=10
    
    --------- beginning of crash
E/AndroidRuntime: FATAL EXCEPTION: DefaultDispatcher-worker-1
    Process: com.zch.kotlin, PID: 3436
    java.lang.StringIndexOutOfBoundsException: length=3; index=10
        at java.lang.String.substring(String.java:1899)
```

#### 取消与异常

- 取消与异常紧密相关，协程内部使用 CancellationException 来进行取消，这个异常会被忽略。
- 当子协程被取消时，不会取消它的父协程。
- 如果一个协程遇到了 CancellationException 以外的异常，它将使用该异常取消它的父协程。当父协程的所有子协程都结束后，异常才会被父协程处理。

```kotlin
@Test
fun `test cancel and exception`() = runBlocking<Unit> {
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            }finally {
                println("Child is cancelled.")
            }
        }
        yield()
        println("Cancelling child.")
        child.cancelAndJoin()
        yield()
        println("Parent is not cancelled.")
    }
    job.join()
}

// 打印结果
Cancelling child.
Child is cancelled.
Parent is not cancelled.
```

```kotlin
@Test
fun `test cancel and exception2`() = runBlocking<Unit> {
    val handler = CoroutineExceptionHandler { _, excption ->
        println("Caught $excption")
    }
    val job = GlobalScope.launch(handler) {
        launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                withContext(NonCancellable) {
                    println("Children are cancelled, but exception is not handled until all children termination.")
                    delay(100)
                    println("The first child finished its non cancellable block.")
                }
            }
        }
        launch {
            delay(10)
            println("Second child throws an exception.")
            throw ArithmeticException()
        }
    }
    job.join()
}

// 打印结果
Second child throws an exception.
Children are cancelled, but exception is not handled until all children termination.
The first child finished its non cancellable block.
Caught java.lang.ArithmeticException
```

#### 异常聚合

当协程的多个子协程因为异常而失败时，一般情况下取第一个异常进行处理。在第一个异常之后发生的所有其它异常，都将被绑定到第一个异常之上。

```kotlin
@Test
fun `test exception aggreation`() = runBlocking<Unit> {
    val handler = CoroutineExceptionHandler { _, excption ->
        println("Caught $excption ${excption.suppressed.contentToString()}")
    }
    val job = GlobalScope.launch(handler) {
        launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                throw ArithmeticException()
            }
        }
        launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                throw IndexOutOfBoundsException()
            }
        }
        launch {
            delay(100)
            throw IOException()
        }
    }
    job.join()
}

// 打印结果
Caught java.io.IOException [java.lang.IndexOutOfBoundsException, java.lang.ArithmeticException]
```

### Flow

#### 异步返回多个值的方案

集合、序列、挂起函数、Flow。

```kotlin
// 返回了多个值，但不是异步
private fun simpleList() = listOf(1, 2, 3)

// 返回了多个值，但不是异步（一次返回一个值）
private fun simpleSequence() = sequence {
    for (i in 1..3) {
        Thread.sleep(1000) // 阻塞
//            delay(1000) // 不能用
        yield(i)
    }
}

// 返回了多个值，是异步（一次性返回了多个值）
private suspend fun simpleList2(): List<Int> {
    delay(1000)
    return listOf(1, 2, 3)
}

// 返回了多个值，是异步（一次返回一个值）
private fun simpleFlow() = flow {
    for (i in 1..3) {
        delay(1000) // 模拟耗时操作
        emit(i) // 发射，返回一个元素
    }
}

@Test
fun `test multiple values`() {
//        simpleList().forEach { println(it) }
    simpleSequence().forEach { println(it) }
}

@Test
fun `test multiple values2`() = runBlocking {
    simpleList2().forEach { println(it) }
}

@Test
fun `test multiple values3`() = runBlocking {
    // 启动另外一个协程，证明 simpleFlow 没有阻塞主线程
    launch {
        for (i in 1..3) {
            println("I'm not blocked $i.")
            delay(1500)
        }
    }
    simpleFlow().collect { println(it) }
}
```

#### Flow 与其它方式的区别

- 名为 flow 的 Flow 类型构建器函数。
- flow{...} 构建块中的代码可以挂起。
- 函数 simpleFlow 不再标有 suspend 修饰符。
- 流使用 emit 函数发射值。
- 流使用 collect 函数收集值。

![](https://gitee.com/zch0304/images/raw/master/note/1663391441985.png) 

#### 冷流

Flow 是一种类似于序列的冷流，flow 构建器中的代码直到流被收集的时候才运行。

```kotlin
private fun simpleFlow2() = flow {
    println("Flow started.")
    for (i in 1..3) {
        delay(1000)
        emit(i)
    }
}

@Test
fun `test flow is cold`() = runBlocking {
    val flow = simpleFlow2()
    println("Calling collect...")
    flow.collect { println(it) }
    println("Calling collect again...")
    flow.collect { println(it) }
}

// 打印结果
Calling collect...
Flow started.
1
2
3
Calling collect again...
Flow started.
1
2
3
```

#### 流的连续性

- 流的每次单独收集都是按顺序执行的，除非使用特殊操作符。
- 从上游到下游每个过渡操作符都会处理每个发射出的值，然后再交给末端操作符。

```kotlin
@Test
fun `test flow continuation`() = runBlocking {
    (1..5).asFlow().filter {
        it % 2 == 0
    }.map {
        "string $it"
    }.collect {
        println("Collect $it.")
    }
}

// 打印结果
Collect string 2.
Collect string 4.
```

#### 流构建器

- flowOf 构建器定义了一个发射固定值集的流。
- 使用 .asFlow() 扩展函数，可以将各种集合与序列转换为流。

```kotlin
@Test
fun `test flow builder`() = runBlocking {
    flowOf("one", "two", "three")
        .onEach { delay(1000) }
        .collect { println(it) }

    (1..3).asFlow().collect { println(it) }
}
```

#### 流上下文

- 流的收集总是在调用协程的上下文中发生，流的该属性称为**上下文保存**。
- flow{...} 构建器中的代码必须遵循上下文保存属性，并且不允许从其它上下文中发射（emit）。
- flowOn 操作符，该函数用于更改流发射的上下文。

```kotlin
private fun simpleFlow3() = flow {
    println("Flow started ${Thread.currentThread().name}.")
    for (i in 1..3) {
        delay(1000)
        emit(i)
    }
}.flowOn(Dispatchers.Default)

@Test
fun `test flow on`() = runBlocking {
    simpleFlow3().collect { println("Collected $it ${Thread.currentThread().name}") }
}

// 打印结果
Flow started DefaultDispatcher-worker-1 @coroutine#2.
Collected 1 Test worker @coroutine#1
Collected 2 Test worker @coroutine#1
Collected 3 Test worker @coroutine#1
```

> 如果去掉 .flowOn(Dispatchers.Default)，那么发射和收集都是在 Test worker 线程中。

#### 在指定协程中收集流

使用 launchIn 替换 collect，我们可以在单独的协程中启动流的收集。

```kotlin
private fun events() = (1..3).asFlow().onEach { delay(100) }.flowOn(Dispatchers.Default)

@Test
fun `test flow launchIn`() = runBlocking {
    val job = events()
        .onEach { println("Event: $it ${Thread.currentThread().name}") }
//            .launchIn(CoroutineScope(Dispatchers.IO))
        .launchIn(this)

//        delay(200)
//        job.cancelAndJoin()
}

// 打印结果
Event: 1 Test worker @coroutine#2
Event: 2 Test worker @coroutine#2
Event: 3 Test worker @coroutine#2
```

#### 流的取消

流采用与协程同样的协作取消。像往常一样，流的收集可以是当流在一个可取消的挂起函数（例如 delay）中挂起的时候取消。

```kotlin
private fun simpleFlow4() = flow {
    for (i in 1..3) {
        delay(1000)
        println("Emitting $i.")
        emit(i)
    }
}

@Test
fun `test cancel flow`() = runBlocking {
    withTimeoutOrNull(2500) {
        simpleFlow4().collect { println(it) }
    }
    println("Done.")
}

// 打印结果
Emitting 1.
1
Emitting 2.
2
Done.
```

#### 流的取消检测

- 为方便起见，流构建器对每个发射值执行附加的 ensureActive 检测以进行取消，这意味着从 flow{...} 发出的繁忙循环是可以取消的。

  ```kotlin
  private fun simpleFlow5() = flow {
      for (i in 1..5) {
          println("Emitting $i.")
          emit(i)
      }
  }
  
  @Test
  fun `test cancel flow check`() = runBlocking {
      simpleFlow5().collect {
          println(it)
          if (it == 3) cancel()
      }
  }
  
  // 打印结果
  Emitting 1.
  1
  Emitting 2.
  2
  Emitting 3.
  3
  
  BlockingCoroutine was cancelled
  kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job="coroutine#1":BlockingCoroutine{Cancelled}@5da3da3d
  ```

- 出于性能原因，大多数其它流操作不会自动执行其它取消检测，在协程处于繁忙循环的情况下，必须明确检测是否取消，通过 **cancellable** 操作符来执行此操作。

  ```kotlin
  @Test
  fun `test cancel flow check`() = runBlocking {
      (1..5).asFlow().cancellable().collect {
          println(it)
          if (it == 3) cancel()
      }
  }
  
  // 打印结果
  1
  2
  3
  
  BlockingCoroutine was cancelled
  kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job="coroutine#1":BlockingCoroutine{Cancelled}@63bbd396
  ```

#### 背压

**1、使用缓冲与 flowOn 处理背压**

- buffer()：并发运行流中发射元素的代码。

- 当必须更改 CoroutineDispatcher 时，flowOn 操作符使用了相同的缓冲机制，但是 buffer 函数显示地请求缓冲而**不改变执行上下文**。

**2、合并与处理最新值**

- conflate()：合并发射项，不对每个值进行处理。
- collectLastest()：取消并重新发射最后一个值。

```kotlin
private fun simpleFlow6() = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
        println("Emitting $i ${Thread.currentThread().name}.")
    }
}

@Test
fun `test flow back pressure`() = runBlocking {
    val time = measureTimeMillis {
        simpleFlow6()
//                .buffer(50) // 并行发射 3 个元素，需要 100 ms
//                .flowOn(Dispatchers.Default) // 让发射在后台线程，也是并行发射，需要 100 ms
//                .conflate()
//                .collect {
            .collectLatest {
                delay(300) // 处理这个元素消耗 300 ms
                println("Collected $it ${Thread.currentThread().name}")
            }
    }
    println("Collected in $time ms.")
}
```

不处理背压情况下，上面程序大概需要消耗 （100 + 300）* 3 = 1200 ms。

- 使用 buffer()，大概需要 100 + 3 * 300 = 1000 ms，因为该方式发射元素是异步的。
- 使用 flowOn 操作符，也是大概需要 100 + 3 * 300 = 1000 ms，该方式发射元素也是异步，但是发射元素切换到了后台线程。
- 使用 conflate，合并发射项，不对每个值进行处理，大概需要 100 + 2 * 300 = 700 ms。该方式发射元素是异步的，收集过程忽略了中间的元素，只处理了前后两个元素。
- 使用 collectLastest，上面程序大概需要消耗 700 ms。该方式发射元素也是异步，它会取消并重新发射最后一个值。

#### 操作符

##### 转换操作符

- 可以使用操作符转换流，就像使用集合与序列一样。
- 转换操作符应用于上游流，并返回下游流。
- 这些操作符也是冷操作符，就像流一样。这类操作符本身不是挂起函数。
- 它运行的速度很快，返回新的转换流的定义。

```kotlin
private suspend fun performRequest(request: Int): String {
    delay(1000)
    return "response $request"
}

@Test
fun `test transform flow operator`() = runBlocking {
    (1..3).asFlow()
//            .map { performRequest(it) }
        .transform {
            emit("Making request $it")
            emit(performRequest(it))
        }
        .collect { println(it) }
}
```

map 转换一次。transform 可以转换发射多次。

##### 限长操作符

```kotlin
private fun numbers() = flow {
    try {
        emit(1)
        emit(2)
        println("This line will not execute.")
        emit(3)
    } finally {
        println("Finally in numbers.")
    }
}

@Test
fun `test limit length operator`() = runBlocking {
    numbers().take(2).collect { println(it) }
}

// 打印结果
1
2
Finally in numbers.
```

##### 末端流操作符

末端流操作符是在流上用于启动流收集的挂起函数。collect 是最基础的末端操作符，但是还有另外一些更方便的使用的末端操作符：

- 转化为各种集合，例如 toList 与 toSet。
- 获取第一个（first）值与确保流发射单个（Single）值的操作符。
- 使用 reduce 与 fold 将流规约到单个值。

```kotlin
@Test
fun `test terminal operator`() = runBlocking {
    val sum = (1..5).asFlow()
        .map { it * it }
        .reduce { a, b -> a + b }
    println(sum)
}

// 打印结果
55
```

##### 组合操作符

就像 Kotlin 标准库中的 Sequence.zip 扩展函数一样，流拥有一个 zip 操作符用于组合两个流中的相关值。

```kotlin
@Test
fun `test zip operator`() = runBlocking {
    val numbs = (1..3).asFlow()
    val strs = flowOf("One", "Two", "Three")
    numbs.zip(strs) { a, b -> "$a -> $b" }.collect { println(it) }
}

// 打印结果
1 -> One
2 -> Two
3 -> Three
```

```kotlin
@Test
fun `test zip2 operator`() = runBlocking {
    val numbs = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("One", "Two", "Three").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    numbs.zip(strs) { a, b -> "$a -> $b" }.collect {
        println("$it at ${System.currentTimeMillis() - startTime} ms from start.")
    }
}

// 打印结果
1 -> One at 442 ms from start.
2 -> Two at 839 ms from start.
3 -> Three at 1246 ms from start.
```

##### 展平操作符

流表示异步接收的值序列，所以很容易遇到这样的情况：每个值都会触发对另一个值序列的请求，然而，由于流具有异步的性质，因此需要不同的展平模式，存在一系列的流展平操作符：

- flatMapConcat 连接模式。
- flatMapMerge 合并模式。
- flatMapLastest 最新展平模式。

![](https://gitee.com/zch0304/images/raw/master/note/1663680667795.jpg) 

```kotlin
private fun requestFlow(i: Int) = flow<String> {
    emit("$i: First.")
    delay(500)
    emit("$i: Second.")
}

@Test
fun `test flatMapConcat operator`() = runBlocking {
    val startTime = System.currentTimeMillis()
    (1..3).asFlow()
        .onEach {
            delay(100)
        }.flatMapConcat {
            requestFlow(it)
        }.collect {
            println("$it at ${System.currentTimeMillis() - startTime} ms from start.")
        }
}

// 打印结果
1: First. at 138 ms from start.
1: Second. at 665 ms from start.
2: First. at 774 ms from start.
2: Second. at 1289 ms from start.
3: First. at 1398 ms from start.
3: Second. at 1911 ms from start.
```

```kotlin
@Test
fun `test flatMapMerge operator`() = runBlocking {
    val startTime = System.currentTimeMillis()
    (1..3).asFlow()
        .onEach {
            delay(100)
        }.flatMapMerge {
            requestFlow(it)
        }.collect {
            println("$it at ${System.currentTimeMillis() - startTime} ms from start.")
        }
}

// 打印结果
1: First. at 192 ms from start.
2: First. at 293 ms from start.
3: First. at 404 ms from start.
1: Second. at 698 ms from start.
2: Second. at 806 ms from start.
3: Second. at 919 ms from start.
```

```kotlin
@Test
fun `test flatMapLatest operator`() = runBlocking {
    val startTime = System.currentTimeMillis()
    (1..3).asFlow()
        .onEach {
            delay(100)
        }.flatMapLatest {
            requestFlow(it)
        }.collect {
            println("$it at ${System.currentTimeMillis() - startTime} ms from start.")
        }
}

// 打印结果
1: First. at 178 ms from start.
2: First. at 326 ms from start.
3: First. at 441 ms from start.
3: Second. at 943 ms from start.
```

#### 流的异常处理

当运算符中的发射器或代码抛出异常时，有几种处理异常的方法：

- try/catch 块。
- catch 函数。

**1、捕获下游异常**：

```kotlin
private fun exceptionFlow() = flow {
    for (i in 1..3) {
        println("Emitting $i.")
        emit(i)
    }
}

@Test
fun `test flow exception`() = runBlocking {
    try {
        exceptionFlow().collect {
            println(it)
            check(it <= 1) {
                "Collect $it."
            }
        }
    } catch (e: Exception) {
        println("Caught $e")
    }
}

// 打印结果
Emitting 1.
1
Emitting 2.
2
Caught java.lang.IllegalStateException: Collect 2.
```

**2、捕获上游异常**：

```kotlin
@Test
fun `test flow exception2`() = runBlocking {
    flow {
        emit(1)
        throw ArithmeticException("Div 0.")
    }.catch {
        println("Caught $it")
    }.flowOn(Dispatchers.IO)
        .collect {
            println(it)
        }
}

// 打印结果
1
Caught java.lang.ArithmeticException: Div 0.
```

**3、恢复异常**：

```kotlin
@Test
fun `test flow exception3`() = runBlocking {
    flow {
        throw ArithmeticException("Div 0.")
        emit(1)
    }.catch {
        println("Caught $it")
        emit(10)
    }.flowOn(Dispatchers.IO)
        .collect {
            println(it)
        }
}

// 打印结果
Caught java.lang.ArithmeticException: Div 0.
10
```

#### 流的完成

当流收集完成时（普通情况或异常情况），它可能需要执行一个动作。

- 命令式 finally 块。
- onCompletion 声明式处理。

1、finally 形式：

```kotlin
@Test
fun `test flow complete in finally`() = runBlocking {
    fun simpleFlow() = (1..3).asFlow()

    try {
        simpleFlow().collect {
            println(it)
        }
    } finally {
        println("Done.")
    }
}

// 打印结果
1
2
3
Done.
```

2、onCompletion 函数可以拿到上游异常信息，但捕获异常还是需要 catch 函数。

```kotlin
@Test
fun `test flow complete in onCompletion`() = runBlocking {
    fun simpleFlow() = flow {
        emit(1)
        throw RuntimeException()
    }
    simpleFlow()
        .onCompletion { exception ->
            if (exception != null) {
                println("Flow completed exceptionally.")
            }
        }
        .catch { exception ->
            println("Caught $exception.")
        }
        .collect {
            println(it)
        }
}

// 打印结果
1
Flow completed exceptionally.
Caught java.lang.RuntimeException.
```

3、onCompletion 函数也能拿到下游异常信息，但捕获异常需要用 try/catch 形式。

```kotlin
@Test
fun `test flow complete in onCompletion2`() = runBlocking {
    fun simpleFlow() = flow {
        emit(1)
        throw RuntimeException()
    }
    try {
        simpleFlow()
            .onCompletion { exception ->
                if (exception != null) {
                    println("Flow completed exceptionally.")
                }
            }.collect {
                println(it)
                check(it <= 1) {
                    "Collect $it."
                }
            }
    } catch (e: Exception) {
        println("Caught $e.")
    }
}

// 打印结果
1
Flow completed exceptionally.
Caught java.lang.RuntimeException.
```

### Channel

#### 什么是 Channel

Channel 实际上是一个**并发安全的队列**，它可以用来连接协程，实现不同协程的通信。

![](https://gitee.com/zch0304/images/raw/master/note/1666439476014.jpg) 

```kotlin
@Test
fun `test know Channel`() = runBlocking {
    val channel = Channel<Int>()
    // 生产者
    val producer = GlobalScope.launch {
        var i = 1
        while (true) {
            delay(1000)
            println("send $i.")
            channel.send(i++)
        }
    }
    // 消费者
    val consumer = GlobalScope.launch {
        while (true) {
            // delay(3000)
            val element = channel.receive()
            println("receive $element.")
        }
    }
    joinAll(producer, consumer)
}

// 打印结果
send 1.
receive 1.
send 2.
receive 2.
......
```

#### Channel 的容量

Channel 实际上就是一个队列，队列中一定存在缓冲区，那么一旦这个缓冲区满了，并且一直没有人调用 receive 并取走函数，send 就需要挂起。故意让接收端的节奏放慢，发现 send 总是会挂起，直到 receive 之后才会继续往下执行。

比如上面的代码让消费者延迟 3s，那么生产者 send 一个元素后，需要等消费者在 3s 之后 receive 到元素才能 send 下一个元素。因为 Channel 默认的容量是 0。

#### 迭代 Channel

Channel 本身确实像序列，所以我们在读取的时候可以直接获取一个 Channel 的 iterator。

```kotlin
@Test
fun `test iterator Channel`() = runBlocking {
    val channel = Channel<Int>(Channel.UNLIMITED)
    // 生产者
    val producer = GlobalScope.launch {
        for (i in 1..5) {
            val element = i * i
            println("send $element.")
            channel.send(element)
        }
    }
    // 消费者
    val consumer = GlobalScope.launch {
//            val it = channel.iterator()
//            while (it.hasNext()) {
//                val element = it.next()
//                println("receive $element.")
//                delay(2000)
//            }

        for (element in channel) {
            println("receive $element.")
            delay(2000)
        }
    }
    joinAll(producer, consumer)
}

// 打印结果
send 1.
send 4.
send 9.
send 16.
send 25.
receive 1.
receive 4.
receive 9.
receive 16.
receive 25.
```

#### produce 与 actor

- 构造生产者与消费者的**便捷方法**。
- 我们可以通过 produce 方法启动一个生产者协程，并返回一个 ReceiveChannel，其它协程就可以用这个 Channel 来接收数据了。反过来，我们可以用 actor 启动一个消费者协程。

```kotlin
@Test
fun `test fast producer Channel`() = runBlocking {
    val receiveChannel: ReceiveChannel<Int> = GlobalScope.produce {
        repeat(5) {
            delay(1000)
            send(it)
        }
    }
    val consumer = GlobalScope.launch {
        for (element in receiveChannel) {
            println("receive $element.")
        }
    }
    consumer.join()
}

// 打印结果
receive 0.
receive 1.
receive 2.
receive 3.
receive 4.
```

```kotlin
@Test
fun `test fast consumer Channel`() = runBlocking {
    val sendChannel: SendChannel<Int> = GlobalScope.actor<Int> {
        while (true) {
            val element = receive()
            println("receive $element.")
        }
    }
    val producer = GlobalScope.launch {
        for (i in 0..3) {
            sendChannel.send(i)
        }
    }
    producer.join()
}

// 打印结果
receive 0.
receive 1.
receive 2.
receive 3.
```

#### Channel 的关闭

- produce 和 actor 返回的 Channel 都会随着对应的协程执行完毕而关闭，也正是这样，Channel 才被称为**热数据流**。
- 对于一个 Channel，如果我们调用了它的 close 方法，它会立即停止接收新元素，也就是说这时它的 **isClosedForSend** 会立即返回 true。而由于 Channel 缓冲区的存在，这时候可能还有一些元素没有被处理完，因此要等所有的元素都被读取之后 isCloseForReceive 才会返回 true。
- Channel 的生命周期最好由主导方来维护，建议**由主导的一方实现关闭**。

```kotlin
@Test
fun `test close Channel`() = runBlocking {
    val channel = Channel<Int>(3)
    // 生产者
    val producer = GlobalScope.launch {
        List(3) {
            println("send $it.")
            channel.send(it)
        }
        channel.close()
        println("Close Channel. | - ClosedForSend: ${channel.isClosedForSend}. | - ClosedForReceive: ${channel.isClosedForReceive}.")
    }
    // 消费者
    val consumer = GlobalScope.launch {
        for (element in channel) {
            println("receive $element.")
            delay(1000)
        }
        println("After Consuming. | - ClosedForSend: ${channel.isClosedForSend}. | - ClosedForReceive: ${channel.isClosedForReceive}.")
    }
    joinAll(producer, consumer)
}

// 打印结果
send 0.
send 1.
send 2.
receive 0.
Close Channel. | - ClosedForSend: true. | - ClosedForReceive: false.
receive 1.
receive 2.
After Consuming. | - ClosedForSend: true. | - ClosedForReceive: true.
```

#### BroadcastChannel

前面提到，发送端和接收端在 Channel 中存在一对多的情形，从数据处理本身来讲，虽然有多个接收端，但是同一个元素只会被一个接收端读到。广播则不然，**多个接收端不存在互斥行为**。

![](https://gitee.com/zch0304/images/raw/master/note/1666502596729.jpg) 

```kotlin
@Test
fun `test broadcastChannel`() = runBlocking {
//        val broadcastChannel = BroadcastChannel<Int>(Channel.BUFFERED)
    val channel = Channel<Int>()
    val broadcastChannel = channel.broadcast(3)
    val producer = GlobalScope.launch {
        List(3) {
            delay(100)
            broadcastChannel.send(it)
        }
        broadcastChannel.close()
    }
    List(3) { index ->
        GlobalScope.launch {
            val receiveChannel = broadcastChannel.openSubscription()
            for (i in receiveChannel) {
                println("[#$index] receive: $i.")
            }
        }
    }.joinAll()
}

// 打印结果
[#2] receive: 0.
[#0] receive: 0.
[#1] receive: 0.
[#1] receive: 1.
[#2] receive: 1.
[#0] receive: 1.
[#1] receive: 2.
[#2] receive: 2.
[#0] receive: 2.
```

### select - 多路复用

#### 什么是多路复用

数据通信系统或计算机网络系统中，传输媒体的带宽或容量往往会大于传输单一信号的需求，为了有效地利用通信线路，希望**一个信道同时传输多路信号**，这就是所谓的多路复用技术（Multiplexing）。

#### 复用多个 await

两个 API 分别从网络和本地缓存获取数据，期望哪个先返回就先用哪个做展示。

![](https://gitee.com/zch0304/images/raw/master/note/1666531343879.png)  

```kotlin
data class Response<T>(val value: T, val isLocal: Boolean)

data class User(val name: String, val address: String)

fun CoroutineScope.getUserFromLocal(name: String) = async(Dispatchers.IO) {
    delay(1000)
    User(name, "Local")
}

fun CoroutineScope.getUserFromServer(name: String) = async(Dispatchers.IO) {
    delay(2000)
    User(name, "Server")
}

@Test
fun `test select await`() = runBlocking {
    GlobalScope.launch {
        val localRequest = getUserFromLocal("AAA")
        val serverRequest = getUserFromServer("BBB")

        val userResponse = select<Response<User>> {
            localRequest.onAwait { Response(it, true) }
            serverRequest.onAwait { Response(it, false) }
        }

        userResponse.value.let {
            println(it)
        }
    }.join()
}

// 打印结果
User(name=AAA, address=Local)
```

#### 复用多个 Channel

跟 await 类似，会接收到最快的那个 Channel 消息。

```kotlin
@Test
fun `test select Channel`() = runBlocking {
    val channels = listOf(Channel<Int>(), Channel<Int>())
    GlobalScope.launch {
        delay(100)
        channels[0].send(200)
    }
    GlobalScope.launch {
        delay(50)
        channels[1].send(100)
    }
    val result = select<Int?> {
        channels.forEach { channel ->
            channel.onReceive { it }
        }
    }
    println(result)
}

// 打印结果
100
```

#### SelectClause

我们怎么知道哪些事件可以被 **select** 呢？其实所有能够被 select 的事件都是 **SelectClauseN** 类型，包括：

- **SelectClause0**：对应事件没有返回值，例如 join 没有返回值，对应的 onJoin 就是这个类型，使用时 onJoin 的参数是一个无参函数。

- **SelectClause1**：对应事件有返回值，前面的 onAwait 和 onReceive 都是此类情况。
- **SelectClause2**：对应事件有返回值，此外还需要额外的一个参数，例如 Channel.onSend 有两个参数，第一个就是一个 Channel 数据类型的值，表示即将发送的值，第二个是发送成功时的回调。

如果我们想要确认挂起函数是否支持 select，只需要查看其是否存在对应的 SelectClauseN 即可。

```kotlin
@Test
fun `test SelectClause0`() = runBlocking {
    val job1 = GlobalScope.launch {
        println("job 1")
        delay(100)
    }
    val job2 = GlobalScope.launch {
        println("job 2")
        delay(10)
    }
    select<Unit> {
        job1.onJoin {
            println("job 1 onJoin")
        }
        job2.onJoin {
            println("job 2 onJoin")
        }
    }
}

// 打印结果
job 1
job 2
job 2 onJoin
```

```kotlin
@Test
fun `test SelectClause2`() = runBlocking {
    val channels = listOf(Channel<Int>(), Channel<Int>())
    println(channels)
    launch(Dispatchers.IO) {
        select {
            launch {
                delay(10)
                channels[1].onSend(200) { sentChannel ->
                    println("sent on $sentChannel")
                }
            }
            launch {
                delay(100)
                channels[0].onSend(100) { sentChannel ->
                    println("sent on $sentChannel")
                }
            }
        }
    }

    GlobalScope.launch {
        println(channels[0].receive())
    }
    GlobalScope.launch {
        println(channels[1].receive())
    }

    delay(1000)
}

// 打印结果
[RendezvousChannel@24c4ddae{EmptyQueue}, RendezvousChannel@766653e6{EmptyQueue}]
200
sent on RendezvousChannel@766653e6{EmptyQueue}
```

#### 使用 Flow 实现多路复用

多数情况下，我们可以通过构造合适的 Flow 来实现多路复用的效果。

```kotlin
@Test
fun `test select flow`() = runBlocking {
    // 函数 -> 协程 -> Flow -> Flow 合并
    val name = "guest"
    coroutineScope {
        listOf(::getUserFromLocal, ::getUserFromServer)
            .map { function ->
                function.call(name)
            }.map { deferred ->
                flow {
                    emit(deferred.await())
                }
            }.merge()
            .collect { user ->
                println(user)
            }
    }
}

// 打印结果
User(name=guest, address=Local)
User(name=guest, address=Server)
```

### 并发安全

由于协程是基于线程的，既然线程有并发问题，那么协程也一定有。

#### 不安全的并发访问

我们使用线程在解决并发问题的时候总是会遇到线程安全问题，而 Java 平台上的 Kotlin 协程实现免不了存在并发调度的情况，因此线程安全同样值得留意。

```kotlin
@Test
fun `test not safe concurrent`() = runBlocking {
    var count = 0
    List(10000) {
        GlobalScope.launch {
            count++
        }
    }.joinAll()
    println(count)
}

// 打印结果（每次打印可能不一样）
9997
```

Java API 中安全的并发访问：

```kotlin
@Test
fun `test safe concurrent`() = runBlocking {
    var count = AtomicInteger(0)
    List(10000) {
        GlobalScope.launch {
            count.incrementAndGet()
        }
    }.joinAll()
    println(count.get())
}

// 打印结果（每次打印都一样）
10000
```

#### 协程的并发工具

除了我们在线程中常用的解决并发问题的手段之外，协程框架也提供了一些并发安全的工具，包括：

- **Channel**：并发安全的消息通道，我们已经非常熟悉。
- **Mutex**：轻量级锁，它的 lock 和 unlock 从语义上与线程锁比较类似，之所以轻量是因为它在获取不到锁时不会阻塞线程，而是挂起等待锁的释放。
- **Semaphore**：轻量级信号量，信号量可以有多个，协程在获取到信号量后即可执行并发操作。当 Semaphore 的参数为 1 时，效果等价于 Mutex。

```kotlin
@Test
fun `test safe concurrent tools`() = runBlocking {
    var count = 0
    val mutex = Mutex()
    List(10000) {
        GlobalScope.launch {
            mutex.withLock {
                count++
            }
        }
    }.joinAll()
    println(count)
}

// 打印结果（每次打印都一样）
10000
```

```kotlin
@Test
fun `test safe concurrent tools2`() = runBlocking {
    var count = 0
    val semaphore = Semaphore(1)
    List(10000) {
        GlobalScope.launch {
            semaphore.withPermit {
                count++
            }
        }
    }.joinAll()
    println(count)
}
```

#### 避免访问外部可变状态

编写函数时要求它不得访问外部状态，只能基于参数做运算，通过返回值提供运算结果。

```kotlin
@Test
fun `test avoid access outer variable`() = runBlocking {
    var count = 0
    val result = count + List(10000) {
        GlobalScope.async { 1 }
    }.map {
        it.await()
    }.sum()
    println(result)
}

// 打印结果（每次打印都一样）
10000
```

### 冷流还是热流

- Flow 是冷流，什么是冷流？简单来说，如果 Flow 有了订阅者 Collector 以后，发射出来的值才会实实在在的存在于内存之中，这跟懒加载的概念很像。
- 与之相对的是热流，StateFlow 和 SharedFlow 是热流，在垃圾回收之前，都是存在于内存之中，并且处于活跃状态的。

### StateFlow

StateFlow 是一个状态容器式**可观察数据流**，可以向其收集器发出当前状态更新和新状态更新。还可通过其 value 属性读取当前状态。

```kotlin
class NumberViewModel : ViewModel() {

    // StateFlow 与 LiveData 很像，但能使用 Flow 的操作符。
    val number = MutableStateFlow(0)

    fun increment() {
        number.value++
    }

    fun decrement() {
        number.value--
    }
}
```

```kotlin
class NumberFragment : Fragment() {

    private val mNumberViewModel: NumberViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        btnPlus.setOnClickListener {
            // 加法
            mNumberViewModel.increment()
        }
        
        btnMinus.setOnClickListener {
            // 减法
            mNumberViewModel.decrement()
        }

        lifecycleScope.launch {
            // 收集数据
            mNumberViewModel.number.collect {
                tvNumber.text = it.toString()
            }
        }
    }
}
```

### SharedFlow

SharedFlow 会向从其中收集值的所有使用方发出数据。

```kotlin
/**
 * 使用 SharedFlow 模拟 EventBus 收发数据
 */
object LocalEventBus {

    val events = MutableSharedFlow<Event>()

    suspend fun postEvent(event: Event) {
        // 发送数据
        events.emit(event)
    }
}

data class Event(val timestamp: Long)
```

```kotlin
class SharedViewModel : ViewModel() {

    private var job: Job? = null

    /**
     * 开始刷新数据
     */
    fun startRefresh() {
        // 开启协程
        job = viewModelScope.launch(Dispatchers.IO) {
            // 发送数据
            while (true) {
                LocalEventBus.postEvent(Event(System.currentTimeMillis()))
            }
        }
    }

    /**
     * 停止刷新数据
     */
    fun stopRefresh() {
        // 关闭协程
        job?.cancel()
    }
}
```

```kotlin
/**
 * 发送数据界面
 */
class SharedFlowFragment : Fragment() {

    private val mSharedViewModel: SharedViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        btnStart.setOnClickListener {
            // 发送[开始刷新数据]指令
            mSharedViewModel.startRefresh()
        }
        btnStop.setOnClickListener {
            // 发送[停止刷新数据]指令
            mSharedViewModel.stopRefresh()
        }
    }
}
```

```kotlin
/**
 * 收集数据界面
 */
class TextsFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
 	    lifecycleScope.launch {
            // 收集数据
            LocalEventBus.events.collect {
                textView.text = "${tag}: ${it.timestamp}"
            }
        }
    }
}
```

### Flow 的应用

#### Flow 与文件下载的应用

```kotlin
/**
 * 下载状态
 */
sealed class DownloadStatus {

    // 空状态
    object None : DownloadStatus()

    // 下载进度
    data class Progress(val value: Int) : DownloadStatus()

    // 错误
    data class Error(val throwable: Throwable) : DownloadStatus()

    // 完成
    data class Done(val file: File) : DownloadStatus()
}
```

```kotlin
/**
 * 下载文件管理类
 */
object DownloadManager {

    /**
     * 下载文件
     * @param url String 远程文件地址
     * @param file File 下载到的本地文件
     * @return Flow<DownloadStatus>
     */
    fun download(url: String, file: File): Flow<DownloadStatus> {
        return flow {
            val request = Request.Builder().url(url).get().build()
            val response = OkHttpClient.Builder().build().newCall(request).execute()
            if (response.isSuccessful) {
                response.body()!!.let { body ->
                    val total = body.contentLength()
                    // 文件读写
                    file.outputStream().use { outputStream ->
                        val inputStream = body.byteStream()
                        var emittedProgress = 0L
                        inputStream.copyTo(outputStream) { bytesCopied ->
                            val progress = bytesCopied * 100 / total
                            if (progress - emittedProgress > 5) {
                                delay(100)
                                emit(DownloadStatus.Progress(progress.toInt()))
                                emittedProgress = progress
                            }
                        }
                    }
                }
                emit(DownloadStatus.Done(file))
            } else {
                throw IOException(response.toString())
            }
        }.catch {
            file.delete()
            emit(DownloadStatus.Error(it))
        }.flowOn(Dispatchers.IO)
    }
}
```

```kotlin
/**
 * 扩展方法 读写文件并返回下载进度
 * @receiver InputStream
 * @param out OutputStream
 * @param bufferSize Int
 * @param progress Function1<Long, Unit>
 * @return Long
 */
inline fun InputStream.copyTo(out: OutputStream, bufferSize: Int = DEFAULT_BUFFER_SIZE, progress: (Long) -> Unit): Long {
    var bytesCopied: Long = 0
    val buffer = ByteArray(bufferSize)
    var bytes = read(buffer)
    while (bytes >= 0) {
        out.write(buffer, 0, bytes)
        bytesCopied += bytes
        bytes = read(buffer)

        progress(bytesCopied)
    }
    return bytesCopied
}
```

```kotlin
/**
 * 下载文件界面
 */
class DownloadFragment : Fragment() {

    private val URL = "https://img1.baidu.com/it/u=413643897,2296924942&fm=253&fmt=auto&app=138&f=JPEG?w=800&h=500"

    private fun download() {
        // sdcard/Android/data/com.zch.flowpractice/files/pic.png
        val file = File(requireActivity().getExternalFilesDir(null)?.path, "pic.jpg")
        lifecycleScope.launch {
            DownloadManager.download(URL, file).collect { status ->
                when (status) {
                    is DownloadStatus.Progress -> {
                        progressBar.progress = status.value
                        tvProgress.text = "${status.value}%"
                    }

                    is DownloadStatus.Error -> {
                        ToastUtils.showLong("下载错误")
                    }

                    is DownloadStatus.Done -> {
                        progressBar.progress = 100
                        tvProgress.text = "100%"
                        ToastUtils.showShort("下载完成")
                    }

                    else -> {
                        ToastUtils.showShort("下载失败")
                    }
                }
            }
        }
    }
}
```

#### Flow 与 Room 的应用

```Kotlin
/**
 * Room 实体类声明
 */
@Entity
data class User(
    // 主键
    @PrimaryKey
    val uid: Int,
    // 数据库里存入的字段名
    @ColumnInfo(name = "first_name")
    val firstName: String,
    // 数据库里存入的字段名
    @ColumnInfo(name = "last_name")
    val lastName: String
)
```

```kotlin
/**
 * Dao 类声明
 */
@Dao
interface UserDao {
    // 这里设置了一下冲突，如果两条记录相同则会替换
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: User)

    // 这里不需要挂起（返回 Flow 或 LiveData 都不需要）
    @Query("SELECT * FROM user")
    fun getAll(): Flow<List<User>>
}
```

```kotlin
/**
 * 数据库操作类
 * 
 * entities      数据库里存入的表，可是多个
 * version       数据库的版本号
 * exportSchema  是否生成 json 文件，用于查看数据库的结构
 */
@Database(entities = [User::class], version = 1, exportSchema = false)
abstract class AppDataBase : RoomDatabase() {

    /**
     * Dao 对象
     */
    abstract fun UserDao(): UserDao

    companion object {
        private var instance: AppDataBase? = null

        fun getInstance(context: Context): AppDataBase {
            // 对象锁
            return instance ?: synchronized(this) {
                Room.databaseBuilder(context, AppDataBase::class.java, "user_db")
                    .build().also { instance = it }
            }
        }
    }
}
```

```kotlin
/**
 * User ViewModel
 */
class UserViewModel(app: Application) : AndroidViewModel(app) {

    /**
     * 插入数据
     * @param user User
     */
    fun insert(user: User) {
        viewModelScope.launch {
            AppDataBase.getInstance(getApplication()).UserDao().insert(user)
        }
    }

    /**
     * 获取所有数据
     * @return Flow<List<User>>
     */
    fun getAll(): Flow<List<User>> {
        return AppDataBase.getInstance(getApplication()).UserDao().getAll()
            .catch { e ->
                e.printStackTrace()
            }.flowOn(Dispatchers.IO)
    }
}
```

```kotlin
/**
 * 用户界面
 */
class UserFragment : BaseBindingFragment() {

    private val mBinding: FragmentUserBinding by lazy {
        FragmentUserBinding.inflate(layoutInflater)
    }

    private val mUserAdapter: UserAdapter by lazy {
        UserAdapter()
    }

    private val mUserViewModel: UserViewModel by viewModels()

    override fun getBindingRoot() = mBinding.root

    override fun initView() {
        mBinding.rvUser.adapter = mUserAdapter
    }

    override fun initEvent() {
        mBinding.btnAddUser.setOnClickListener {
            mBinding.run {
                // 插入数据
                mUserViewModel.insert(
                    User(
                        edtUid.text.toString().toInt(),
                        edtFirstName.text.toString(),
                        edtLastName.text.toString()
                    )
                )
            }
        }
    }

    override fun initData() {
        lifecycleScope.launch {
            // 获取所有数据
            mUserViewModel.getAll().collect {
                mUserAdapter.submitList(it)
            }
        }
    }
}
```

#### Flow 与 Retrofit 的应用

```kotlin
/**
 * 文章实体类
 * @property id 文章 id
 * @property text 文章内容
 */
data class Article(val id: Int, val text: String)
```

```kotlin
/**
 * 文章接口
 */
interface ArticleApi {

    @GET("article")
    suspend fun searchArticle(@Query("key") key: String): List<Article>
}
```

```kotlin
/**
 * Retrofit 网络请求管理类
 */
object RetrofitClient {

    private const val URL = ""

    private val instance: Retrofit by lazy {
        Retrofit.Builder()
            .client(OkHttpClient.Builder().build())
            .baseUrl(URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    val articleApi: ArticleApi by lazy {
        instance.create(ArticleApi::class.java)
    }
}
```

```kotlin
/**
 * 文章 ViewModel
 */
class ArticleViewModel : ViewModel() {

    val articles = MutableLiveData<List<Article>>()

    fun searchArticle(key: String) {
        viewModelScope.launch(Dispatchers.Main) {
            flow {
                val list = RetrofitClient.articleApi.searchArticle(key)
                emit(list)
            }.flowOn(Dispatchers.IO)
                .catch { e -> e.printStackTrace() }
                .collect {
                    articles.value = it
                }
        }
    }
}
```

```kotlin
/**
 * 文章界面
 */
class ArticleFragment : BaseBindingFragment() {

    private val articleViewModel by viewModels<ArticleViewModel>()

    private val mBinding: FragmentArticleBinding by lazy {
        FragmentArticleBinding.inflate(layoutInflater)
    }

    override fun getBindingRoot() = mBinding.root

    override fun initView() {
    }

    override fun initEvent() {
    }

    override fun initData() {
        // 请求数据
        articleViewModel.searchArticle("三国演义")
        
        // 监听数据据返回
        articleViewModel.articles.observe(viewLifecycleOwner) {

        }
    }
}
```
