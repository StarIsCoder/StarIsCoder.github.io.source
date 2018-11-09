---
title: Screenshot
date: 2018/11/09
categories: Android
---
虽然没做过android screenshot，但是一直以为调用某个系统提供的api，今天搜索了一下才知道各个版本之间的screenshot方式还不太一样。

4.0以下采用JNI ： http://blog.csdn.net/zmyde2010/article/details/6925498

4.0~4.2采用反射方法获取截屏api ：http://blog.csdn.net/cjd6568358/article/details/39120037

4.3~4.4 api被hide标签修饰，需要root，执行`adb shell  /system/bin/screencap -p /sdcard/screenshot.png`

5.0以上则主要用MediaProjection来实现。

其实也不算很复杂，用MediaProjectionManager的`createScreenCaptureIntent`方法构建出一个intent并且发送出去就行。
```java
void startScreenshot() {
    startActivityForResult(mediaProjectionManager.createScreenCaptureIntent(), REQUEST_MEDIA_PROJECTION);
}
```
查看源码其实是直接启动一个权限相关的Activity弹框，实际显示的时候也会弹框出现。当用户允许了之后就会开始截屏操作。
```java
public Intent createScreenCaptureIntent() {
    Intent i = new Intent();
    final ComponentName mediaProjectionPermissionDialogComponent =
            ComponentName.unflattenFromString(mContext.getResources().getString(
                    com.android.internal.R.string
                    .config_mediaProjectionPermissionDialogComponent));
    i.setComponent(mediaProjectionPermissionDialogComponent);
    return i;
}

<string name="config_mediaProjectionPermissionDialogComponent" translatable="false">
    com.android.systemui/com.android.systemui.media.MediaProjectionPermissionActivity
</string>
```
当然这边只是启动screenshot，还需要拿到screenshot的数据。既然是`startActivityForResult`，那肯定是在`onActivityResult`方法中拿到数据的。
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    if (requestCode == REQUEST_MEDIA_PROJECTION) {
        resultData = data;
        this.resultCode = resultCode;
    }
    mediaProjection = mediaProjectionManager.getMediaProjection(this.resultCode, resultData);
    mediaProjection.createVirtualDisplay("Screenshot", surfaceView.getWidth(),
                surfaceView.getHeight(), screenDensity,
                DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
                mSurface, null, null);
}
```
只要能正确实例化MediaProjection就可以拿到screenshot的数据了，这边是直接将其显示在surface上，如果希望保存下来的话需要实例化一个ImageReader。
```java
ImageReader imageReader = ImageReader.newInstance(displayWidth, displayWidth, PixelFormat.RGBA_8888, 2);
imageReader.setOnImageAvailableListener(new ImageReader.OnImageAvailableListener() {
    @Override
    public void onImageAvailable(ImageReader reader) {
        ......
        //Get screenshot
        image = reader.acquireLatestImage();
        Bitmap bitmap = convertImagetoBitmap(image);
        ......
        imageView.setImageBitmap(bitmap);
    }
}, null);
```

不过，对比一下Qt下的截屏，Android真的是太复杂了，而且暂时也想不到有什么跨平台的方式能实现Android平台的screenshot。
```C++
int screenNumber = QApplication::desktop()->screenNumber(QCursor::pos());
QScreen* screen = QApplication::screens().at(screenNumber);
QRect screenGeometry = screen->geometry();
QPixmap pixmap = screen->grabWindow(0, screenGeometry.x(), screenGeometry.y(), screenGeometry.width(), screenGeometry.height());
QImage screenshot = pixmap.toImage();
```
## 总结
1. 发送intent
```java
startActivityForResult(mediaProjectionManager.createScreenCaptureIntent(), REQUEST_MEDIA_PROJECTION);
```
2. 覆写onActivityResult，在方法中拿到实例化的MediaRejection
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    ......
    mediaProjection = mediaProjectionManager.getMediaProjection(this.resultCode, resultData);
}
```
3. 如果要显示在SurfaceView上，就传入SurfaceView的Surface，如果要保存数据则传入`imageReader.getSurface()`
```java
mediaProjection.createVirtualDisplay("Screenshot", surfaceView.getWidth(),
                surfaceView.getHeight(), screenDensity,
                DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
                //surfaceView.getHolder().getSurface()
                //imageReader.getSurface()
                mSurface, null, null);
```
