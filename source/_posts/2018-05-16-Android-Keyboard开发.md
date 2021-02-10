---
title: Android Keyboard开发
date: 2018-05-16 16:17:43
category: Android
tags: 
    - Android
---

要开始做 Android 键盘的开发，最好首先参考一下 Api Demo 里面的 SoftKeyboard 例子。这个例子比较老了，导入到 Android Studio 之后会发现无法运行，这是因为没有 Launcher Activity，在 Run/Debug Configurations 里面把 Launch Options 设置为 Nothing 后就能运行了，当然运行后在桌面是看不懂图标的，需要在设置里面添加输入法，之后再输入文本的时候就能调出刚才安装的输入法了。
设置界面是在 AndroidManifest.xml 里面声明的：

``` xml
<activity
    android:name=".ImePreferences"
    android:exported="false"
    android:label="@string/settings_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
    </intent-filter>
</activity>
```

跳转到 ImePreferences.java，发现它是一个 Activity，继承自 PreferenceActivity，不过有个一个警告，意思是 PreferenceActivity 的子类不能 export，那么修改一一下 AndroidManifest.xml 

``` xml
<activity
    android:name=".ImePreferences"
    android:exported="false"
    android:label="@string/settings_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
    </intent-filter>
</activity>
```

警告消失了，这是个输入法的设置界面，总是到系统设置里面找不太方便，我们效仿其他输入法，在程序列表里面添加一个主界面，既然 ImePreferences 是 Activity 那么我们之间 start 这个 Activity 就可以了：

``` kotlin
val intent = Intent(this, ImePreferences::class.java)
startActivity(intent)
```


