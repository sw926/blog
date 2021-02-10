---
title: 读懂 Gradle 的 DSL
date: 2019-02-02 12:16:42
category: Android
tags: 
    - Android
    - Gradle
---

现在 Android 开发免不了要和 Gradle 打交道，所有的 Android 开发肯定都知道这么在 `build.gradle` 中添加依赖，或者添加配置批量打包，但是真正理解这些脚本的人恐怕很少。其实 Gradle 的 `build.gradle` 可以说是一个代码文件，熟悉 Java 的人理解起来很简单的，之所以不愿意去涉及，主要感觉没有必要去研究。要能看懂 `build.gradle`，除了要了解 Groovy 的语法，还要了解 Gradle 的构建流程，要研究还是要花一些时间的，所以这篇文章可以让一个 Java 程序员在一个小时内看懂 Gradle 的脚本。

## Gradle 简单介绍

Gradle 构建由 Project 和 Task 组成，Project 保存项目的属性，例如 name，版本号，代码文件位置。Task 也是 Project 的一部分，但是它是可执行的任务，我们最常使用的 `build` 就是一个 Task，Task 可以依赖于另外一个 Task，一个 Task 在执行的时候，它依赖的 Task 会先执行。这样，当我们 build 的时候，这个 Task 可能依赖很多的 Task，比如代码检查、注解处理，这样一层层的依赖，最终通过 build Task 全部执行。

## Gradle 和 Groovy

Gradle 和 Groovy 这两个名字很容易让人产生混淆，这里先解释一下，Groovy 是一门编程语言，和 Java 一样。Gradle 和一个自动化构建工具，其他知名的构建工具还有 Maven 和 Ant。什么自动化构建工具？用 Android 来举例，打包一个 Apk 文件要做很多工作，代码预处理，lint代码检查、处理资源、编译 Java 文件等等，使用自动化构建工具，一个命令就可以生成 Apk 了。

Gradle 的 DSL 目前支持两种语言的格式，Groovy 和 Kotlin，Kotlin 格式的 DSL 是在 5.0 引入的，相比 Groovy，Kotlin 使用的人数更多，也更好理解，在这儿主要介绍 Groovy 格式的 DSL。

介绍一下什么是 DSL，DSL 是 `Domain Specific Language` 的缩写，既领域专用语言。Gradle 的 DSL 专门用于配置项目的构建，不能做其他工作，而像 Java 、C/C++ 这些就属于通用语言，可以做任何工作。

我们还要理解什么是脚本文件。在写代码 Java 代码时，程序是从 `main()` 函数开始执行的，只有在 `main()` 中调用的代码才会执行。但是脚本文件不一样，只要在文件内写的代码都会执行，`Groovy` 是支持脚本文件的，我们配置好 Groovy 的开发环境，新建一个文件 `test.groovy`，输入以下内容：

``` groovy
String hello = "Hello World!"
println(hello)

println("The End")
```

然后运行：

``` sh
groovy test.groovy
```

输出结果为：

``` sh
Hello World!
The End
```

虽然没有 `main` 函数，但是里面的代码都执行了。很明显，`build.gradle` 就是一个 Groovy 的脚本文件，里面就是 Groovy 代码，里面添加的所有代码都会运行，我们可以试验以下，随便打开一个 Gradle 格式的项目，在 `build.gradle` 最下面添加一些 Java 代码：

``` java
String hello = "Hello World!"
System.out.println(hello)
```

然后执行：

``` sh
./gradlew -q # -q 是不输出额外的信息
```

我们会看到输出了 `Hellow World`，说明我们添加的代码被执行了，那么为什么可以在 `build.gradle` 里面写 Java 代码呢？这是因为 Groovy 是支持 Java 的语法的，在 Groovy 文件写 Java 代码是完全没有问题的。

## `build.gradle` 的执行方式

现在总结一下，`build.gradle` 就是一个 Groovy 格式脚本文件，里面是 Groovy 或者 Java 代码，构建的时候会顺序执行，但是打开 `build.gradle`，可能还是一头雾水，一个个字符和大括号组成的东西到底是什么鬼？我们来看一下最长使用的 `dependencies`：

