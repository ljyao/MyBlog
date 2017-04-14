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
```
  private void startCamera(SurfaceTexture surface) {
        surface.setDefaultBufferSize(1080, 1920);

        Log.d("lll", "open camera begin");
         camera = Camera.open(Camera.CameraInfo.CAMERA_FACING_BACK);
        Camera.Parameters params = camera.getParameters();
        params.setPreviewSize(1920, 1080);
        params.setPreviewFormat(ImageFormat.YV12);
        camera.setParameters(params);

        camera.setDisplayOrientation(90);
        try {
            camera.setPreviewTexture(surface);
            camera.startPreview();
        } catch (IOException e) {
            e.printStackTrace();
        }
        final boolean[] flag = {true};
        byte[] buffer = new byte[getYuvBuffer(1920, 1080)];
        for (int i = 0; i < 2; i++) {
            camera.addCallbackBuffer(buffer);
        }
        camera.setPreviewCallbackWithBuffer(new Camera.PreviewCallback() {
            @Override
            public void onPreviewFrame(byte[] data, Camera camera) {
                if (flag[0]) {
                    flag[0] = false;
                    Log.d("lll", "open camera end");
                }
                camera.addCallbackBuffer(data);
            }
        });
    }

    public static int getYuvBuffer(int width, int height) {
        // stride = ALIGN(width, 16)
        int stride = (int) Math.ceil(width / 16.0) * 16;
        // y_size = stride * height
        int y_size = stride * height;
        // c_stride = ALIGN(stride/2, 16)
        int c_stride = (int) Math.ceil(width / 32.0) * 16;
        // c_size = c_stride * height/2
        int c_size = c_stride * height / 2;
        // size = y_size + c_size * 2
        return y_size + c_size * 2;
    }
```


# 使用camera录视频基本流程
Open Camera - Use the Camera.open() to get an instance of the camera object.
Connect Preview - Prepare a live camera image preview by connecting a SurfaceView to the camera using Camera.setPreviewDisplay().
Start Preview - Call Camera.startPreview() to begin displaying the live camera images.
Start Recording Video - The following steps must be completed in order to successfully record video:
Unlock the Camera - Unlock the camera for use by MediaRecorder by calling Camera.unlock().
Configure MediaRecorder - Call in the following MediaRecorder methods in this order. For more information, see the MediaRecorder reference documentation.
setCamera() - Set the camera to be used for video capture, use your application's current instance of Camera.
setAudioSource() - Set the audio source, use MediaRecorder.AudioSource.CAMCORDER.
setVideoSource() - Set the video source, use MediaRecorder.VideoSource.CAMERA.
Set the video output format and encoding. For Android 2.2 (API Level 8) and higher, use the MediaRecorder.setProfile method, and get a profile instance using CamcorderProfile.get(). For versions of Android prior to 2.2, you must set the video output format and encoding parameters:
setOutputFormat() - Set the output format, specify the default setting or MediaRecorder.OutputFormat.MPEG_4.
setAudioEncoder() - Set the sound encoding type, specify the default setting or MediaRecorder.AudioEncoder.AMR_NB.
setVideoEncoder() - Set the video encoding type, specify the default setting or MediaRecorder.VideoEncoder.MPEG_4_SP.
setOutputFile() - Set the output file, use getOutputMediaFile(MEDIA_TYPE_VIDEO).toString() from the example method in the Saving Media Files section.
setPreviewDisplay() - Specify the SurfaceView preview layout element for your application. Use the same object you specified for Connect Preview.
Caution: You must call these MediaRecorder configuration methods in this order, otherwise your application will encounter errors and the recording will fail.

Prepare MediaRecorder - Prepare the MediaRecorder with provided configuration settings by calling MediaRecorder.prepare().
Start MediaRecorder - Start recording video by calling MediaRecorder.start().
Stop Recording Video - Call the following methods in order, to successfully complete a video recording:
Stop MediaRecorder - Stop recording video by calling MediaRecorder.stop().
Reset MediaRecorder - Optionally, remove the configuration settings from the recorder by calling MediaRecorder.reset().
Release MediaRecorder - Release the MediaRecorder by calling MediaRecorder.release().
Lock the Camera - Lock the camera so that future MediaRecorder sessions can use it by calling Camera.lock(). Starting with Android 4.0 (API level 14), this call is not required unless the MediaRecorder.prepare() call fails.
Stop the Preview - When your activity has finished using the camera, stop the preview using Camera.stopPreview().
Release Camera - Release the camera so that other applications can use it by calling Camera.release().
Note: It is possible to use MediaRecorder without creating a camera preview first and skip the first few steps of this process. However, since users typically prefer to see a preview before starting a recording, that process is not discussed here.

