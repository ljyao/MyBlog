---
title: Property Animation and View Animation
date: 2016-08-01 14:42:51
category: android
tags: animation
---
一、Property Animation
创建一个动画通过修改对象属性在一段时间。
二、View Animation
包含2种类型：
Tween animation（补间动画）: 
创建一个动画通过执行一系列变化在一个图像上。
Frame animation（帧动画）:
创建一个动画通过顺序播放一系列图片。

<!-- more -->
# Property Animation

```
<set
  android:ordering=["together" | "sequentially"]>

    <objectAnimator
        android:propertyName="string"
        android:duration="int"
        android:valueFrom="float | int | color"
        android:valueTo="float | int | color"
        android:startOffset="int"
        android:repeatCount="int"
        android:repeatMode=["repeat" | "reverse"]
        android:valueType=["intType" | "floatType"]/>

    <animator
        android:duration="int"
        android:valueFrom="float | int | color"
        android:valueTo="float | int | color"
        android:startOffset="int"
        android:repeatCount="int"
        android:repeatMode=["repeat" | "reverse"]
        android:valueType=["intType" | "floatType"]/>

    <set>
        ...
    </set>
</set>


```
# View Animation
View动画框架支持补间动画和帧动画，它们都是被定义在XML中。

## Tween animation（补间动画）: 
一种定义在XML中的动画，可以生动的执行过渡动画,像：旋转，逐渐消失，移动，伸展动画。

### 文件位置
res/anim/filename.xml

### 语法：
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@[package:]anim/interpolator_resource"
    android:shareInterpolator=["true" | "false"] >
    <alpha
        android:fromAlpha="float"
        android:toAlpha="float" />
    <scale
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        android:pivotX="float"
        android:pivotY="float" />
    <translate
        android:fromXDelta="float"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float" />
    <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotX="float"
        android:pivotY="float" />
    <set>
        ...
    </set>
