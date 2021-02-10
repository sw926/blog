---
title: 'Tips:Java Thread'
date: 2019-07-03 12:35:25
category: Java
tags: 
    - Java Thread
---

## 先看一个经典的多线程的例子：

``` kotlin
class A {
    private var value = 0

    fun increase() {
        value++
    }

    fun printValue() {
        println("value = $value")
    }
}

fun main() {
    val a = A()
    for (i in 0 until 100) {
        Thread(Runnable {
            for (j in 0 until 100000) {
                a.increase()
            }
        }).start()
    }

    // 等待所有线程执行完，activeCount 根据环境不同，为 1 或 2
    while (Thread.activeCount() > 2) { 
        Thread.yield()
    }
    
    a.printValue()
}
```

输出的结果可能每次都不一样
同步的方法：

 - synchronized
 - lock
 - 使用 Atomic

加上时间的统计，测试一下速度：

``` kotlin
fun main() {
    val start = System.currentTimeMillis()
    val a = A()
    for (i in 0 until 100) {
        Thread(Runnable {
            for (j in 0 until 100000) {
                a.increase()
            }
        }).start()
    }

    while (Thread.activeCount() > 2) {
        Thread.yield()
    }

    a.printValue()
    println("spent: ${System.currentTimeMillis() - start}ms")
}
```

## 首先使用 synchronized

``` 
@Synchronized
fun increase() {
    value++
}
```

花费时间 `550ms` 左右。

还有一种方式是使用同步代码块：

``` kotlin
private val lock = Any()

fun increase() {
    synchronized(lock) {
        value++
    }
}
```

花费时间比使用 synchronized 函数要少一点，在 `480ms` 左右。

## 使用 Lock

``` kotlin
private val lock = ReentrantLock()
fun increase() {
    lock.lock()
    value++
    lock.unlock()
}
```

花费时间 `310ms` 左右

## 最后使用 Atomic

``` kotlin
private var value = AtomicLong()

fun increase() {
    value.incrementAndGet()
}
```

花费时间 `230ms` 左右。



