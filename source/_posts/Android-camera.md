---
title: Android Camera
date: 2016-09-23 11:32:24
tags: camera
---
# 使用camera拍照基本流程
1. Obtain an instance of Camera from open(int).
获取相机实例，open指定前后摄像头。
2. Get existing (default) settings with getParameters().
通过getParameters获取相机默认设置参数
3. If necessary, modify the returned Camera.Parameters object and call setParameters(Camera.Parameters). 
setParameters设置相机参数
4. Call setDisplayOrientation(int) to ensure correct orientation of preview.
setDisplayOrientation设置预览图像旋转角度
5. Important: Pass a fully initialized SurfaceHolder to setPreviewDisplay(SurfaceHolder). Without a surface, the camera will be unable to start the preview.
显示预览到surfaceview，没有surface相机是不可以预览。
6. Important: Call startPreview() to start updating the preview surface. Preview must be started before you can take a picture.
startPreview开始预览，拍照前一定要预览。
7. When you want, call takePicture(Camera.ShutterCallback, Camera.PictureCallback, Camera.PictureCallback, Camera.PictureCallback) to capture a photo. Wait for the callbacks to provide the actual image data.
takePicture拍照
8. After taking a picture, preview display will have stopped. To take more photos, call startPreview() again first.
拍照后预览将会停止，重拍一定要调用startPreview。
9. Call stopPreview() to stop updating the preview surface.
调用stopPreview停止预览
10. Important: Call release() to release the camera for use by other applications. Applications should release the camera immediately in onPause() (and re-open() it in onResume()).
调用release释放相机。

# 使用camera录视频基本流程
1. Obtain and initialize a Camera and start preview as described above.
2. Call unlock() to allow the media process to access the camera.
3. Pass the camera to setCamera(Camera). See MediaRecorder information about video recording.
4. When finished recording, call reconnect() to re-acquire and re-lock the camera.
5. If desired, restart preview and take more photos or videos.
6. Call stopPreview() and release() as described above.


# 踩坑记
## camera orientation
CameraInfo.orientation 是设备和相机之间的角度

1. setDisplayOrientation(int) 设置预览图像旋转角度
只对预览起作用（详见第5条），拍照的图片需要自己旋转。
首先获取相机的固定Rotation，代表需要旋转角度，后置摄像头是顺时针，前置摄像头是逆时针。
```
private int getPhotoRotation() {
        if (mCamera != null) {
            int rotation;
            int orientation = 0;
            Camera.CameraInfo info = new Camera.CameraInfo();
            Camera.getCameraInfo(mCameraID, info);

            if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
                rotation = (info.orientation - orientation + 360) % 360;
            } else {
                rotation = (info.orientation + orientation) % 360;
            }

            return rotation;
        } else {
            return 90;
        }
    }
```
2. setRotation 设置拍照图片旋转角度
存在兼容性问题，建议对bitmap进行旋转。
```
Matrix matrix=new Matrix();
matrix.reset();
matrix.postRotate(degrees);
dstBmp=Bitmap.createBitmap(srcBmp, 0, 0, srcBmp.getWidth(), srcBmp.getHeight(), matrix, true);
```
3. setPreviewSize设置预览图像分辨率，setPictureSize拍照分辨率
设置的分辨率不支持会抛异常
```
//获取预览支持的分辨率
cameraParameters.getSupportedPreviewSizes()
//获取拍照支持的分辨率 
cameraParameters.getSupportedPictureSizes() 
```
## 曝光过高
预览帧数过低会导致部分机型曝光过高，解决方案是录视频时降低帧率。

