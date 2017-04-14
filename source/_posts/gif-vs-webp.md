---
title: Gif vs WebP
date: 2017-04-14 12:21:09
tags:
---
# WebP
[WebP](https://developers.google.cn/speed/webp/)，是一种支持有损压缩和无损压缩的图片文件格式，派生自图像编码格式 VP8。根据 Google 的测试，无损压缩后的 WebP 比 PNG 文件少了 45％ 的文件大小，即使这些 PNG 文件经过其他压缩工具压缩之后，WebP 还是可以减少 28％ 的文件大小。
# 性能测试
同时播放三张相同的动图.
### Gif样图
由video转gif,文件大小10.7MB
<img src="/album/pic_gif.gif" width = "45%" />
### WebP样图
使用[gif2webp](https://developers.google.cn/speed/webp/docs/gif2webp)无损转换,文件大小6.8MB
<img src="/album/pic_gif.gif" width = "45%" />
### GIF性能
1. 单张解码时间:210ms
2. 同时播放3张动画流畅性：卡顿
3. cpu与内存
![](/album/gif_test.png)

### WebP性能
1. 单张解码时间:140ms
2. 同时播放3张动画流畅性：流畅
3. cpu与内存
![](/album/webp_test.png)

### 结论
相同画质动画，webp在文件大小、内存、cpu要优于GIF

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