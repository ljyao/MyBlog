---
title: MeasureSpec
date: 2016-07-26 10:43:15
category: android
tags: view
---
最近发现，重写了viewGroup的onMeasure方法，造成子view match_parent不起作用，代码如下：
```
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = (int) (width / 0.75f);
        super.onMeasure(width, height);
        setMeasuredDimension(width, height);
    }
```
<!-- more -->
功能为实现一个 宽：高为3/4的自定义view，发现view match_parent失效，变成自适应。

虽然使用setLayoutParams,可以实现同样功能，但是会重绘一次，觉得性能差些，最后还是想通过onMeasure实现，通过查阅一些资料，找到问题所在。

 先看MeasureSpec.getSize()源码：
```
 measureSpec & ~MODE_MASK
 int MODE_MASK  = 0x3 << MODE_SHIFT;
```
就是去掉前2位模式，得到实际值，当我计算后把这个值传给子View时，前2位为00，即UNSPECIFIED模式，造成子view自适应。

解决办法就是加上模式值：
heightSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);
# MeasureSpec官方文档
A MeasureSpec encapsulates the layout requirements passed from parent to child. Each MeasureSpec represents a requirement for either the width or the height. A MeasureSpec is comprised of a size and a mode. There are three possible modes:

UNSPECIFIED：
The parent has not imposed any constraint on the child. It can be whatever size it wants.
mode UNSPECIFIED = 0 << 30;

EXACTLY：
The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
mode EXACTLY = 1 << 30;

AT_MOST：
The child can be as large as it wants up to the specified size.
mode AT_MOST = 2 << 30;

MeasureSpecs are implemented as ints to reduce object allocation. This class is provided to pack and unpack the <size, mode> tuple into the int.

前2位表示模式，