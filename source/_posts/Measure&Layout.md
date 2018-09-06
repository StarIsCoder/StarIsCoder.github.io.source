---
title: View的Measure&和Layout
date: 2018/9/6
categories: Android
---
在自定义view的时候比较重要的就是三个方法。
* onMeasure(int widthMeasureSpec, int heightMeasureSpec)
* onLayout(boolean changed, int left, int top, int right, int bottom)
* onDraw(Canvas canvas)

# onMeasure
onMeasure方法是在view的measure方法中调用的。主要是控制控件的尺寸。
```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {

            ......

            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // measure ourselves, this should set the measured dimension flag back
                onMeasure(widthMeasureSpec, heightMeasureSpec);
                mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                long value = mMeasureCache.valueAt(cacheIndex);
                // Casting a long to int drops the high 32 bits, no mask needed
                setMeasuredDimensionRaw((int) (value >> 32), (int) value);
                mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            }

            ......

```
继续看onMeasure的实现，一直到最后一系列的调用就是给mMeasuredWidth和mMeasuredHeight赋值。
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```
```java
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight){
    boolean optical = isLayoutModeOptical(this);
    if (optical != isLayoutModeOptical(mParent)) {
        Insets insets = getOpticalInsets();
        int opticalWidth  = insets.left + insets.right;
        int opticalHeight = insets.top  + insets.bottom;

        measuredWidth  += optical ? opticalWidth  : -opticalWidth;
        measuredHeight += optical ? opticalHeight : -opticalHeight;
    }
    setMeasuredDimensionRaw(measuredWidth, measuredHeight);
}
```
```java
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
    }
    return result;
}
```
插播一段：因为在调用measure方法的时候传入的两个参数widthMeasureSpec和heightMeasureSpec都是通过MeasureSpec构造出来的，因此这个类可以简单了解下。

这个measureSpec是一个32位int类型的数据，前两位表示mode，后30位表示size。因为他的掩码是11后面加30个零。例如getMode，掩码和measureSpec进行与运算，得到的一定是measureSpec的前两位，即mode值。也就是说根据mode值来设置返回不同的size。
```java
private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

public static int getMode(int measureSpec) {
    return (measureSpec & MODE_MASK);
}

public static int getSize(int measureSpec) {
    return (measureSpec & ~MODE_MASK);
}
```
**因此如果复写onMeasure方法，能改动的变量也只有重新设置mMeasuredHeight和mMeasuredWidth。也就是调用setMeasuredDimension这个方法，将我们需要的height和width传进去即可。**

# onLayout
onLayout方法一样也是在layout()方法中调用的，作用是控制子控件的布局。因此view的onLayout方法是空的,因为只有ViewGroup才有子控件。不过ViewGroup的onLayout方法也只是个抽象类，实现的地方还是在ViewGroup的子类中。
```java
@Override
    protected abstract void onLayout(boolean changed,int l, int t, int r, int b);
```
例如LinearLayout的onLayout方法,根据orientation进行不同的布局排版。
```java
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l, t, r, b);
    }
}
```

之前碰到一个坑就是在vertical的LinearLayout情况下gravity的top和bottom不生效，当时理解为因为纵向线形布局的高度都是根据子view来决定的，这边看下源码印象更深刻,以vertical为例。
```java
for (int i = 0; i < count; i++) {

    ......

    switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
        case Gravity.CENTER_HORIZONTAL:
            childLeft = paddingLeft + ((childSpace - childWidth) / 2)
                                + lp.leftMargin - lp.rightMargin;
            break;

         case Gravity.RIGHT:
            childLeft = childRight - childWidth - lp.rightMargin;
            break;

        case Gravity.LEFT:
        default:
            childLeft = paddingLeft + lp.leftMargin;
            break;
    }

    ......

}
```
在layoutVertical中对子view的排版只有RIGHT和LEFT两个属性，并没有对TOP和BOTTOM处理。

其余还有些padding和margin等balabala各种属性要考虑到其中就不赘述了。

# 自定义Layout属性
在学习写自定义Layout的时候，顺便练习了下自定义属性，主要以下几步
* 新建attrs.xml,declare-styleable的name是自定义的布局名称，attr的name和format都自己定义
```xml
<resources>

    <declare-styleable name="MyLayout">
        <attr name="isVertical" format="boolean" />
    </declare-styleable>

</resources>
```
* 在布局文件中设置该属性
```xml
<MyLayout 
    .......
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:isVertical="true"
    tools:context=".MainActivity">
```
* 在该类初始化的时候获取属性值
```kotlin
var isVertical: Boolean = false
var t = context?.obtainStyledAttributes(attrs, R.styleable.MyLayout)
isVertical = t!!.getBoolean(R.styleable.MyLayout_isVertical, false)
```