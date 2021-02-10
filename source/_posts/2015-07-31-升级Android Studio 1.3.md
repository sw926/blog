---
title: 升级Android Studio 1.3
date: 2015-07-31 15:21:05
tags: 
    - Android
    - Android Studio
category: Android
---
终于等到了1.3正式版，必须要升级一下。
一下内容来自
<http://android-developers.blogspot.jp/2015/07/get-your-hands-on-android-studio-13.html>
Android Studio 1.3是今年最大的功能升级，包含新的内存检查工具，提升的测试功能的支持，还有C++完整的编辑功能和调试功能。
##### 性能和测试工具
- 原生的内存查看工具 Android Memory (HPROF) Viewer
- 内存分配跟踪工具
- APK Tests in Module
##### 代码和SDK管理
- App权限注解 
- Data Binding Support
- SDK自动升级和SDK Manager
- C++支持

C++的支持需要Gradle2.5
设置NDK路径
![设置NDK路径](/images/ndk-install-link.png)
另外要使用Experimental Gradle Plugin <http://tools.android.com/tech-docs/new-build-system/gradle-experimental>
