---
title: Android Studio 2.2新功能预览
date: 2016-06-02 10:57:06
tags:
    - Android
    - Android Studio
---
# 升级SDK可用Background
多加了个按钮，可用一边写代码一边下载SDK

# Instant Run
修改代码一秒启动

# APK analyzer
- 分析任何的APK
- 查看APK下载包的大小，解压后的实际大小
- 反编译资源文件，甚至能还原layout中的资源id，还有，代码，代码，代码，重要的事情说三遍，可以和APKTOOL，dex2jar说拜拜
- 分析dex，显示每部分的方法数，直观的告诉你是怎么超过64k的
打开方法：Build -> Analyz APK
ConstraintLayout

# 改进的Manifest Editor
下方添加了一个Merge Manifest，可用查看APK最终的Manifest，分析Manifest里面的东西都是从哪儿过来的，跳转到对应的Manifest

# 全新的Project Structure
- dependency可视化，贴心的提醒那些依赖有新版本了，一键升级到最新版本
- 添加依赖直接搜索，方便的配置使用debug还是release
感觉Google在干微软的活

# NDK支持
- 不用experimental Gradle plugin了
- 支持external build systems，可用用CMAKE了(虽然我不知道这是干什么)
- 干货，调试的时候直接从java跳到C/C++代码！！！这是要抛弃java的节奏吗

# 命令行build，直接下载缺失的sdk
gradle.properties中添加
```
android.builder.sdkDownload = true
```
编译的时候直接下载没有安装的sdk和工具，如果用过bundle，npm install，你会更了解这是做什么的
有了这个功能，在服务上进行编译更方便，基本一个命令就搞定了

# 可视化编程
- 首先，scroll在编辑的时候可以滑动的
- 添加了blueprint mode，像x光一样，可用直接查看layout的全部的结构
- ConstraintLayout，关于这个，我想说，同学，你知道安利吗，不对，你知道c#、xib吗。再一次，google干了微软事。
上面的是调侃，其实我觉得ConstraintLayout以后会是首选的布局模式，就像Fragment一样，这是google对布局大的改进，减少布局层级，可视化编程，提高编程效率。和Databinding结合，借助Android Studio提供的工具，可用将程序员画布局中解脱出来，去关注逻辑上的实现。
- 接上个，Google丧心病狂的提供了普通布局转换到ConstraintLayout工具

# Editor
- 直接拖Firebase的代码到editor
- 不知道代码怎么用了，右键Find Sample Code，显示sample code
- Leak检查，静态引用了Context会显示警告
- annotitions， @WorkThread, @AnyThread, @RequiresApi，@Dimension，@Px
- @Keep 你懂的
- 生成动态权限代码，如果你Activity中使用了相机权限，但是没有对Android6.0的动态权限适配，可以直接使用Android Studio生成相关的代码
- 移除unused resource，没有用到的string可用一键删除了

# Expresso test
简单来说，录制对App的操作，然后播放，这不是monkey，播放脚本和屏幕大小无关。这会大大的减少初级测试人员，缩短测试时间。录制的脚本可用在云端测试，可用在任何尺寸的机器上测试。

总的来说，新版的Android Studio对开发者表现了极大的诚意。
Preview版本的Android Studio下载地址：<http://tools.android.com/recent>
Google I/O上对Preview 2.2/2.3版本的介绍：<https://www.youtube.com/watch?v=csaXml4xtN8>


