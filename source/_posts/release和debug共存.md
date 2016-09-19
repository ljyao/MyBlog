---
title: release和debug包共存
date: 2016-09-08 20:44:28
tags: tools
---
PackageName，所有的Android应用程序都有一个包名。包名是设备上的这个应用程序的唯一标识，所以应用共存从包名下手。
# 1.修改AndroidManifest中package
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.my.app"
    android:versionCode="1"
    android:versionName="1.0" >
```
大量的包、R.java的重命名，不推荐。
# 2. 配置应用的build.gradle文件
```
android{
        ...
        buildTypes{
            debug{
                //在编译打包时会给包名加上后缀
                applicationIdSuffix'.debug'
            }
            release{

            }
        }
        ...
    }
```
<!-- more -->
遇到的问题：
error:Install shows error in console: INSTALL FAILED CONFLICTING PROVIDER
修改AndroidManifest.xml
```
  <provider
      android:name=""
      android:authorities=""
      android:exported="false" />
```