``` gradle
dependencies {
    // This dependency is found on compile classpath of this component and consumers.
    implementation 'com.google.guava:guava:26.0-jre'

    // Use JUnit test framework
    testImplementation 'junit:junit:4.12'
}
```

`implementation` 也可以这样写：

``` gradle
implementation('com.google.guava:guava:26.0-jre')
```

`implementation` 其实就是一个函数，在 Groovy 中，函数调用可以使用空格加参数的形式调用，例如

``` groovy
void foo(String params1, int param2) {
    println("param1 = $params1, param2 = $param2")
}

foo "aaa", 2
```

`implementation 'com.google.guava:guava:26.0-jre'` 就是调用了 `implementation` 函数添加了一个依赖。以此类推，`dependencies` 也是一个函数，在 `IDEA` 中，我们可以直接 `Ctrl` 加鼠标左键点击进去看它的声明：

``` java
public interface Project extends Comparable<Project>, ExtensionAware, PluginAware {
    // ...
    void dependencies(Closure configureClosure);
    // ...
}
```

我们看到 `dependencies` 是 `Project` 一个方法，为什么可以在 `build.gradle` 调用 `Project` 的方法呢，官方文档里面有相关的介绍。一个 Gradle 项目一般有一个 `settings.gradle` 文件和一个 `build.gradle` 文件，`settings.gradle` 用来配置目录结构，子工程就是在 `settings.gradle` 里面配置，`Project` 和 `build.gradle` 是一一对应的关系，Gradle 的构建流程如下：

 1、生成一个 `Settings` 对象，执行 `settings.gradle` 对这个对象进行配置
 2、使用 `Settings` 对象生成工程结构，创建 `Project` 对象
 3、对所有 `Project` 执行对应的 `build.gradle` 进行配置

`build.gradle` 就是对 `Project` 的操作，例如，在 `build.gradle` 中输入以下代码

``` groovy
println "project name is ${this.name}"
```

输出结果为： `project name is java_demo`，`java_demo` 就是我们的 project name，我们可以认为对 `this` 的操作就是对 `project` 的操作。

Groovy 也是有语法糖的，类的属性可以直接使用名字，例如 `Project` 的有两个函数：

``` java
Object getVersion();
void setVersion(Object version);
```

那么这就说明 `Project` 有一个 `version` 属性，在 `build.gradle` 中我们可以这样来使用：

``` gradle
version = "1.0" // 赋值，调用 setVersion()
println version // 读取，调用 getVersion()
```

在 `Project` 中没有 `getter` 方法的属性是不能赋值的，例如 `name`，我们可以输出 `name` 的值，但是 `name = "demo"` 是错误的。

所以，在 `build.gradle` 中的代码就是修改 `Project`，方式就是修改属性或者调用相关的方法，`plugins` 方法是添加插件，`repositories` 方法是添加代码仓库，

## Groovy 闭包

闭包可以认为是可以执行的代码块，Groovy 中闭包的声明和执行方式如下：

``` groovy
Closure closure = { String item ->
    println(item)
}

closure("Hello") // 执行
```

和 Lambda 表达式很像，但是 Groovy 的闭包可以先声明，然后设置代理来执行，例如我们声明一个闭包：

``` groovy
Closure closure = {
    sayHello()
}
```

这个闭包里面执行了 `sayHello()` 函数，但是我们没有在任何地方声明这个函数，在 Java 中，这是个编译错误，但是 Groovy 是允许的，完整的执行的例子如下：

``` groovy
Closure closure = {
    sayHello()
}
class Foo {
    void sayHello() {
        println("Hello!!!")
    }
}
def foo = new Foo()

closure.delegate = foo
closure()
```

输出结果为：

```
Hello!!! 
```

我们为闭包设置了一个代理 `delegate`，只要这个代理有 `sayHello()` 方法，代码就能执行，这就是为什么我们查看 `Project` 的源码，里面很多函数参数类型都是 `Closure`，例如：

