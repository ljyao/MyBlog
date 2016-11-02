---
title: Activity全屏切换
date: 2016-10-21 17:13:06
tags:
---
# 1. Theme
```
 android:theme="@android:style/Theme.Holo.NoActionBar.Fullscreen" >
```
优点
- 容易记住,不容易犯错
- ui更加平滑，因为系统知道全屏信息

# 2. WindowManager flags
```
// FullScreen
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                        WindowManager.LayoutParams.FLAG_FULLSCREEN);
// Cancel
getWindow().setFlags(~WindowManager.LayoutParams.FLAG_FULLSCREEN,
                     WindowManager.LayoutParams.FLAG_FULLSCREEN);
```
**全屏切换时，布局会重新拉伸，造成卡顿，避免方法：**
- 通过FLAG_LAYOUT_IN_SCREEN flag使全屏时布局在状态栏以下。
- FLAG_LAYOUT_NO_LIMITS flag使全屏时布局全屏
**FLAG_LAYOUT_NO_LIMITS flag会导致系统函数getWindowVisibleDisplayFrame（主要用来计算键盘高度）失效，致命bug。**

**Flag bug避免与优化**
使用marginTop避免Resize卡顿:
不使用有FLAG_LAYOUT_NO_LIMITS flag，给全屏展示的rootview设置一个marginTop，为负的状态栏高，切换过程看起来就像拉伸至全屏，当全屏后去掉marginTop。

虽然解决掉不是有FLAG_LAYOUT_NO_LIMITS 的bug，但是状态栏收缩回去会卡白一下。
# 3. SetSystemUiVisibility()
**API 16以上**
```
View decorView = getWindow().getDecorView();
// Hide the status bar.
int uiOptions = View.SYSTEM_UI_FLAG_FULLSCREEN;
decorView.setSystemUiVisibility(uiOptions);
// Remember that you should never show the action bar if the
// status bar is hidden, so hide that too if necessary.
ActionBar actionBar = getActionBar();
actionBar.hide();
```
使用SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN使布局拉伸至全屏，使用SYSTEM_UI_FLAG_LAYOUT_STABLE 避免布局resize

### **Immersive**（退出全屏后，状态栏自动隐藏）
- SYSTEM_UI_FLAG_IMMERSIVE_STICKY **API 19**
```
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_FULLSCREEN);
    } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_FULLSCREEN);
    } else {
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);
    }
 ```
- 监听window,实现API19以下的Immersive。
```
 @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        //Immersive
        if (hasFocus && isFullScreen) {
            setFullScreen(isFullScreen);
        }
    }
```
缺点：弹出键盘返回桌面都自动退出全屏。

# 4. WindowManager和SetSystemUiVisibility混合使用（完美方案）
- 布局全屏，避免全屏切换闪烁。
```
    if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.JELLY_BEAN){
        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
    }
```
- 全屏切换设置
```
  //全屏
   getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);
   //退出全屏
   getWindow().clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
```
**官方文档**
[Managing the System UI](https://developer.android.com/training/system-ui/index.html)
[Hiding the Status Bar](https://developer.android.com/training/system-ui/status.html)