---
title: 礼物连击数字
date: 2017-06-07 15:35:22
tags:
---
礼物连击效果，显示当前连击数，有缩放动画。
# 绘制数字
由于数字有描边，需要自定义控件，描边颜色不同于填充颜色
1. 画笔初始化
一个Paint为数字，另一个Paint为数字描边
```
        //stroke
        strokePaint = new Paint();
        strokePaint.setColor(strokeColor);
        strokePaint.setTextSize(textSize);
        strokePaint.setTypeface(Typeface.DEFAULT_BOLD);
        strokePaint.setStyle(Paint.Style.FILL_AND_STROKE);
        strokePaint.setStrokeWidth(strokeWidth);
        strokePaint.setAntiAlias(true);
        strokePaint.setAlpha(76);
        strokePaint.setStrokeJoin(Paint.Join.ROUND);

        //number
        textPaint = new Paint();
        textPaint.setColor(textColor);
        textPaint.setTextSize(textSize);
        textPaint.setTypeface(Typeface.DEFAULT_BOLD);
        textPaint.setAntiAlias(true);
```
2. 计算绘制区域
计算绘制区域宽高，开始x,y坐标。
3. 绘制
<!--more-->
```
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        if (TextUtils.isEmpty(numStr)) {
            return;
        }

        //x
        canvas.drawText(X, xx, xy, strokePaint);
        canvas.drawText(X, xx, xy, textPaint);

        //number
        canvas.drawText(numStr, numX, numY, strokePaint);
        canvas.drawText(numStr, numX, numY, textPaint);
    }
```
# 礼物连击动画
1. 大小从1.93->0.5倍缩小，透明度从0->1，耗时50%
2. 大小从0.3->1.25倍缩小，耗时30%
3. 大小从1.25->1倍缩小，耗时20%

- version 1.0
低端机快速连击卡顿，出现x,y轴不同步
```
    AnimatorSet dribbleFirstAnimSet = new AnimatorSet();
    Animator scaleAnim1X = ObjectAnimator.ofFloat(dribbleNumberView, "scaleX", 1.93f, 0.5f);
    Animator scaleAnim1Y = ObjectAnimator.ofFloat(dribbleNumberView, "scaleY", 1.93f, 0.5f);
    Animator alphaAnim1 = ObjectAnimator.ofFloat(dribbleNumberView, "alpha", 0.05f, 1f);
    dribbleFirstAnimSet.playTogether(scaleAnim1X,scaleAnim1Y,alphaAnim1);
    dribbleFirstAnimSet.setDuration(175);

    AnimatorSet dribbleSecondAnimSet = new AnimatorSet();
    Animator scaleAnim2X = ObjectAnimator.ofFloat(dribbleNumberView, "scaleX", 0.5f, 1.25f);
    Animator scaleAnim2Y = ObjectAnimator.ofFloat(dribbleNumberView, "scaleY", 0.5f, 1.25f);
    dribbleSecondAnimSet.playTogether(scaleAnim2X,scaleAnim2Y);
    dribbleSecondAnimSet.setDuration(105);

    AnimatorSet dribbleThirdAnimSet = new AnimatorSet();
    Animator scaleAnim3X = ObjectAnimator.ofFloat(dribbleNumberView, "scaleX", 1.25f, 1f);
    Animator scaleAnim3Y = ObjectAnimator.ofFloat(dribbleNumberView, "scaleY", 1.25f, 1f);
    dribbleThirdAnimSet.playTogether(scaleAnim3X,scaleAnim3Y);
    dribbleThirdAnimSet.setDuration(70);

    dribbleAnimSet =newAnimatorSet();
    dribbleAnimSet.playSequentially(dribbleFirstAnimSet,dribbleSecondAnimSet,dribbleThirdAnimSet);
```
<!--more-->
- version 2.0
无明显卡顿
```
    PropertyValuesHolder pvh1X = PropertyValuesHolder.ofFloat("scaleX", 1.93f, 0.5f);
    PropertyValuesHolder pvh1Y = PropertyValuesHolder.ofFloat("scaleY", 1.93f, 0.5f);
    PropertyValuesHolder pvhA = PropertyValuesHolder.ofFloat("alpha", 0.05f, 1f);
    Animator dribbleFirstAnim = ObjectAnimator.ofPropertyValuesHolder(dribbleNumberView, pvh1X, pvh1Y, pvhA);
    dribbleFirstAnim.setDuration(175);

    PropertyValuesHolder pvh2X = PropertyValuesHolder.ofFloat("scaleX", 0.5f, 1.25f);
    PropertyValuesHolder pvh2Y = PropertyValuesHolder.ofFloat("scaleY", 0.5f, 1.25f);
    Animator dribbleSecondAnim = ObjectAnimator.ofPropertyValuesHolder(dribbleNumberView, pvh2X, pvh2Y);
    dribbleSecondAnim.setDuration(105);

    PropertyValuesHolder pvh3X = PropertyValuesHolder.ofFloat("scaleX", 1.25f, 1f);
    PropertyValuesHolder pvh3Y = PropertyValuesHolder.ofFloat("scaleY", 1.25f, 1f);
    Animator dribbleThirdAnim = ObjectAnimator.ofPropertyValuesHolder(dribbleNumberView, pvh3X, pvh3Y);
    dribbleSecondAnim.setDuration(70);

    dribbleAnimSet =newAnimatorSet();
    dribbleAnimSet.playSequentially(dribbleFirstAnim,dribbleSecondAnim,dribbleThirdAnim);
```
- version 3.0
```
    Keyframe keyframeS1 = Keyframe.ofFloat(0, 1.93f);
    Keyframe keyframeS2 = Keyframe.ofFloat(0.5f, 0.5f);
    Keyframe keyframeS3 = Keyframe.ofFloat(0.8f, 1.25f);
    Keyframe keyframeS4 = Keyframe.ofFloat(1, 1f);
    PropertyValuesHolder pvhX = PropertyValuesHolder.ofKeyframe("scaleX", keyframeS1, keyframeS2, keyframeS3, keyframeS4);
    PropertyValuesHolder pvhY = PropertyValuesHolder.ofKeyframe("scaleY", keyframeS1, keyframeS2, keyframeS3, keyframeS4);

    Keyframe keyframeA1 = Keyframe.ofFloat(0, 0f);
    Keyframe keyframeA2 = Keyframe.ofFloat(0.5f, 1f);
    PropertyValuesHolder pvhA = PropertyValuesHolder.ofKeyframe("alpha", keyframeA1, keyframeA2);

    dribbleAnim =ObjectAnimator.ofPropertyValuesHolder(dribbleNumberView,pvhX,pvhY,pvhA);
    dribbleAnim.setDuration(350);
    dribbleAnim.addListener(newAnimatorListener() {
        @Override
        public void onAnimationEnd (Animator animation){
            Worker.postMain(new Runnable() {
                @Override
                public void run() {
                    clearGiftStatusFlags(Status.DRIBBLE);
                }
            });
        }
    });
```

** 遇到的问题 **
1. 缩放中心
最初设控件宽度自适应，缩放中心为控件中心，每次设置数字时requestLayout,性能非常差；解决方法，设置控件固定长，动态算缩放中心。
2. match_parent
当控制宽度固定100dp，发现连击1000以上连击绘制出界，然后修改为match_parent，父布局为RelativeLayout ，连击控件A在B控件右边，当B控件动画x轴移动，发现在大量Measure卡顿。