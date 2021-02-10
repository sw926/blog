---
title: Android Databinding编辑xml时进行预览
date: 2016-09-20 11:30:03
tags: 
    - Android
    - Android Studio
    - Databinding
category: Android
---

使用了Databinding后，编辑xml的时候，一些属性就无法预览了，比如TextView的text，ImageView的src。这时可以使用Designtime Layout Attributes，参考

<http://tools.android.com/tips/layout-designtime-attributes>

<http://tools.android.com/tech-docs/tools-attributes>

例如

```
<TextView
...
android:text="@{viewModel.text}"
tools:text="这是标题"
/>
```
这个属性只会在Android Studio预览时有效，预览时会覆盖 android: 属性，编译时会删除，不用担心自己预览效果时随便填的文本会被发布出去。而且可以预览ListView的item，
container的Fragment，merge。

使用Databinding时需要注意，声明prefix的时候必须放在xml的根节点，也就是 layout节点，包括xmlns:android，xmlns:app，xmlns:tools，不然Designtime Layout Attributes在Preview时显示不出来

```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    ...
</layout>
```