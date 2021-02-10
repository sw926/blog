---
layout: post
title: "Android Studio使用greenDao"
date: 2015-05-25 15:46:12 +0800
comments: true
tags: 
    - Android
    - Android Studio
category: Android
---
在工程的根目录下新建一个目录，作为DaoGenerator的目录，
```sh
$ mkdir -p ./MyDaoGenerator/src/main/java
```
目录结构如下
```sh
.
└── src
    └── main
        └── java
```
java目录下就是存放代码的目录，在java目录下新建一个包，
```sh
$ mkdir -p com/example/mydaogenerator
```
新建一个有main函数的class备用
```java
public class MyDaoGenerator {
    public static void main(String[] args) throws Exception {
    }
}
```
在MyDaoGenerator目录下添加build.gradle，填入以下内容
```
apply plugin: 'java'
apply plugin: 'application'

applicationName = 'MyDaoGenerator'
mainClassName = 'com.example.mydaogenerator.MyDaoGenerator'

repositories {
    mavenCentral()
}

dependencies {
    compile('de.greenrobot:DaoGenerator:+')
}
```
MyDaoGenerator目录结构如下
```
.
├── build.gradle
└── src
    └── main
        └── java
            └── com
                └── example
                    └── mydaogenerator
```

添加了build.gradle之后，Studio会提示是否添加新的gradle文件，添加同步之后，新建module就添加到工程中了，
如果错过了添加的提示，就在工程根目录下面的settings.gradle后面加上
```
include ':MyDaoGenerator'
```

DaoGenerator就搭建好了，打开Android Studio的task菜单，执行application run
![task](/images/mydaogenrator_task.png)


一个完整的DaoGenrator如下
```java
package com.example.mydaogenerator;

import de.greenrobot.daogenerator.DaoGenerator;
import de.greenrobot.daogenerator.Entity;
import de.greenrobot.daogenerator.Schema;

public class MyDaoGenerator {
    public static void main(String[] args) throws Exception {
        Schema schema = new Schema(1, "com.example.greendao.dao");
        addNote(schema);
        new DaoGenerator().generateAll(schema, "../app/src/main/java");
    }

    private static void addNote(Schema schema) {
        Entity note = schema.addEntity("Notes");
        note.addIdProperty();
        note.addStringProperty("content");
    }
}
```


