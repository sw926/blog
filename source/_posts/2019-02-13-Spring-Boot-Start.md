---
title: Spring Boot Start
date: 2019-02-13 10:30:56
category: Java
tags: 
    - Spring Boot
---

首先添加 Gradle 的插件:

``` gradle
buildscript {
    ext {
        springBootVersion = '2.0.6.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}
```

然后添加 starter：

``` gradle
// starter
compile 'org.springframework.boot:spring-boot-starter-web'
compile 'org.springframework.boot:spring-boot-starter-thymeleaf'
compile 'org.springframework.boot:spring-boot-starter-amqp'
compile 'org.springframework.boot:spring-boot-configuration-processor'
testCompile 'org.springframework.boot:spring-boot-starter-test'
```

**注意不要写版本号**

## mysql

创建用户：
``` sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

授权：
``` sql
GRANT all privileges ON databasename.tablename TO 'username'@'host';
flush  privileges;
```

说明:
 - privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
 - databasename：数据库名
 - tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*