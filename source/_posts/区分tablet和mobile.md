---
title: 区分tablet和mobile
date: 2018/8/27
categories: Android
---
最近项目上UE有个需求是增加一个按钮切换横竖屏，当然很容易想到使用
```java
activity.setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
```
传入的值一般有如下，按字面意思理解即可：
```java
SCREEN_ORIENTATION_PORTRAIT
SCREEN_ORIENTATION_LANDSCAPE
SCREEN_ORIENTATION_REVERSE_PORTRAIT
SCREEN_ORIENTATION_LANDSCAPE
```
但是这样设置有个问题，那就是手动设置了横竖屏之后系统的AutoRotate失效了。有一种workaround就是手动去处理转屏事件。加一个内部类继承OrientationEventListener。只要底层重力传感器的值发生变化就会回调这个类中的onOrientationChanged方法。之后根据上报的orientation值和一些flag进行横竖屏处理。
```java
private class IncallOrientationListener extends OrientationEventListener {
    public IncallOrientationListener(Context context) {
        super(context);
    }

    @Override
    public void onOrientationChanged(int orientation) {
        //do anything what we want
        orientationUtils.onOrientationChanged(getApplicationContext(), orientation);
    }     
}
```
然而在上一次GT的时候发现平板下横竖屏全反了，原因是平板是横着的时候oreintation为0，这时候如果设置SCREEN_ORIENTATION_PORTRAIT就会有问题，当转屏之后又设置了SCREEN_ORIENTATION_LANDSCAPE。也就是说对于横着的平板和手机相比应该走完全相反的逻辑。那么就需要区分平板和手机了，官方似乎没有相关文档，但是根据以前我处理分屏下布局的bug，系统应该是根据长宽的大小比较来决定是使用port文件夹还是land文件夹下的布局。这个在stackoverflow上的workaround
```java
public static boolean isTablet(Context context) {
        return (context.getResources().getConfiguration().screenLayout & Configuration.SCREENLAYOUT_SIZE_MASK) >= Configuration.SCREENLAYOUT_SIZE_LARGE;
    }
```
其实我个人认为通过长宽来判断应该设置横屏还是竖屏更合理一点。因为有的平板就是放大版的手机，这种平板其实就应该走手机的逻辑。用WindowManagerService来获取屏幕的宽高。
```java
WindowManager wm = (WindowManager) getSystemService(Context.WINDOW_SERVICE);
DisplayMetrics metrics = new DisplayMetrics();
wm.getDefaultDisplay().getMetrics(metrics);
int heightPixels = metrics.heightPixels;
Log.d(TAG, "heightPixels :" + heightPixels);
int widthPixels = metrics.widthPixels;
Log.d(TAG, "widthPixels :" + widthPixels);
```
```
Result:
heightPixels :1440 widthPixels :2560 //横屏
heightPixels :2560 widthPixels :1440 //竖屏
```