```
void repositories(Closure configureClosure);
void dependencies(Closure configureClosure);
```

`repositories` 在 `build.gradle` 中是这样调用的：

``` gradle
repositories {
    // Use jcenter for resolving your dependencies.
    // You can declare any Maven/Ivy/file repository here.
    jcenter()
}
```

我们通过 IDE 进入 `jcenter()` 的声明，进入的是：

``` java
public interface RepositoryHandler extends ArtifactRepositoryContainer {
    // ...
}
```

由于没看过源码，我也只能猜，我猜 `repositories` 这个闭包的 `delegate` 是一个 `RepositoryHandler`，通过执行 `RepositoryHandler` 的方法，为工程添加 `Repository`

## Plugin

来看我们使用最多的 `dependencies`

``` gradle
dependencies {
    // This dependency is found on compile classpath of this component and consumers.
    implementation 'com.google.guava:guava:26.0-jre'

    implementation('com.google.guava:guava:26.0-jre')

    // Use JUnit test framework
    testImplementation 'junit:junit:4.12'
}
```

在 Java 和 Android 项目中 `implementation` 是一定会用到的，但是一个 Gradle Basic 项目是没有 `implementation` 的，实际上，在 `dependencies` 是不能直接添加任何依赖的。

这里我们有说一下 Gradle 怎么解决依赖。

Gradle 空白项目没有编译 Java 项目的能力，但是它能从仓库下载依赖的库并且配置到 `Project` 中。在我们编译 Java 项目的时候，一个配置是不够的，至少要有个测试版，正式版，两个版本依赖的库可能是不一样的，两个版本部分代码也是不一样的，那么我们怎么区分呢？在 Gradle 中，是通过 `configurations`，也就是配置，每个配置可以单独的添加依赖，在编译的时候，也就是执行某个 Task 的时候，通过读取配置中的依赖来添加 `classpath`，例如：

``` gradle
repositories {
    mavenCentral()
}

configurations {
    test
    release
}

dependencies {
    test 'org.apache.commons:commons-lang3:3.0'
    release 'org.slf4j:slf4j-log4j12:1.7.2'
}



task buildTest {
    doLast {
        println configurations.test.name
        println configurations.test.asPath
    }
} 
```

执行 ` ./gradlew buildTest -q`，输出结果为：

```
test
/Users/xxx/.gradle/caches/modules-2/files-2.1/org.apache.commons/commons-lang3/3.0/8873bd0bb5cb9ee37f1b04578eb7e26fcdd44cb0/commons-lang3-3.0.jar
```

如果在 `buildTest` 这个 Task 中进行编译工作的话，我们就可以直接读取 `configurations.test` 的路径设置为 `classpath`。

`implementation` 就是通过添加了一个 `implementation` 配置来实现的。这个配置是通过：

``` gradle
plugins {
    // Apply the java plugin to add support for Java
    id 'java'

    // Apply the application plugin to add support for building an application
    id 'application'
}
```

添加的，我们通过 `plugins` 可以给 `Project` 添加属性，Tasks，配置，例如我们写一个最简单的插件：

``` groovy
package com.demo

import org.gradle.api.Plugin
import org.gradle.api.Project

class DemoPlugin implements Plugin<Project> {
    void apply(Project project) {

        project.task("hello") {
            doLast {
                println "Hello World"
            }
        }

        project.configurations {
            demoCompile
        }
    }
}
```

这个插件为 `Project` 添加了一个 Task，添加了一个配置，我们将这个文件 `DemoPlugin.groovy` 放在项目根目录下的 `buildSrc/src/main/groovy/demo/` 下，就可以在 `build.gradle` 中直接使用了：

``` gradle
apply plugin: com.demo.DemoPlugin
```

## buildscript 

对于 `buildscript`，例如：

``` gradle
buildscript {
    repositories {
        mavenCentral()
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.3.0'
    }
}
```

它的作用是为构建脚本提供依赖，例如我们在项目中使用了 Android 的 Plugin，这个 Plugin 的要从哪找下载？这就需要在 buildscript 中指定。

