---
title: Dagger2 Module
date: 2016-05-03 21:23:16
tags:
    - Android
    - Dagger2
category: Android
---

# 新建工程
使用Android Studio，新建一个空白工程，最小支持Android 4.0

# 添加Dagger2 依赖
修改app目录下的build.gradle
```gradle
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

...

apply plugin: 'com.neenbedankt.android-apt'

...

dependencies {
...
    compile 'com.google.dagger:dagger:2.4'
    compile 'org.glassfish:javax.annotation:10.0-b28'
}
```

# 在空白工程上测试一下Dagger2
依赖注入的可以通过<https://github.com/android-cn/blog/tree/master/java/dependency-injection>来了解一下。
通过之前的了解，实现一个简单的例子
```java
HttpEngine.java
@Singleton
public class HttpEngine {

    @Inject
    public HttpEngine() {
    }
}

DataComponent.java
@Singleton
@Component
public interface DataComponent {
    void inject(MainActivity mainActivity);
}

MainActivity.java
public class MainActivity extends AppCompatActivity {

    @Inject HttpEngine mHttpEngine;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        DaggerDataComponent.builder().build().inject(this);
    }
}
```
# Dagger2自动生成的代码
上面的代码可以为MainActivity注入一个HttpEngine对象，现在有两个问题，怎么注入的，注入的是单例的对象吗。
查看Dagger2生成的代码
![](/images/dagger2_1.png)
首先查看MainActivity_MembersInjector.java
```java
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
  private final Provider<HttpEngine> mHttpEngineProvider;

  public MainActivity_MembersInjector(Provider<HttpEngine> mHttpEngineProvider) {
    assert mHttpEngineProvider != null;
    this.mHttpEngineProvider = mHttpEngineProvider;
  }

  public static MembersInjector<MainActivity> create(Provider<HttpEngine> mHttpEngineProvider) {
    return new MainActivity_MembersInjector(mHttpEngineProvider);
  }

  @Override
  public void injectMembers(MainActivity instance) {
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    instance.mHttpEngine = mHttpEngineProvider.get();
  }

  public static void injectMHttpEngine(
      MainActivity instance, Provider<HttpEngine> mHttpEngineProvider) {
    instance.mHttpEngine = mHttpEngineProvider.get();
  }
}
```
MainActivity_MembersInjector从名字就能看出，为MainActivity注入Members。在构造的时候传入一个Provider<HttpEngine>对象，在injectMembers函数中通过Provider的get获取一个HttpEngine对象，注入到MainActivity中，在MainActivity中，被赋值的成员函数用@Inject标识。到这里，明白了怎么向MainActivity注入Members的，但是Provider怎么提供要注入的对象的，还是不知道。

DaggerDataComponent.java
```java
public final class DaggerDataComponent implements DataComponent {
  private MembersInjector<MainActivity> mainActivityMembersInjector;

  private DaggerDataComponent(Builder builder) {
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static DataComponent create() {
    return builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {

    this.mainActivityMembersInjector =
        MainActivity_MembersInjector.create(HttpEngine_Factory.create());
  }

  @Override
  public void inject(MainActivity mainActivity) {
    mainActivityMembersInjector.injectMembers(mainActivity);
  }

  public static final class Builder {
    private Builder() {}

    public DataComponent build() {
      return new DaggerDataComponent(this);
    }
  }
}
```
在DaggerDataComponent的initialize()函数中，创建了一个 MainActivity_MembersInjector 的实例，传入的是一个 HttpEngine_Factory 的实例。在inject()函数中会调用MainActivity_MembersInjector的injectMembers()函数进行注入。


HttpEngine_Factory.java
```java
public enum HttpEngine_Factory implements Factory<HttpEngine> {
  INSTANCE;

  @Override
  public HttpEngine get() {
    return new HttpEngine();
  }

  public static Factory<HttpEngine> create() {
    return INSTANCE;
  }
```
HttpEngine_Factory提供了HttpEngine的构造方法，使用enum的方式保证单例模式。

# Provider，Injector，Component
至此，一个最简单的注入就完成了，通过上面的例子，可以将Dagger2的注入分为三个部分，Provider，Injector，Component。Provider提供对象或者说构造方法，Injector负责将Provider提供的对象进行注入，至于Injector要使用哪个Provider，由Component来进行指定。

