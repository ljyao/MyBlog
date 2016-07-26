---
title: “unstartactivity”
date: 2016-01-27 20:21:41
tags:
---
    今天遇到一个奇怪现象，startActivity没反应，没有任何log，突然想到重写了activity的startActivityForResult方式，而且还把super删了，
纪念傻傻的我。。。。。。。
```
 @Override
    public void startActivityForResult(Intent intent, int requestCode, Bundle options) {
        super.startActivityForResult(intent, requestCode, options);
    }
```
