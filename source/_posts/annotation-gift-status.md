---
title: 礼物状态
date: 2017-06-07 14:44:04
tags:
---
状态通常有三种实现方式：常量、枚举、注解
- 常量：简单，便于位运算，占用内存少，但是携带信息少，可读性差。
- 枚举：便于封装，保证了类型安全，可读性高，占用内存大，dex变大
ps.Enums often require more than twice as much memory as static constants. You should strictly avoid using enums on Android.
- 注解：相对常量比较安全，编译期就可以排除错误类型
礼物控件用来播放礼物，从出现到消失共有8种状态（空闲、加载资源、进入中、播放礼物效果、连击动画、离开中、活跃的），多种状态是可以并行的，所以通过flag标记，一位表示一种状态，然后使用注解确保值的安全。
<!-- more -->
```
public class GiftDisplayStatus {
    private int mFlag = FREE;

    public void addFlag(@Status int flag) {
        mFlag |= flag;
    }

    public void clearFlag(@Status int flag) {
        mFlag &= ~flag;
    }

    public void reset() {
        mFlag = FREE;
    }

    public boolean isAvailable() {
        return (mFlag & ~ACTIVE) == FREE;
    }

    public boolean isFree() {
        return mFlag == FREE;
    }

    @IntDef(flag = true, value = {FREE, PREPARE, INTO, PLAY, DRIBBLE, LEAVE, ACTIVE})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Status {
        int FREE = 0x00000001;
        int PREPARE = 0x00000002;
        int INTO = 0x00000004;
        int PLAY = 0x00000008;
        int DRIBBLE = 0x00000010;
        int LEAVE = 0x00000020;
        int ACTIVE = 0x00000040;
    }
}
```