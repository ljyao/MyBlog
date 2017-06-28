---
title: 直播礼物动画
date: 2017-04-14 12:21:09
tags:
---
最近在做直播礼物，需要展示礼物动画，调研的方案有 帧动画，gif，apng，视频，webp，多次试验权衡后，确定最佳方案使用webp动画。
<!--[移动端图片格式调研](http://blog.ibireme.com/2015/11/02/mobile_image_benchmark/)-->

# PNG帧动画
## png简介
诞生在 1995 年，比 JPEG 晚几年。它本身的设计目的是替代 GIF 格式，所以它与 GIF 有更多相似的地方。PNG 只支持无损压缩，所以它的压缩比是有上限的。相对于 JPEG 和 GIF 来说，它最大的优势在于支持完整的透明通道。
## 帧动画实现
通过系统的AnimationDrawable类播放帧动画，用BitmapDrawable加载bitmap作为一帧，播放结束后主动recycle，释放bitmap。

主要代码
```
    private void initAnim(final Context context) {
        if (TextUtils.isEmpty(dirPath)) {
            return;
        }
        try {
            AssetManager assetManager = context.getAssets();
            String[] framePaths = assetManager.list(dirPath);
            int frameDuration = duration != 0 ? duration / framePaths.length : FRAME_DURATION;
            for (String frame : framePaths) {
                String framePath = dirPath + "/" + frame;
                Bitmap frameBmp = BitmapFactory.decodeStream(assetManager.open(framePath));
                addFrame(new BitmapDrawable(context.getResources(), frameBmp), frameDuration);
            }
            addFrame(getFrame(0), 0);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private void destroyBitmap() {
        try {
            for (int i = 0; i < getNumberOfFrames(); i++) {
                Drawable frame = getFrame(i);
                if (frame instanceof BitmapDrawable) {
                    Bitmap bmp = ((BitmapDrawable) frame).getBitmap();
                    if (bmp != null && !bmp.isRecycled()) {
                        bmp.recycle();
                    }
                }
                frame.setCallback(null);
            }
            setCallback(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
* 遇到的问题
1. 性能问题：内存开销大，播放一个全屏动画（10帧，750p），内存涨了40MB,gc频繁
2. 释放问题：为了避免OOM，动画执行完后，主动释放bitmap，调用stop后动画仍在绘制，导致抛异常，加回调后，部分机型仍会抛异常。
3. 稳定性：由于自己封装，边界问题考虑不是很好，容易卡死。
4. 文件太大
<!-- more -->

# Gif
诞生于 1987 年，随着初代互联网流行开来。它有很多缺点，比如通常情况下只支持 256 种颜色、透明通道只有 1 bit、文件压缩比不高。它唯一的优势就是支持多帧动画，凭借这个特性，它得以从 Windows 1.0 时代流行至今，而且仍然大受欢迎。

* 优点 ：
1. 制作简单
2. 文件小
* 缺点
1. 锯齿，边界模糊

# 视频
主要问题是，背景不可以透明，播放器开销大，稳定性

# APNG
是 Mozilla 在 2008 年发布的一种图片格式，旨在替换掉画质低劣的 GIF 动画。它实际上只是相当于 PNG 格式的一个扩展，所以 Mozilla 一直想把它合并到 PNG 标准里面去。然而 PNG 开发组并没有接受 APNG 这个扩展，而是一直在推进它自己的 MNG 动图格式。MNG 格式过于复杂以至于并没有什么系统或浏览器支持，而 APNG 格式由于简单容易实现，目前已经渐渐流行开来。Mozilla 自己的 Firefox 首先支持了 APNG，随后苹果的 Safari 也开始有了支持， Chrome 目前也已经尝试开始支持 ，可以说未来前景很好。

没有找到很好的播放库
# WebP
[WebP](https://developers.google.cn/speed/webp/)是 Google 在 2010 年发布的图片格式，希望以更高的压缩比替代 JPEG。它用 VP8 视频帧内编码作为其算法基础，取得了不错的压缩效果。它支持有损和无损压缩、支持完整的透明通道、也支持多帧动画，并且没有版权问题，是一种非常理想的图片格式。借由 Google 在网络世界的影响力，WebP 在几年的时间内已经得到了广泛的应用。看看你手机里的 App：微博、微信、QQ、淘宝、网易新闻等等，每个 App 里都有 WebP 的身影。Facebook 则更进一步，用 WebP 来显示聊天界面的贴纸动画。

- 制作webp动画
1. 在官网下载转换工作
2. 把png转为webp
转换命令，q为质量
```
cwebp -q 50 -lossless picture.png -o picture_lossless.webp
```
3. 把webp系列图转为webp动画
转换命令，+b会把所有帧重叠，-b不重叠
```
webpmux -frame 1.webp +100 -frame 2.webp +100+50+50 -frame 3.webp +100+50+50+1+b -loop 10 -bgcolor 255,255,255,255 -o anim_container.webp
```

目前只发现了通过命令转换，很不方便，改一个参数就需要重新输命令，于是请同学写了一个Python脚本，在此感谢@黎潇大神。
Python代码

```
# -*- coding: utf-8 -*-
import os
import sys
import re

rootdir = sys.argv[1]

toolDir = '/Users/shine/Documents/libwebp-0.4.1-mac-10.8/bin'
os.system('cd '+toolDir)

for subdir, dirs, files in os.walk(rootdir):
    if subdir.startswith('.') or len(files) == 0 :
        continue
    webps = []
    sortTmp = {}
    
    for fname in files:
        fpath = os.path.join(subdir, fname)
        l = fname.split('.')
        l[-1] = 'webp'
        opath = os.path.join(subdir, '.'.join(l))
        if (not fname.endswith('png')) or fname.startswith('.') :
            continue

        os.system('cwebp %s -q 100 -lossless -o %s' % (fpath, opath) )
        webps.append(opath)

        match = re.search(r'([0-9]+)', fname)
        num = int(match.group(0))
        sortTmp[num] = opath

    s = sortTmp.keys()
    s.sort()

    opts = ''
    opath = os.path.join(rootdir, subdir+'.webp')
       
    for row in s:
        fname = sortTmp[row]
        opts += ' -frame ' + fname +  ' +45+0+0+0-b'
    os.system('webpmux %s -loop 1 -o %s' % (opts, opath))

    print 'webps: ', s
```

# 使用fresco加载GIF和webp
## gradle添加依赖
```
    compile 'com.facebook.fresco:fresco:1.2.0'

    //gif 支持
    compile 'com.facebook.fresco:animated-gif:1.2.0'

    //webp动画支持
    compile 'com.facebook.fresco:animated-webp:1.2.0'
```
## Java
1. 自动播放
```
Uri uri;
DraweeController controller = Fresco.newDraweeControllerBuilder()
    .setUri(uri)
    .setAutoPlayAnimations(true)
    . // 其他设置（如果有的话）
    .build();
mSimpleDraweeView.setController(controller);
```
2. 手动播放
```
ControllerListener controllerListener = new BaseControllerListener<ImageInfo>() {
    @Override
    public void onFinalImageSet(
        String id,
        @Nullable ImageInfo imageInfo,
        @Nullable Animatable anim) {
        if (anim != null) {
          // 其他控制逻辑
          anim.start();
        }
    }
};

Uri uri;
DraweeController controller = Fresco.newDraweeControllerBuilder()
    .setUri(uri)
    .setControllerListener(controllerListener)
    // 其他设置（如果有的话）
    .build();
mSimpleDraweeView.setController(controller);
```