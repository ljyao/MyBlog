---
title: 属性动画实现进度条
date: 2017-05-27 19:10:45
tags:
---
实现一个圆形自定义进度条，进度条的颜色随着进度变化而变化，按进度下去后反向回去重置；为了解耦，主要有2个模块：1、自定义View，2、Controler
# 自定义view
提供进度、进度条颜色、背景颜色等设置接口，负责UI绘制。
```
  @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        final int width = getWidth();
        final int height = getHeight();

        float delta = Math.max(finishedStrokeWidth, unfinishedStrokeWidth) / 2;
        finishedOuterRect.set(delta,
                delta,
                width - delta,
                height - delta);
        unfinishedOuterRect.set(delta,
                delta,
                width - delta,
                height - delta);

        float innerCircleRadius = (width - 2 * Math.min(finishedStrokeWidth, unfinishedStrokeWidth) + Math.abs(finishedStrokeWidth - unfinishedStrokeWidth)) / 2f;
        canvas.drawCircle(width / 2.0f, height / 2.0f, innerCircleRadius, innerCirclePaint);

        int startingDegree = getStartingDegree();
        float progressAngle = getProgressAngle();

        if (clockwise) {
            canvas.drawArc(unfinishedOuterRect, startingDegree + progressAngle, 360 - progressAngle, false, unfinishedPaint);
            canvas.drawArc(finishedOuterRect, startingDegree, progressAngle, false, finishedPaint);
        } else {
            canvas.drawArc(unfinishedOuterRect, startingDegree, 360 - progressAngle, false, unfinishedPaint);
            canvas.drawArc(finishedOuterRect, startingDegree + progressAngle, progressAngle, false, finishedPaint);
        }

        if (!TextUtils.isEmpty(text)) {
            float textHeight = textPaint.descent() + textPaint.ascent();
            canvas.drawText(text, (getWidth() - textPaint.measureText(text)) / 2.0f, (getWidth() - textHeight) / 2.0f, textPaint);
        }
    }

```

# Controler
负责进度条、颜色等刷新

UI定时刷新方式有以下几种
1. Thread.sleep
在短视频拍摄中，视频的处理占用大量cpu资源，发现sleep时间相当不准，尤其在低端机。
查看Thread.sleep源码发现，在一个while循环中调用native sleep方法，然后比较睡眠时间，判断是否继续睡眠。
```
// Wait may return early, so loop until sleep duration passes.
        synchronized (lock) {
            while (true) {
                sleep(lock, millis, nanos);

                long now = System.nanoTime();
                long elapsed = now - start;

                if (elapsed >= duration) {
                    break;
                }

                duration -= elapsed;
                start = now;
                millis = duration / NANOS_PER_MILLI;
                nanos = (int) (duration % NANOS_PER_MILLI);
            }
        }
```
2. Timer
用定时任务去属性进度，但是子线程与主线程的切换也是一种开销。
3. Handler
通过不停发延时handler去刷新UI，但是当主线有大量绘制任务，handler周期时间变得不可控。
4. HandlerThread
通过子线程发送延时handler，与Timer相比，handler带来了很多便利，但是子线程与主线程的切换也是一种开销。
5. 属性动画（自定义TypeEvaluator）
通过自定义TypeEvaluator，使用ValueAnimator去控制自定义的属性
TypeEvaluator去计算最新的值
、、、
public class ProgressBarEvaluator implements TypeEvaluator<ProgressBarEvaluator.ProgressValues> {

    @Override
    public ProgressValues evaluate(float fraction, ProgressValues startValue, ProgressValues endValue) {
        ProgressValues values = new ProgressValues();

        //color
        int startInt = startValue.color;
        int startA = (startInt >> 24) & 0xff;
        int startR = (startInt >> 16) & 0xff;
        int startG = (startInt >> 8) & 0xff;
        int startB = startInt & 0xff;

        int endInt = endValue.color;
        int endA = (endInt >> 24) & 0xff;
        int endR = (endInt >> 16) & 0xff;
        int endG = (endInt >> 8) & 0xff;
        int endB = endInt & 0xff;

        values.color = ((startA + (int) (fraction * (endA - startA))) << 24)
                | ((startR + (int) (fraction * (endR - startR))) << 16)
                | ((startG + (int) (fraction * (endG - startG))) << 8)
                | ((startB + (int) (fraction * (endB - startB))));


        values.progress = startValue.progress + fraction * (endValue.progress - startValue.progress);

        values.scale = startValue.scale + fraction * (endValue.scale - startValue.scale);
        return values;
    }

    public static class ProgressValues {
        public float progress;
        public int color;
        public float scale;

        public ProgressValues() {
        }

        public ProgressValues(float progress, int color, float scale) {
            this.progress = progress;
            this.color = color;
            this.scale = scale;
        }
    }
}
、、、
在AnimatorUpdateListener回调去刷新UI
*** 注意setEvaluator要放在setObjectValues后 ***
```
 progressAnim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                ProgressBarEvaluator.ProgressValues values = (ProgressBarEvaluator.ProgressValues) animation.getAnimatedValue();
                progressbar.setScaleX(values.scale);
                progressbar.setScaleY(values.scale);

                progressbar.setProgress(values.progress);

                progressbar.setUnfinishedStrokeColor(values.color);
            }
        });
```
6. 属性动画
读完源码后，属性动画通过反射调用对象方法，驼峰命名，所以不需要自定义TypeEvaluator,注意方法名的混淆，加上注解 @Keep
```
        PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("scaleX", startScale, endScale);
        PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("scaleY", startScale, endScale);
        PropertyValuesHolder pvhP = PropertyValuesHolder.ofFloat("progress", startProgress, endProgress);
        PropertyValuesHolder pvhC = PropertyValuesHolder.ofInt("unfinishedStrokeColor", startColor, endColor);
        pvhC.setEvaluator(new ArgbEvaluator());
        progressAnim = ObjectAnimator.ofPropertyValuesHolder(progressbar, pvhX, pvhY, pvhP, pvhC);
```

