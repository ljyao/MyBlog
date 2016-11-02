---
title: 无线调试
date: 2016-10-31 19:25:29
tags:
---
# 有root
手机安装adb wifi，root授权。
# 无root
## 先用数据线连接电脑，打开命令行输入
```
adb tcpip 5555 
```
## 确保手机连接的wifi和电脑是同一个局域网
```
adb connect *.*.*.*
```