---
title: 子线程绘制UI
date: 2017-07-18 15:11:11
tags: android
---
   在直播间点赞动画，通过在父控件addView，然后属性动画改变view位置、大小以透
明度，从而实现点赞动画，当大量点赞时，addview引起重新布局、属性动画都十分耗时，造成卡顿。从未使用子线程绘制，避免主线程卡顿。优化方式主要有三种：View绘制，TextureView，SurfaceView，然后定时刷新帧，计算当前帧赞的位置、大小、透明度。
# View绘制
重写ondraw，定时postInvalidateDelayed刷新UI,然后遍历绘制当前帧每一个赞。
1. 优点
兼容性较好
2. 缺点
绘制比较耗时

# SurfaceView
surfaceView定时刷新赞动画，和view绘制最大区别是可以放在子线程绘制
## surfaceView生命周期
- surfaceCreated(SurfaceHolder holder) 
surface可用，创建绘制线程，设置view微透明。
getHolder().setFormat(PixelFormat.TRANSPARENT);

- surfaceDestroyed(SurfaceHolder holder)
surface销毁 释放资源，终止绘制线程。

## 绘制
先等待surface可用，通过getHolder().lockCanvas()方法获取当前canvas，遍历绘制当前帧，最后getHolder().unlockCanvasAndPost(canvas)提交canvas刷新UI
```
        startTime = SystemClock.uptimeMillis();
        renderLock.lock();
        if (mHasSurface) {
            if (getHolder().getSurface().isValid()) {
                canvas = getHolder().lockCanvas();

                if (canvas != null) {
                    clearCanvas(canvas);
                    drawLikes(canvas);
                    getHolder().unlockCanvasAndPost(canvas);
                }
            }
        }
```
<!--more-->
# TextureView
## TextureView生命周期
- onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height)
SurfaceTexture可用，启动渲染线程
setOpaque 设置微透明
- onSurfaceTextureDestroyed(SurfaceTexture surface)
销毁资源
如果 returns true, 这个这个方法后 surface texture不再渲染， 如果 returns false, 客户端需要主动调用 SurfaceTexture#release()释放资源

## 绘制
获取canvas lockCanvas(dirty)，遍历绘制，提交canvas unlockCanvasAndPost，但是在onSurfaceTextureDestroyed后，canvas不可以继续使用，不然native会抛出异常，对onSurfaceTextureDestroyed draw方法加锁后解决问题，但是会阻塞主线程；最后使用surface.lockCanvas，上述问题surface会在java层抛出异常。
```
       surface = new Surface(mSurfaceTexture);
        Canvas canvas = surface.lockCanvas(dirty);
        if (canvas == null) {
            Log.d(TAG, "lockCanvas() failed");
            return;
        }
        try {
            clearCanvas(canvas);
            drawLikes(canvas);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            surface.unlockCanvasAndPost(canvas);
        }
```
# 总结
在层次、性能方面TextureView有绝对的优势，View兼容低端机是较好的方案。