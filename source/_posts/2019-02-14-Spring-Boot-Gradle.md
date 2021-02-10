---
title: Spring Boot Gradle
date: 2019-02-14 10:34:46
category: Java
tags: 
    - Spring Boot
    - Gradle
---

翻译自官方文档：

## 添加插件

必须要添加的是 `org.springframework.boot`，这个插件已经上传到 Gradle 插件网站上，可以直接添加：

``` groovy
plugins {
    id 'org.springframework.boot' version '2.1.0.RELEASE'
}
```

这个插件会自动检测工程中的其他插件，从而做出对应的行为，例如添加了 java 插件，就有有生成 jar 文件的 task，使用 java 一般至少要添加：

``` groovy
apply plugin: 'java'
apply plugin: 'io.spring.dependency-management' 
```

## 依赖管理

Spring Boot 通过 starter 添加一系列的依赖，这些依赖的版本通过插件和 starter 指定，在项目中添加 starter 的时候不要写版本号，例如：

``` groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

如果要修改其中一个特定依赖的版本，可以使用下面的方式：

``` groovy
ext['slf4j.version'] = '1.7.20'
```

完整的列表在：https://github.com/spring-projects/spring-boot/blob/v2.1.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml


