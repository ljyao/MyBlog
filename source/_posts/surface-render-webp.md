---
title: webp动画渲染
date: 2017-10-17 10:31:41
tags: 直播礼物
---
&emsp;&emsp;直播礼物上线以来，礼物越来越复杂，由最开始的静态图、小动画、全屏动画，动画越来越复杂，帧数也越来越多，文件也越来越大；从开始的效果放在apk到现在服务端可配；为了不影响用户体验，觉定开始优化直播礼物效果，优化先要找出问题所在；当动画是全屏动画，绘制和解码都耗费大量cpu，大文件的全屏动画绘制直接导致主线程卡顿，而且绘制cpu的开销增加导致解码更慢，在低端机都无法观看。

- 优化方向
1. 优化动画文件，加快解码速度。
2. 优化解码，使用bitmap缓存池，减少GC。
3. 使用GPU绘制。

# 使用fresco渲染
&emsp;&emsp;由于项目一直使用[fresco](www.fresco-cn.org)加载图片，礼物上线后也使用fresco加载（网络下载缓存、解码显示），直播礼物动画把序列帧压成webp文件。
&emsp;&emsp;查看fresco源码后，fresco在绘制当前帧时，再去预加载下一帧，当帧间隔时间较短或者解码较慢，预加载缓存就无效，导致了跳帧。
<!-- more -->
# fresco 自定义View
加快绘制，使用textureview canvas，在子线程渲染；实现fresco DraweeHolder显示view，fresco负责解码和每一帧回调；
- 设置uri加载webp，并获取Drawable
```
  DraweeHolder mDraweeHolder;
  .........
  ControllerListener<ImageInfo> controllerListener = new BaseControllerListener<ImageInfo>() {
            @Override
            public void onFinalImageSet(String id, ImageInfo imageInfo, final Animatable animatable) {
                if (animatable != null) {
                    if (animatable instanceof AnimatedDrawable2) {
                       ....
                    }
                }
            }

           ....
        };
    DraweeController mDraweeController = Fresco.newDraweeControllerBuilder()
            .setAutoPlayAnimations(false)
            .setOldController(getController())
            .setControllerListener(controllerListener)
            .setUri(uri)//设置uri
            .build();
    mDraweeHolder.setController(mDraweeController);
```
- 实现 AnimatedDrawable2回调
```
       mAnimatable.setCallback(new Drawable.Callback() {
            @Override
            public void invalidateDrawable(@NonNull Drawable who) {
                onDraw(who);
            }

           ....
        });
```
在invalidateDrawable回调中实现绘制
- 实现draw方法
在子线程中绘制
```
 public void onDraw(Drawable who){
        if (getSurfaceTexture() != null ) {
            if (surface == null) {
                surface = new Surface(getSurfaceTexture());
            }
            Canvas canvas = surface.lockCanvas(dirty);
            canvas.drawColor(Color.TRANSPARENT, android.graphics.PorterDuff.Mode.CLEAR);
            who.draw(canvas);
            surface.unlockCanvasAndPost(canvas);
        }
    }
```
## 总结
- 优点：实现简单，兼容好（fresco所以特性都支持，如缩放），降低了主线程耗时
- 缺点：canvas  cpu绘制慢，低端机100-200ms，fresc解码缺陷仍在

# TextureView surface
使用GPU绘制在surface上，自己实现解码队列。
## 解码
webp文件使用fresco解码
1. 加载webp文件，获取WebPImage
```
 public void setUri(final Uri uri, final WebpDecodeListener webpDecodeListener) {
        isActive = true;

        ImageRequest imageRequest = ImageRequestBuilder.newBuilderWithSource(uri).build();
        DataSource<CloseableReference<CloseableImage>> dataSource =
                Fresco.getImagePipeline().fetchDecodedImage(imageRequest, null);
        dataSource.subscribe(new BaseDataSubscriber<CloseableReference<CloseableImage>>() {
            @Override
            protected void onNewResultImpl(final DataSource<CloseableReference<CloseableImage>> dataSource) {
                    .....
            }
            ....
        }, CallerThreadExecutor.getInstance());
    }
```
2. 解码队列，缓存池
- 缓存池，使用Pools.SynchronizedPool，v4包，线程安全
- 解码队列,单独一个线程负创建帧（空帧）放入LinkedBlockingQueue，当队列容量大于缓存最大值，线程阻塞；并用线程池去实际解码.
- 使用AtomicInteger计数，缓存池放满后，回调准备好了

