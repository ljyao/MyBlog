---
title: 视频预加载
date: 2016-08-08 16:11:58
tags: 性能优化
---
同时播放多个视频时，播放完上一个视频切换到下一个视频，需要MediaPlayer先释放上一个视频，然后加载下一个视频，这时候难免会有一会卡顿出现；我解决方案是：通过MediaPlayer渲染到SurfaceView去播放视频，使用2个MediaPlayer，当第一个视频准备好时，去预加载下一个视频，减少MediaPlayer释放和加载视频消耗的时间。
<!-- more -->

```
public class PreLoadVideoView extends RelativeLayout implements SurfaceHolder.Callback {
    @ViewById(R.id.video_surface)
    protected SurfaceView surface;
    private MediaPlayer mediaPlayer;
    private MediaPlayer nextMediaPlayer;
    private SurfaceHolder surfaceHolder;

    public PreLoadVideoView(Context context, AttributeSet attrs) {
        super(context, attrs);

        Worker.postWorker(new Runnable() {
            @Override
            public void run() {
                initMediaPlayer();
                Log.i("PreLoadVideoView", "postWorker->initMediaPlayer->end");
            }
        });
    }

    @AfterViews
    public void initView() {
        surfaceHolder = surface.getHolder();
        surfaceHolder.addCallback(this);
        surfaceHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }

    private void initMediaPlayer() {
        if (mediaPlayer == null) {
            synchronized (this) {
                if (mediaPlayer == null) {
                    try {
                        mediaPlayer = MediaPlayer.create(getContext(), R.raw.video_guide);
                        mediaPlayer.setOnInfoListener(new MediaPlayer.OnInfoListener() {
                            @Override
                            public boolean onInfo(MediaPlayer mp, int what, int extra) {
                                Log.i("PreLoadVideoView", "what" + what + "  extra" + extra);
                                return false;
                            }
                        });
                        mediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                            @Override
                            public void onPrepared(MediaPlayer mp) {
                                Log.i("PreLoadVideoView", "onPrepared");

                                //next
                                initNextMediaPlayer();
                            }
                        });
                        mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                            @Override
                            public void onCompletion(MediaPlayer mp) {
                                playNextVideo();
                            }
                        });

                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }

    }

    private void initNextMediaPlayer() {
        Worker.postWorker(new Runnable() {
            @Override
            public void run() {
                nextMediaPlayer = MediaPlayer.create(getContext(), R.raw.video_publish_guide_bg);
                nextMediaPlayer.setLooping(true);
            }
        });

    }


    private void playNextVideo() {
        if (nextMediaPlayer == null) {
            return;
        }

        Worker.postWorker(new Runnable() {
            @Override
            public void run() {
                mediaPlayer.release();
                mediaPlayer = nextMediaPlayer;
                nextMediaPlayer.setDisplay(surfaceHolder);
                nextMediaPlayer.start();
            }
        });
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        Log.i("PreLoadVideoView", "surfaceCreated");

        if (mediaPlayer == null) {
            initMediaPlayer();
        }

        mediaPlayer.setDisplay(surfaceHolder);
        mediaPlayer.start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {

    }

    public void release() {
        if (mediaPlayer != null) {
            if (mediaPlayer.isPlaying()) {
                mediaPlayer.stop();
            }
            mediaPlayer.release();
            mediaPlayer = null;
        }
        if (nextMediaPlayer != null) {
            if (nextMediaPlayer.isPlaying()) {
                nextMediaPlayer.stop();
            }
            nextMediaPlayer.release();
            nextMediaPlayer = null;
        }
    }
}
```
