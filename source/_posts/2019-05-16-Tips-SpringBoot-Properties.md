---
title: 'Tips:SpringBoot Properties'
date: 2019-05-16 10:11:04
category: Java
tags: 
    - Spring Boot
---

新建项目里面都有一个 `application.properties`，我们可以在这个文件中添加一些属性，

```
test.foo=Hello
test.bar=World 
```

在组件中可以通过添加 `@Value` 注解来读取

``` java
@RestController
@RequestMapping("/test")
public class TestController {

    @Value("${test.foo}")
    private String foo;
    @Value("${test.bar}")
    private String bar;


    @GetMapping("/hello")
    public String hello() {
        return foo + " " + bar;
    }
}
```

`test/hello` 接口输出的是 `Hello World`，我们可以通过 Java 的 JVM Args 在运行的时候覆盖这些属性：

```
java -Dtest.bar=bar -jar xxx.jar
```

这样 `test/hello` 接口输出的是 `Hello bar`，那么 SystemProperties 有没有这些属性呢，实验一下，修改 `hello()` 为

``` java
@GetMapping("/hello")
public String hello() {
    return foo + " " + bar + ", from system: " +  SystemProperties.get("test.foo") + " " +  SystemProperties.get("test.bar");
}
```

输出结果为：`Hello bar, from system: null bar`，看来 `application.properties` 是不会写入 SystemProperties 的，另外 `@Value` 不是直接读取 `application.properties` 的值，而是好几个源，有优先级的。
`@Value` 可以注入一下的资源：

- 注入普通字符串
- 注入操作系统属性
- 注入表达式结果
- 注入其他Bean属性：注入beanInject对象的属性another
- 注入文件资源
- 注入URL资源

示例：

``` java
@Value("normal")
private String normal; // 注入普通字符串

@Value("#{systemProperties['os.name']}")
private String systemPropertiesName; // 注入操作系统属性

@Value("#{ T(java.lang.Math).random() * 100.0 }")
private double randomNumber; //注入表达式结果

@Value("#{valueTestBean.name}")
private String fromAnotherBean; // 注入其他Bean属性：注入beanInject对象的属性another，类具体定义见下面

@Value("classpath:resources/static/value_test.dat")
private Resource resourceFile; // 注入文件资源

@Value("http://www.baidu.com")
private Resource testUrl; // 注入URL资源
```
Properties 的加载顺序可以查看官方文档

https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config



