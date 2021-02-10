---
layout: post
title: Android Studio工程使用Grade Wrapper
tags: 
    - Android
    - Gradle
category: Android
---
配置Gradle的环境很简单，从网上下载Gradle包，解压到任意文件夹，将bin目录添加到环境变量，输入gradle -v，如果能正确看到版本号，说明gradle配置成功

```bash
$ gradle -v

------------------------------------------------------------
Gradle 2.3
------------------------------------------------------------

Build time:   2015-02-16 05:09:33 UTC
Build number: none
Revision:     586be72bf6e3df1ee7676d1f2a3afd9157341274

Groovy:       2.3.9
Ant:          Apache Ant(TM) version 1.9.3 compiled on December 23 2013
JVM:          1.7.0_71 (Oracle Corporation 24.71-b01)
OS:           Mac OS X 10.10.2 x86_64
```

在导入工程的时候选择**Use local gradle distribution**，可以使用本地的Gradle，Android Studio会忽略Gradle Wrapper。

但是推荐使用Gradle Wrapper，Gradle Wrapper为工程指定了一个Gradle版本，导入工程的时候，会自动下载对应版本的Gradle，这样在team中就不会出现多个版本的Gradle。

使用Android Studio新建工程的时候会自动生成Wrapper，在根目录下生成一个gradle文件夹，里面有gradle-wrapper.jar和gradle-wrapper.properties两个文件，gradle-wrapper.properties内容如下

    #Tue Apr 07 14:27:52 CST 2015
    distributionBase=GRADLE_USER_HOME
    distributionPath=wrapper/dists
    zipStoreBase=GRADLE_USER_HOME
    zipStorePath=wrapper/dists
    distributionUrl=https\://services.gradle.org/distributions/gradle-2.3-bin.zip

distributionUrl指定了gradle的下载地址

如果工程根目录中没有gradle这个文件夹，可以使用命令生成wrapper
```bash
$ gradle init wrapper
```
使用的是你本地gradle的版本

如果想指定gradle的版本，在build.gradle中添加一个task

    task wrapper(type: Wrapper) {
        gradleVersion = '2.2.1'
    }

执行

```bash
$ gradle wrapper
```

wrapper会修改为刚才指定的版本

使用了gradle wrapper，在命令行下，应该使用gradlew，例如打包的时候，应该
```bash
$ ./gradlew assembleRelease
```
这样使用的时wrapper指定的gradle版本进行编译。

导入工程的时候，Gradle project应该选择工程的根目录，有时候后面会跟上gradle，应该去掉，
然后选择Use default gradle wrapper(recommended)

点击OK，然后就悲剧的开始读条，显示正在下载gradle。。。

一个gradle的二进制zip包有40多个M，没有一个好翻墙渠道，那要下载好一会了，而且只读条，没有进度，不知道什么时候下载好，这个忍不了！
mac下，gradle的wrapper文件在**~/.gradle/wrapper**目录下，目录结构如下
```bash
[~/.gradle/wrapper] $ tree
.
└── dists
    └── gradle-2.3-bin
        └── a48v6zq5mdp1uyn9rwlj56945
            ├── gradle-2.3-bin.zip.lck
            └── gradle-2.3-bin.zip.part

3 directories, 2 files
```
这是下载到一半的情况，我实在忍受不了，说以强制退出了。通过上面的目录结构，结合

    https://services.gradle.org/distributions/gradle-2.3-bin.zip
    
可以推断出gradle wrapper在下载的时候临时文件的保存位置，`a48v6zq5mdp1uyn9rwlj56945`是url的hash，至于怎么得出的，通过分析gradle的源码，找到了urlhash的计算方法

```java
public static String getHash(String string) {
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            byte[] bytes = string.getBytes();
            messageDigest.update(bytes);
            return new BigInteger(1, messageDigest.digest()).toString(36);
        } catch (Exception e) {
            throw new RuntimeException("Could not hash input string.", e);
        }
    }
```

下载对应的gradle的zip包，拷贝到wrapper目录，在工程目录下执行./gradlew assembleRelease，应该不会看到下载gradle的信息了。