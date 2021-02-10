---
title: Dagger2笔记 Provider和Compoent
date: 2015-07-30 17:14:29
tags: 
    - Android
    - Dagger2
category: Android
---
在Android Studio中配置Dagger
```
...
apply plugin: 'com.neenbedankt.android-apt'

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

...

dependencies {
   ...
    apt 'com.google.dagger:dagger-compiler:2.0.1'
    compile 'com.google.dagger:dagger:2.0.1'
    compile 'org.glassfish:javax.annotation:10.0-b28'
}
```

我想在Activity中注入一个单例的HttpEngine，HttpEngine的代码如下
```java
@Singleton
public class HttpEngine {

    @Inject
    public HttpEngine() {
    }

    public void getData(GetDataCallback callback) {
        callback.onCallback("data from http");
    }

    public interface GetDataCallback {
        void onCallback(String data);
    }
}
```
Activity代码如下
```java
public class MainActivity extends AppCompatActivity {

    @Inject HttpEngine mHttpEngine;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.btn_get_data).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mHttpEngine.getData(new HttpEngine.GetDataCallback() {
                    @Override
                    public void onCallback(String data) {
                        new AlertDialog.Builder(MainActivity.this).setMessage(data).create().show();
                    }
                });
            }
        });
    }
}
```
我是这么想的，我用@Inject标识了HttpEngine的构造方法，然后使用
```java
@Inject HttpEngine mHttpEngine;
```
就成功注入了一个HttpEngine，这样我就可以使用这个对象了。结果当然是个空指针的错误。
怎么样能让Activity和HttpEngine关联起来呢？看一下Dagger2自动生成的代码
```java HttpEngine_Factory.java
@Generated("dagger.internal.codegen.ComponentProcessor")
public enum HttpEngine_Factory implements Factory<HttpEngine> {
INSTANCE;

  @Override
  public HttpEngine get() {  
    return new HttpEngine();
  }

  public static Factory<HttpEngine> create() {  
    return INSTANCE;
  }
}
```
Factory继承自Provider
```java
public interface Provider<T> {
    T get();
}
```

```java MainActivity_MembersInjector.java
@Generated("dagger.internal.codegen.ComponentProcessor")
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
  private final MembersInjector<AppCompatActivity> supertypeInjector;
  private final Provider<HttpEngine> mHttpEngineProvider;

  public MainActivity_MembersInjector(MembersInjector<AppCompatActivity> supertypeInjector, Provider<HttpEngine> mHttpEngineProvider) {  
    assert supertypeInjector != null;
    this.supertypeInjector = supertypeInjector;
    assert mHttpEngineProvider != null;
    this.mHttpEngineProvider = mHttpEngineProvider;
  }

  @Override
  public void injectMembers(MainActivity instance) {  
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    supertypeInjector.injectMembers(instance);
    instance.mHttpEngine = mHttpEngineProvider.get();
  }

  public static MembersInjector<MainActivity> create(MembersInjector<AppCompatActivity> supertypeInjector, Provider<HttpEngine> mHttpEngineProvider) {  
      return new MainActivity_MembersInjector(supertypeInjector, mHttpEngineProvider);
  }
}
```

```java MembersInjector.java
public interface MembersInjector<T> {
  void injectMembers(T instance);
}
```
HttpEngine_Factory一个Provider，负责提供HttpEngine的构造方法.
MainActivity_MembersInjector是一个MembersInjector，负责向MainActivity进行注入。
只要new一个MainActivity_MembersInjector对象，调用injectMembers方法，就能够将HttpEngine注入到MainActivity中。
我们需要创建一个Component，Component是Provider和Injector之间的桥梁。
```java ActivityComponent
@Singleton
@Component
public interface ActivityComponent {
    void inject(MainActivity activity);
}
```
Dagger2自动生成一个DaggerActivityComponent
```java DaggerActivityComponent.java
@Generated("dagger.internal.codegen.ComponentProcessor")
public final class DaggerActivityComponent implements ActivityComponent {
  private Provider<HttpEngine> httpEngineProvider;
  private MembersInjector<MainActivity> mainActivityMembersInjector;

  private DaggerActivityComponent(Builder builder) {  
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {  
    return new Builder();
  }

  public static ActivityComponent create() {  
    return builder().build();
  }

  private void initialize(final Builder builder) {  
    this.httpEngineProvider = ScopedProvider.create(HttpEngine_Factory.create());
    this.mainActivityMembersInjector = MainActivity_MembersInjector.create((MembersInjector) MembersInjectors.noOp(), httpEngineProvider);
  }

  @Override
  public void inject(MainActivity activity) {  
    mainActivityMembersInjector.injectMembers(activity);
  }

  public static final class Builder {
    private Builder() {  
    }
  
    public ActivityComponent build() {  
      return new DaggerActivityComponent(this);
    }
  }
}
```
在MainActivity中调用
```java
DaggerActivityComponent.create().inject(this);
```
就成功注入了HttpEngine，至此就理清了Provder和Component的关系。