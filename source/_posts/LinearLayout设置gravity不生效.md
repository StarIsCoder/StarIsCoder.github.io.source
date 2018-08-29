---
title: LinearLayout设置gravity不生效
date: 2018/8/29
categories: Android
---
最近在学习Transition动画的时候，有个动画是将button从左上移到右下，代码其实很简单。
```kotlin
TransitionManager.beginDelayedTransition(transitionContainer, ChangeBounds().setPathMotion(ArcMotion()).setDuration(1000))
var params = button_view.layoutParams as FrameLayout.LayoutParams
params.gravity = Gravity.BOTTOM or Gravity.RIGHT
button_view.layoutParams = params
```
这边使用的是FrameLayout,最开始使用LinearLayout的时候修改gravity始终无效。后来意识到因为LinearLayout是或横向或纵向的布局，因此如果当它处于横向的情况时，只有top和bottom是有效的，因为横向的布局都依赖于子控件的个数。反之如果是纵向布局，只有right和left是有效的。
