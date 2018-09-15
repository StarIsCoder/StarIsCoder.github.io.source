---
title: Matrix&Camera
date: 2018/8/29
categories: Android
---
最近在学习自定义view，先将学到了东西记录一下。
## Matrix
Matrix就是矩阵的意思，具体的数学原理已经还给老师了，因此只是记录使用方式。
由于矩阵并不满足交换律，即：
A*B ≠ B*A
因此矩阵的方法都是成对的，用pre或者post作为前缀表示前乘或者后乘，这个前和后的意思就是将初始矩阵放在前还是后的意思。
使用方法：
```java
Matrix matrix = new Matrix();
matrix.preRotate(45);
matrix.postTranslate(200, 0);
canvas.setMatrix(matrix);
canvas.drawBitmap(bitmap, 100, 100, paint);
```

Matrix的强大之处在于可以完成一切的形变，而canvas只支持一部分的形变。并且Matrix可以根据前乘和后乘（绝对不是根据简单的pre和post）来决定形变的顺序。

## Camera
Camera顾名思义就是照相机，这是一个在三维坐标轴上处于z轴的一个相机，他的工作原理就是将三维模型投影在canvas，也就是我们实际看到的图形。插播一段它的坐标系。

View的坐标系:以屏幕左上方为原点。x轴左负右正，y轴上负下正。

Camera的坐标系:以屏幕左上方为原点，x轴左负右正,y轴上正下负，z轴内正外负。

如果我们直接调用camera的rotate方法的话，假设让它绕x轴旋转30度，由于camera是始终处于z轴上的，因此如果这时候投影的话，那么投影出来的图形并不是我们想要的效果，而是不对称且右边拉长的结果。解决办法就是在旋转之前先将图片移到原点。之后再移回来，不过代码中canvas的移动要反过来写，因为它是反向执行的。
```java
camera.rotateX(30);
canvas.translate(centerX, centerY); 
camera.applyToCanvas(canvas); 
canvas.translate(-centerX, -centerY); 
```
其余的一些调用api的就不写了，等用的时候一查就有了。
