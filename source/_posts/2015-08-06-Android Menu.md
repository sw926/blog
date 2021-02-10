---
title: Android Menu
date: 2015-08-06 12:06:54
tags: 
    - Android
category: Android
---
关于menu，在Activity中和Fragment中都搞不清楚。

在Activity中创建menu
``` java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_main, menu);
    return true;
}
```
onCreateOptionsMenu只会调用一次，返回值为true时显示菜单，为false不显示菜单
menu的xml文件如下
``` xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_new"
        android:orderInCategory="100"
        android:title="@string/new"
        app:showAsAction="never"/>
    <item
        android:id="@+id/action_delete"
        android:orderInCategory="101"
        android:title="@string/delete"
        app:showAsAction="never"/>
    <item
        android:id="@+id/action_exit"
        android:icon="@drawable/icon"
        android:orderInCategory="102"
        android:title="@string/exit"
        app:showAsAction="never"/>
</menu>
```
首先 **showAsAction** 有三个可选项，always，ifRoom，never
*always* 用于显示在actionbar上
*ifRoom* 如果空间不足时显示会显示在三个点的菜单中
*never* 永远显示在3个点中
**android:orderInCategory**是菜单的排序

``` java
@Override
public boolean onPrepareOptionsMenu(Menu menu) {
    menu.findItem(R.id.new).setVisible(true);
    return true;
}
```
每次显示菜单前调用，来控制菜单的显示隐藏，返回true显示菜单，返回false隐藏菜单。

```java
@Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        
            }
           
        }
        return super.onOptionsItemSelected(item);
    }
```