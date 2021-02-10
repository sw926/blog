---
title: 'Tips:sdkman'
date: 2019-06-11 10:32:37
category: Other
tags: 
    - sdkman
---

# 安装

`https://sdkman.io/install`

``` sh
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
$ sdk version
```

# 安装 JDK

``` sh
sdk list java
```

输出：

``` sh
================================================================================
Available Java Versions
================================================================================
     19.0.0-grl          11.0.2-zulufx       1.0.0-rc-16-grl
     13.ea.24-open       10.0.2-zulu
     12.0.1-sapmchn      10.0.2-open
     12.0.1-zulu         9.0.7-zulu
     12.0.1-open         9.0.4-open
     12.0.1.j9-adpt      8.0.212-zulu
     12.0.1.hs-adpt      8.0.212-amzn
     12.0.1-librca       8.0.212.j9-adpt
     11.0.3-librca       8.0.212.hs-adpt
     11.0.3-sapmchn      8.0.212-librca
     11.0.3-zulu         8.0.202-zulu
     11.0.3-amzn         8.0.202-amzn
     11.0.3.j9-adpt      8.0.202-zulufx
     11.0.3.hs-adpt      7.0.222-zulu
     11.0.2-open         7.0.181-zulu

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```

大部分人应该都是使用的 Oracle 版本的 JDK，各个版本的区别可以参考文章：

```
https://www.oschina.net/news/99836/time-to-look-beyond-oracles-jdk
```

然后安装：

```
sdk install java 8.0.212-zulu
```