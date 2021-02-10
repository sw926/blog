---
title: Android Studio编辑器一直显示错误的问题
date: 2019-01-14 11:08:07
category: Android
tags: 
    - Android
    - Android Studio
---

周一上班打开电脑，打开 Android Studio，显示有几个文件出现错误，又是同样的问题，直接 Build，没有任何错误。

这在 Android Studio 上经常遇到，应该是编辑器的 Index 没有刷新，解决方法很简单：

``` 
File -> Invalidate Caches/Restart
```

有几个选项：

 - Just Restart
 - Invalidate
 - Invalidate and Restart

最后一个选项肯定是能解决问题的，但是会重启，重新建立索引，重新建立缓存，项目比较大的话，10分钟没有了，电脑风扇还会呼呼的转。为了不重启就能解决这个问题，先选择 `Invalidate` 试试，点击之后没有反应，直接运行，错误还变多了，现在编辑器显示的错误是找不到一下的引用：

``` kotlin
import android.support.v4.app.Fragment
import androidx.navigation.findNavController
import kotlinx.android.synthetic.main.fragment_home.*
```

项目代码是没有问题的，可以运行，那么就是 Android Studio 的问题了，应该和 Gradle 有关，

``` kotlin
import android.support.v4.app.Fragment
import androidx.navigation.findNavControlle
```

是在 Gradle 文件中配置的引用。

``` kotlin
import kotlinx.android.synthetic.main.fragment_home.*
```

是 Gradle 插件生成的代码。

那么，执行一下 `Sync Project with Gradle Files`，问题解决了，下次就不用重启 Android Studio 了，先执行：

```
File -> Invalidate Caches/Restart -> Invalidate
```

然后

```
Sync Project with Gradle Files
```



