---
title: PopupWindow show at location
date: 2018/7/25
categories: Android
---
##PopupWindow show at location
Recently,UE Team has updated some features.One of them is click on the button,and show PopupWindow right above the button.
The point is can't use hardcode because different devices has different resolution. So the best way is get the location in the window.
There is method named ``getLocationInWindow(View view)`` in View.java.All we have to do is invoke the method by the button and pass an array.This array will store two value,X coordinate and Y coordinate.
```java
int[] viewLocation = new int[2];
view.getLocationInWindow(viewLocation);
//X coordinate viewLocation[0]
//Y coordinate viewLocation[1]
```

