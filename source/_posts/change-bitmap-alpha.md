---
title: 修改bitmap透明度
date: 2017-12-15 16:39:22
tags: android
---
修改bitmap透明度，实现视频贴纸透明度动画，实现方法：
1. canvas 修改paint的alpha，然后绘制在一个新的bitmap。
2. 修改图片的A通道。

# Canvas
```
 public Bitmap setBitmapAlpha(Bitmap bitmap, int alpha) {
        Bitmap newBitmap = Bitmap.createBitmap(bitmap.getWidth(), bitmap.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(newBitmap);
        Paint paint = new Paint();
        paint.setAlpha(alpha);
        //绘制到新的bitmap
        canvas.drawBitmap(bitmap, 0, 0, paint);
        bitmap.recycle();
        return newBitmap;
    }
```
<!-- more -->
# 修改A通道
bitmap为ARGB_8888，native修改，native中格式为RGBA
```

void JniBitmapOperator_jniBitmapAlpha(JNIEnv *env, jobject obj,
                                      jobject bitmap, jint alpha) {
    int ret;
    AndroidBitmapInfo bitmapInfo;
    //获取图片信息
    if ((ret = AndroidBitmap_getInfo(env, bitmap, &bitmapInfo)) < 0) {
        return;
    }
    int count = bitmapInfo.width * bitmapInfo.height * 4;
    char *bitmapPixels;
    //获取图像数组
    if ((ret = AndroidBitmap_lockPixels(env, bitmap, (void **) &bitmapPixels)) < 0) {
        return;
    }
    int i = 3;
    bitmapPixels += 3;
    while (i < count) {
        if (*bitmapPixels > alpha) {
            *bitmapPixels = alpha;
        }
        bitmapPixels += 4;
        i += 4;
    }
    AndroidBitmap_unlockPixels(env, bitmap);
}
```