</set>
```
文件必需包含一个根元素:  <alpha>, <scale>, <translate>, <rotate>中一个, 或是 <set> 元素， 她包含着一组或多组其他元素(甚至嵌套 <set> 元素.

### 元素:
<set>
包含其他动画元素(<alpha>, <scale>, <translate>, <rotate>) 或其他 <set> 元素. 代表一个AnimationSet.
attributes:

android:interpolator
Interpolator resource. An Interpolator to apply on the animation. The value must be a reference to a resource that specifies an interpolator (not an interpolator class name). There are default interpolator resources available from the platform or you can create your own interpolator resource. See the discussion below for more about Interpolators.
android:shareInterpolator
Boolean. "true" if you want to share the same interpolator among all child elements.
<alpha>
A fade-in or fade-out animation. Represents an AlphaAnimation.
attributes:

android:fromAlpha
Float. Starting opacity offset, where 0.0 is transparent and 1.0 is opaque.
android:toAlpha
Float. Ending opacity offset, where 0.0 is transparent and 1.0 is opaque.
For more attributes supported by <alpha>, see the Animation class reference (of which, all XML attributes are inherrited by this element).

<scale>
A resizing animation. You can specify the center point of the image from which it grows outward (or inward) by specifying pivotX and pivotY. For example, if these values are 0, 0 (top-left corner), all growth will be down and to the right. Represents a ScaleAnimation.
attributes:

android:fromXScale
Float. Starting X size offset, where 1.0 is no change.
android:toXScale
Float. Ending X size offset, where 1.0 is no change.
android:fromYScale
Float. Starting Y size offset, where 1.0 is no change.
android:toYScale
Float. Ending Y size offset, where 1.0 is no change.
android:pivotX
Float. The X coordinate to remain fixed when the object is scaled.
android:pivotY
Float. The Y coordinate to remain fixed when the object is scaled.
For more attributes supported by <scale>, see the Animation class reference (of which, all XML attributes are inherrited by this element).

<translate>
A vertical and/or horizontal motion. Supports the following attributes in any of the following three formats: values from -100 to 100 ending with "%", indicating a percentage relative to itself; values from -100 to 100 ending in "%p", indicating a percentage relative to its parent; a float value with no suffix, indicating an absolute value. Represents a TranslateAnimation.
attributes:

android:fromXDelta
Float or percentage. Starting X offset. Expressed either: in pixels relative to the normal position (such as "5"), in percentage relative to the element width (such as "5%"), or in percentage relative to the parent width (such as "5%p").
android:toXDelta
Float or percentage. Ending X offset. Expressed either: in pixels relative to the normal position (such as "5"), in percentage relative to the element width (such as "5%"), or in percentage relative to the parent width (such as "5%p").
android:fromYDelta
Float or percentage. Starting Y offset. Expressed either: in pixels relative to the normal position (such as "5"), in percentage relative to the element height (such as "5%"), or in percentage relative to the parent height (such as "5%p").
android:toYDelta
Float or percentage. Ending Y offset. Expressed either: in pixels relative to the normal position (such as "5"), in percentage relative to the element height (such as "5%"), or in percentage relative to the parent height (such as "5%p").
For more attributes supported by <translate>, see the Animation class reference (of which, all XML attributes are inherrited by this element).

<rotate>
A rotation animation. Represents a RotateAnimation.
attributes:

android:fromDegrees
Float. Starting angular position, in degrees.
android:toDegrees
Float. Ending angular position, in degrees.
android:pivotX
Float or percentage. The X coordinate of the center of rotation. Expressed either: in pixels relative to the object's left edge (such as "5"), in percentage relative to the object's left edge (such as "5%"), or in percentage relative to the parent container's left edge (such as "5%p").
android:pivotY
Float or percentage. The Y coordinate of the center of rotation. Expressed either: in pixels relative to the object's top edge (such as "5"), in percentage relative to the object's top edge (such as "5%"), or in percentage relative to the parent container's top edge (such as "5%p").
For more attributes supported by <rotate>, see the Animation class reference (of which, all XML attributes are inherrited by this element).

### 例子:
1.xml代码
```
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:shareInterpolator="false">
    <scale
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:fromXScale="1.0"
        android:toXScale="1.4"
        android:fromYScale="1.0"
        android:toYScale="0.6"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fillAfter="false"
        android:duration="700" />
    <set
        android:interpolator="@android:anim/accelerate_interpolator"
        android:startOffset="700">
        <scale
            android:fromXScale="1.4"
            android:toXScale="0.0"
            android:fromYScale="0.6"
            android:toYScale="0.0"
            android:pivotX="50%"
            android:pivotY="50%"
            android:duration="400" />
        <rotate
            android:fromDegrees="0"
            android:toDegrees="-45"
            android:toYScale="0.0"
            android:pivotX="50%"
            android:pivotY="50%"
            android:duration="400" />
    </set>
</set>
```
2.播放动画
```
ImageView image = (ImageView) findViewById(R.id.image);
Animation hyperspaceJump = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
image.startAnimation(hyperspaceJump);
```

## Frame animation（帧动画）: 
一种定义在XML中的动画，顺序显示一组图片（像电影）

### 文件位置：
res/drawable/ 文件下，放在res/anim/会报错
如：res/drawable/filename.xml
### 语法：
```
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource_name"
        android:duration="integer" />
</animation-list>
```
### 元素:
/<animation-list>/
必需的，一定要放在根元素， 包含一个或多个 <item> 元素.
属性:
android:oneshot
Boolean. "true" 播放动画一次; "false" 循环播放动画.
<item>
动画的一帧，一定包含在 <animation-list> 元素下.
属性:
android:drawable
图片资源. 该帧动画对应得图片.
android:duration
Integer. 显示该帧动画时间，单位毫秒ms.
### 例子:
```
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item android:drawable="@drawable/rocket_thrust1" android:duration="200" />
    <item android:drawable="@drawable/rocket_thrust2" android:duration="200" />
    <item android:drawable="@drawable/rocket_thrust3" android:duration="200" />
</animation-list>
```
设置动画作为一个View的背景, 然后播放动画：
```
ImageView rocketImage = (ImageView) findViewById(R.id.rocket_image);
rocketImage.setBackgroundResource(R.drawable.rocket_thrust);

rocketAnimation = (AnimationDrawable) rocketImage.getBackground();
rocketAnimation.start();
```



