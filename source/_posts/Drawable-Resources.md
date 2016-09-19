---
title: Drawable Resources
date: 2016-08-04 12:24:42
category: android
tags: drawable
---
Drawable Resources
- Bitmap File
- .9.png
- Layer List
- selector
- Level List
- Clip Drawable
- Scale Drawable
- Shape Drawable
- animation-list
<!-- more -->
# Bitmap File
图片文件 (.png, .jpg, or .gif).创建一个BitmapDrawable.
# Nine-Patch File (.9.png)
NinePatch是一个定义了可伸展区域的PNG图像，被用在view大小超出图片大小。
# Layer List
一个Drawable管理者一组其他Drawable.相当于布局。 java->LayerDrawable.
```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
      <bitmap android:src="@drawable/android_red"
        android:gravity="center" />
    </item>
    <item android:top="10dp" android:left="10dp">
      <bitmap android:src="@drawable/android_green"
        android:gravity="center" />
    </item>
    <item android:top="20dp" android:left="20dp">
      <bitmap android:src="@drawable/android_blue"
        android:gravity="center" />
    </item>
</layer-list>
```
# State List(selector)
An XML file that references different bitmap graphics for different states (for example, to use a different image when a button is pressed). Creates a StateListDrawable.
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>


```
# Level List
An XML file that defines a drawable that manages a number of alternate Drawables, each assigned a maximum numerical value. Creates a LevelListDrawable.
```
<?xml version="1.0" encoding="utf-8"?>
<level-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/status_off"
        android:maxLevel="0" />
    <item
        android:drawable="@drawable/status_on"
        android:maxLevel="1" />
</level-list>
```
Once this is applied to a View, the level can be changed with setLevel() or setImageLevel().

# Transition Drawable
An XML file that defines a drawable that can cross-fade between two drawable resources. Creates a TransitionDrawable.
# Inset Drawable
An XML file that defines a drawable that insets another drawable by a specified distance. This is useful when a View needs a background drawble that is smaller than the View's actual bounds.
# Clip Drawable
An XML file that defines a drawable that clips another Drawable based on this Drawable's current level value. Creates a ClipDrawable.

剪下drawable一部分显示，progressBar进度条，

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="4.5dp" />
            <solid android:color="#252525" />
        </shape>
    </item>

    <item android:id="@android:id/progress">
        <clip>
            <layer-list>
            <item>
                <color android:color="#00000000" />
            </item>
            <item
                android:bottom="2.5dp"
                android:left="2.5dp"
                android:right="2.5dp"
                android:top="2.5dp">
                <shape>
                    <solid android:color="@color/orange_color_normal" />
                    <corners android:radius="2dp" />
                </shape>
            </item>

        </clip>
    </item>

</layer-list>
```
ps:由于没有达到UE的进度两边都是半圆要求，放弃了clip，使用scale实现
```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="4.5dp" />
            <solid android:color="#252525" />
        </shape>
    </item>

    <item android:id="@android:id/progress">
        <scale
            android:drawable="@drawable/publish_progress_primary"
            android:scaleWidth="100%" />
    </item>

</layer-list>
```
# Scale Drawable
An XML file that defines a drawable that changes the size of another Drawable based on its current level value. Creates a ScaleDrawable
# Shape Drawable
An XML file that defines a geometric shape, including colors and gradients. Creates a ShapeDrawable.
Also see the Animation Resource document for how to create an AnimationDrawable.

# animation-list
Note: A color resource can also be used as a drawable in XML. For example, when creating a state list drawable, you can reference a color resource for the android:drawable attribute (android:drawable="@color/green").