生产队列，取消isActive置为false，并中断。
```
    private void createFrames(final WebPImage webPImage, final WebpDecodeListener webpDecodeListener) throws Exception {
       .......

        try {
            .....
            final WebpImageInfo imageInfo = new WebpImageInfo();
            ......

            for (int i = 0; i < frameCount && isActive; i++) {
                FrameInfo frameInfo = frameBitmapPool.acquire();
                if (frameInfo == null) {
                    Bitmap frameBitmap = Bitmap.createBitmap(imageWidth, imageHeight, Bitmap.Config.ARGB_8888);
                    frameInfo = new FrameInfo(frameBitmap);
                } else {
                    frameInfo.setValid(false);
                }

                if (i > mRenderFactor) {
                    final WebPFrame webPFrame = webPImage.getFrame(i);
                    .....

                    renderFrame(frameInfo, imageInfo, webpDecodeListener);
                } else {
                    //decode skip frame
                    Log.v(TAG, "decode skip frame index: " + i);
                }
                frameQueue.put(frameInfo);
            }
        } finally {
            webPImage.dispose();
        }
    }
```

线程池解码,调用webPFrame.dispose() 会中断渲染并抛出异常。
```
private void renderFrame(final FrameInfo frameInfo, final WebpImageInfo imageInfo, final WebpDecodeListener webpDecodeListener) {
        Worker.postWorker(new Runnable() {
            @Override
            public void run() {
                if (!isActive) {
                    return;
                }
                ....
                frameInfo.render();
                ....
            }
        });
    }
```
## 渲染
渲染线程使用HandlerThread

消息有：
1. MSG_INIT_SURFACE 
textureview onSurfaceTextureAvailable回调，SurfaceTexture可用，创建surface，gpu环境。
2. MSG_START
开始动画
3. MSG_STOP
停止动画
4. MSG_DESTROY
释放资源
5. MSG_DRAW
每一帧绘制
6. MSG_SET_URI
设置uri，开始解码
7. MSG_INIT_IMAGE
缓存区准备好了
8.  MSG_SET_LISTENER
设置动画回调

绘制方法
```
 private void drawFrame() {
        try {
            //按时间计算当前帧
            int index = (int) ((SystemClock.uptimeMillis() - mStartTimeMs) / imageInfo.getFrameInterval());
            //超时，动画结束
            if (index >= imageInfo.getFrameCount()) {
                sgpuImageEngine.drawByBitmap(EMPTY_BITMAP, 0, 0, mWidth, mHeight);

                stopInternal();
                onWebpAnimationEnd();
                return;
            }
            //跳帧
            while (index > mFrameFactor) {
                FrameInfo frame = webpDecodeHelper.takeFrame(index);
                webpDecodeHelper.releaseFrame(frame);

                mFrameFactor++;
                onAnimationFrame(mFrameFactor);
                Log.v(TAG, "draw skip frame factor: " + mFrameFactor);
            }

            //listener
            onAnimationFrame(mFrameFactor);

            //取出当前帧
            FrameInfo frame = webpDecodeHelper.takeFrame(mFrameFactor);
            //判断是否渲染完成
            if (frame.isValid()) {
                long startDrawMillis = System.currentTimeMillis();
                Rect dstRect = frame.getRenderRect(scaleX, scaleY);
                sgpuImageEngine.drawByBitmap(frame.getBitmap(), dstRect.left, dstRect.top, dstRect.width(), dstRect.height());
                Log.v(TAG, "factor: " + mFrameFactor + " draw time: " + String.valueOf(System.currentTimeMillis() - startDrawMillis));
            } else {
                //skip frame
                Log.v(TAG, "draw skip frame factor: " + mFrameFactor);
            }
            //用完的帧放回缓存池
            webpDecodeHelper.releaseFrame(frame);

            mFrameFactor++;
        } catch (Exception e) {
            e.printStackTrace();

            stopInternal();
            onWebpAnimationEnd();
        }
    }
```