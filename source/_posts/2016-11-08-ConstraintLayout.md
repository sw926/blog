---
title: ConstraintLayout（约束布局）笔记
date: 2016-11-08 14:15:59
tags: 
    - Android
    - ConstraintLayout
category: Android
---
## 相对位置
类似于RelativeLayout
横轴（Horizontal Axis） Left, Right, Start, End
竖轴（Vertical Axis）Top, Bottom, Text Baseline
可用的属性有

```
layout_constraintLeft_toLeftOf
layout_constraintLeft_toRightOf
layout_constraintRight_toLeftOf
layout_constraintRight_toRightOf
layout_constraintTop_toTopOf
layout_constraintTop_toBottomOf
layout_constraintBottom_toTopOf
layout_constraintBottom_toBottomOf
layout_constraintBaseline_toBaselineOf
layout_constraintStart_toEndOf
layout_constraintStart_toStartOf
layout_constraintEnd_toStartOf
layout_constraintEnd_toEndOf
```
## Margins(边距)
提供View.GONE时的间距

```
layout_goneMarginStart
layout_goneMarginEnd
layout_goneMarginLeft
layout_goneMarginTop
layout_goneMarginRight
layout_goneMarginBottom
```
## 居中和偏差
设置好位置后默认是居中的

```
<android.support.constraint.ConstraintLayout ...>
             <Button android:id="@+id/button" ...
                 app:layout_constraintLeft_toLeftOf="parent"
                 app:layout_constraintRight_toRightOf="parent/>
         </>
```
## 偏差（Bias）
默认是居中的可以使用一下两个参数调整

```
layout_constraintHorizontal_bias
layout_constraintVertical_bias
```
例如

```
<android.support.constraint.ConstraintLayout ...>
             <Button android:id="@+id/button" ...
                 app:layout_constraintHorizontal_bias="0.3"
                 app:layout_constraintLeft_toLeftOf="parent"
                 app:layout_constraintRight_toRightOf="parent/>
         </>
```
## 可见行为（Visibility behavior）
GONE控件在一般情况下不会显示和占用布局空间，但是他们的尺寸是保留的

* 但是在计算布局时，GONE控件会计算在内
* 如果有其他控件被GONE的控件约束，这种约束仍然存在，但是所有的margins都会变为0
* 如果想保留margins，使用layout_goneMargin*属性
## 尺寸约束（Dimensions constraints）
### 最小尺寸约束
```
android:minWidth
android:minHeight
```
在WRAP_CONTENT是有效
### 空间尺寸约束
对于androdi:layout_width和android:layout_height有三种方式
* 使用一个固定的尺寸，例如123dp
* 使用WRAP_CONTENT
* 使用0dp, 相当于MATCH_CONSTRAINT
**MATCH_PARENT不支持约束布局！！！**
MATCH_CONSTRAINT的left/right top/bottom设置为parent

## 比例（Ratio）
```
 <Button android:layout_width="wrap_content"
                   android:layout_height="0dp"
                   app:layout_constraintDimensionRatio="1:1" />
```
值可以是一个float，页可以是“width:height”的形式
也可以把宽和高都设为MATCH_CONSTRAINT (0dp)，设置在Ratio的时候添加“W”或“H”以适应宽度活高度
例如
```
<Button android:layout_width="0dp"
                   android:layout_height="0dp"
                   app:layout_constraintDimensionRatio="H,16:9"
                   app:layout_constraintBottom_toBottomOf="parent"
                   app:layout_constraintTop_toTopOf="parent"/>
```
按钮的比例是16:9，宽度会匹配parent的约束
## 链（Chains）
比较难理解，简单的认为是一组控件在横轴或纵轴首尾相连，就是一条链
### 创建链 
一组控件在一个方向，首尾的控件连接到parent，中间的控件双向连接，如下图
![chains](/images/chains.png)
### Chain Header
链被第一个控件控制，也就是最左边或者最顶端的
### Margins in chains
如果在连接上指定边距，会计算这些边距。在spread chains中，边距会从分配的控件中扣除
### Chain Style
为header设置layout_constraintHorizontal_chainStyle或者layout_constraintVertical_chainStyle

* CHAIN_SPREAD -- 元素将被展开
* Weighted chain -- 在 CHAIN_SPREAD 模式, 如果有控件为MATCH_CONSTRAINT，这个控件会占据所有可用的空间
* CHAIN_SPREAD_INSIDE -- 两端不会分配空间
* CHAIN_PACKED -- 所有元素打包在一起，空间分配到两端
![chains](/images/chains-styles.png)
### Weighted chains
spreed chain默认所有元素占据他们所需的空间，如果有元素为MATCH_CONSTRAINT，他们会使用所有的空白区域，使用layout_constraintHorizontal_weight和layout_constraintVertical_weight可用控制他们的比例。
