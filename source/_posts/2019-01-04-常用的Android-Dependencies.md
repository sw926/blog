---
title: 常用的Android Dependencies
date: 2019-01-04 11:16:00
tags: 
    - Android
category: Android
---

2019.01.04

## 启用 DataBinding：

在项目根目录下的 `gradle.properties` 下添加：

```
android.databinding.enableV2=true
```

在 `app` 目录下的 `build.gradle` 下添加：

``` gradle
android {

    // ...

    dataBinding {
        enabled = true
    }

    // ...

}
```

## lifecycle

``` gradle
implementation 'android.arch.lifecycle:common-java8:1.1.1'
implementation 'android.arch.lifecycle:extensions:1.1.1'
implementation 'android.arch.lifecycle:runtime:1.1.1'
```

上面的依赖使用的是 Java8，不用添加 `apt` 或者 `kapt` 或者 `annotationProcessor`，但是需要设置 `compileOptions`：

``` gradle
android {

    // ...

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    // ...

}
```

## Navigation

根目录下：

``` gradle
buildscript {
    // ...
    dependencies {
        // ...
        classpath "android.arch.navigation:navigation-safe-args-gradle-plugin:1.0.0-alpha09"
    }
}
```

`app` 目录下：

``` gradle
apply plugin: "androidx.navigation.safeargs"
```

添加依赖：

``` gradle
implementation "android.arch.navigation:navigation-fragment-ktx:1.0.0-alpha09" // use -ktx for Kotlin
implementation "android.arch.navigation:navigation-ui-ktx:1.0.0-alpha09" // use -ktx for Kotlin

// optional - Test helpers
androidTestImplementation "android.arch.navigation:navigation-testing-ktx:1.0.0-alpha06" // use -ktx for Kotlin
```

在 `Android Studio 3.2` 中，还要开启 `Experimental` 设置：

```
Preferences -> Experimental
```

剩下的就是在 `res` 目录下新建 `navigation` 文件夹，添加一个 `navigation.xml` 文件