Tip: If your application is typically used for recording video, set setRecordingHint(boolean) to true prior to starting your preview. This setting can help reduce the time it takes to start recording.
/**
private boolean prepareVideoRecorder(){

    mCamera = getCameraInstance();
    mMediaRecorder = new MediaRecorder();

    // Step 1: Unlock and set camera to MediaRecorder
    mCamera.unlock();
    mMediaRecorder.setCamera(mCamera);

    // Step 2: Set sources
    mMediaRecorder.setAudioSource(MediaRecorder.AudioSource.CAMCORDER);
    mMediaRecorder.setVideoSource(MediaRecorder.VideoSource.CAMERA);

    // Step 3: Set a CamcorderProfile (requires API Level 8 or higher)
    mMediaRecorder.setProfile(CamcorderProfile.get(CamcorderProfile.QUALITY_HIGH));

    // Step 4: Set output file
    mMediaRecorder.setOutputFile(getOutputMediaFile(MEDIA_TYPE_VIDEO).toString());

    // Step 5: Set the preview output
    mMediaRecorder.setPreviewDisplay(mPreview.getHolder().getSurface());

    // Step 6: Prepare configured MediaRecorder
    try {
        mMediaRecorder.prepare();
    } catch (IllegalStateException e) {
        Log.d(TAG, "IllegalStateException preparing MediaRecorder: " + e.getMessage());
        releaseMediaRecorder();
        return false;
    } catch (IOException e) {
        Log.d(TAG, "IOException preparing MediaRecorder: " + e.getMessage());
        releaseMediaRecorder();
        return false;
    }
    return true;
}
**/


# 开发笔记
## camera orientation
CameraInfo.orientation 是设备和相机之间的角度

1. setDisplayOrientation(int) 设置预览图像旋转角度
只对预览起作用（详见第5条）
```
    public static void setCameraDisplayOrientation(Activity activity,
                                                   int cameraId, android.hardware.Camera camera) {
        android.hardware.Camera.CameraInfo info =
                new android.hardware.Camera.CameraInfo();
        android.hardware.Camera.getCameraInfo(cameraId, info);
        int rotation = activity.getWindowManager().getDefaultDisplay()
                .getRotation();
        int degrees = 0;
        switch (rotation) {
            case Surface.ROTATION_0:
                degrees = 0;
                break;
            case Surface.ROTATION_90:
                degrees = 90;
                break;
            case Surface.ROTATION_180:
                degrees = 180;
                break;
            case Surface.ROTATION_270:
                degrees = 270;
                break;
        }

        int result;
        if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
            result = (info.orientation + degrees) % 360;
            result = (360 - result) % 360;  // compensate the mirror
        } else {  // back-facing
            result = (info.orientation - degrees + 360) % 360;
        }
        camera.setDisplayOrientation(result);
    }
```
2. setRotation 设置拍照图片旋转角度

通过设置相机setRotation旋转照片
```
 /**
     * Called when the orientation of the device has changed.
     * orientation parameter is in degrees, ranging from 0 to 359.
     * orientation is 0 degrees when the device is oriented in its natural position,
     * 90 degrees when its left side is at the top, 180 degrees when it is upside down, 
     * and 270 degrees when its right side is to the top.
     * {@link #ORIENTATION_UNKNOWN} is returned when the device is close to flat
     * and the orientation cannot be determined.
     *
     * @param orientation The new orientation of the device.
     *
     *  @see #ORIENTATION_UNKNOWN
     */
 public void onOrientationChanged(int orientation) {
        if (orientation == ORIENTATION_UNKNOWN) return;
        android.hardware.Camera.CameraInfo info =
                new android.hardware.Camera.CameraInfo();
        android.hardware.Camera.getCameraInfo(cameraId, info);
        orientation = (orientation + 45) / 90 * 90;
        int rotation = 0;
        if (info.facing == CameraInfo.CAMERA_FACING_FRONT) {
            rotation = (info.orientation - orientation + 360) % 360;
        } else {  // back-facing camera
            rotation = (info.orientation + orientation) % 360;
        }
        mParameters.setRotation(rotation);
    }
```

拍照成功后获得bytes，从exif中获取照片旋转角度
```
/**
ORIENTATION_NORMAL
Constant Value: 1 (0x00000001)

ORIENTATION_ROTATE_180
Constant Value: 3 (0x00000003)

ORIENTATION_ROTATE_270
Constant Value: 8 (0x00000008)

ORIENTATION_ROTATE_90
Constant Value: 6 (0x00000006)
**/
    public ExifInterface getExifInterface() throws IOException {
        if (exif == null) {
            exif = new ExifInterface();

            exif.readExif(jpegOriginal);
        }

        return (exif);
    }

    public int getOrientation() throws IOException {
        ExifTag tag = getExifInterface().getTag(ExifInterface.TAG_ORIENTATION);

        return (tag == null ? -1 : tag.getValueAsInt(-1));
    }
```

旋转照片
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

