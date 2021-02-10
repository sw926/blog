---
layout: post
title: 自定义View的measure
date: 2015-04-21
tags: 
    - Android
category: Android
---
onMeasure函数的两个参数，每个参数包含一个mode和一个size
``` Java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
}
```
通过以下方法可以取到
``` Java
int widthMode = MeasureSpec.getMode(widthMeasureSpec);
int heightMode = MeasureSpec.getMode(heightMeasureSpec);
int widthSize = MeasureSpec.getSize(widthMeasureSpec);
int heightSize = MeasureSpec.getSize(heightMeasureSpec);
```

mode有三种
``` Java
MeasureSpec.EXACTLY
MeasureSpec.AT_MOST
MeasureSpec.UNSPECIFIED
```

**MeasureSpec.EXACTLY**
控件的大小是固定的，如果layout_width或者layout_height是固定的大小或者**match_parent**

**MeasureSpec.AT_MOST**
指定了控件允许的最大的大小，例如layout_width或者layout_height是**wrap_content**，会给定一个最大允许的大小

**MeasureSpec.UNSPECIFIED**
没有指定大小

通过mode和size，计算控件所需的大小，调用 
``` Java
void setMeasuredDimension(int measuredWidth, int measuredHeight)
```
保存计算结果结果