上面的例子，有两个疑问：
1. 我们没有在任何地方手动的指定HttpEngine和DataComponent的关联，DaggerDataComponent是怎么找到HttpEngine的构造方法的呢？
2. 怎么使用Dagger2创建一个不是单例的Provider

第一个问题，和Android apt有关，以后慢慢研究。
第二个问题，可以使用 @Provides
```java
@mdule
public class DataModule {

    @Singleton
    @Provides
    HttpEngine provideHttpEngine() {
        return new HttpEngine();
    }
}
```
修改component，指定Module
```java
@Singleton
@Component(modules = DataModule.class)
public interface DataComponent {
    void inject(MainActivity mainActivity);

}
```
会生成一个新的class
```java
public final class DataModule_ProvideHttpEngineFactory implements Factory<HttpEngine> {
  private final DataModule module;

  public DataModule_ProvideHttpEngineFactory(DataModule module) {
    assert module != null;
    this.module = module;
  }

  @Override
  public HttpEngine get() {
    return Preconditions.checkNotNull(
        module.provideHttpEngine(), "Cannot return null from a non-@Nullable @Provides method");
  }

  public static Factory<HttpEngine> create(DataModule module) {
    return new DataModule_ProvideHttpEngineFactory(module);
  }
}
```
get方法获取的不再是单例的对象。
Module就像是一个Provider的仓库，我们可以定义多个仓库，可以在compont中指定使用那个仓库。

# Module
关于Module和@Provides，还有一些疑问：
1. 一个Compnot指定多个Module会是什么情况。
2. Module可以继承吗
3. 如何让@Provides作为一个单例

我们修改一下
HttpEngine.java
```java
public interface HttpEngine {
    String getTag();
}
```
添加
```java
public class NormalHttpEngine implements HttpEngine {

    @Override
    public String getTag() {
        return "NormalHttpEngine";
    }
}
```
```java
public class OkHttpEngine implements HttpEngine {

    @Override
    public String getTag() {
        return "OkHttpEngine";
    }
}
```
```java
@Module
public class DataModule2 {

    @Singleton
    @Provides
    HttpEngine provideHttpEngine() {
        return new NormalHttpEngine();
    }
}
```
DataComponent修改为
```java
@Singleton
@Component(modules = {DataModule.class, DataModule2.class})
public interface DataComponent {
    void inject(MainActivity mainActivity);
}
```
build，会出现error
```
Error:(15, 10) error: com.sw926.xyz.com.sw926.xyz.data.HttpEngine is bound multiple times:
@Provides com.sw926.xyz.com.sw926.xyz.data.HttpEngine com.sw926.xyz.com.sw926.xyz.data.DataModule.provideHttpEngine()
@Provides com.sw926.xyz.com.sw926.xyz.data.HttpEngine com.sw926.xyz.com.sw926.xyz.data.DataModule2.provideHttpEngine()
```
得出结论，可以指定多个Module，但Provider不能冲突。

修改DataModule2和DataComponent
```java
@Module
public class DataModule2 extends DataModule {

    @Provides
    HttpEngine provideHttpEngine() {
        return new NormalHttpEngine();
    }
}
```
```java
@Singleton
@Component(modules = DataModule2.class)
public interface DataComponent {
    void inject(MainActivity mainActivity);
}
```
build，仍然出现error
```
Error:(13, 16) error: @Provides methods may not override another method. Overrides: @Provides com.sw926.xyz.com.sw926.xyz.data.HttpEngine com.sw926.xyz.com.sw926.xyz.data.DataModule.provideHttpEngine()
```
**@Provides 的函数不能被override，但是可以被继承**

第三个问题，@Provides 前面加上 @Singleton并不会变成单例的模式，如果想使用单例，可以在Application中创建component，在Activity中进行注入,代码如下：
```java
public class App extends Application {

    private DataComponent mDataComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        mDataComponent = DaggerDataComponent.builder().build();
    }

    public DataComponent getDataComponent() {
        return mDataComponent;
    }
}
```
```java
public class MainActivity extends AppCompatActivity {

    @Inject HttpEngine mHttpEngine;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ((App)getApplication()).getDataComponent().inject(this);
    }
}
```

项目地址: <https://github.com/sw926/xyz/tree/dagger2>





