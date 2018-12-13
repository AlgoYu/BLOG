---
title: SVG矢量(向量)详解及Android开发
id: SVGForAndroid
directory: Guide
tags:
  - Android
  - SVG
cover: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542860358885&di=05fbdec08118357cbf93f01d19d44d54&imgtype=0&src=http%3A%2F%2Fp18.qhimg.com%2Fd%2F_open360%2Fdesign0326%2F12.jpg
date: 2018-11-22 09:29:34
---
> **什么是SVG？**
可缩放矢量图形是基于可扩展标记语言（标准通用标记语言的子集），用于描述二维矢量图形的一种图形格式。它由万维网联盟制定，是一个开放标准。

> - SVG 指可伸缩矢量图形 (Scalable Vector Graphics)
- SVG 用来定义用于网络的基于矢量的图形
- SVG 使用 XML 格式定义图形
- SVG 图像在放大或改变尺寸的情况下其图形质量不会有所损失
- SVG 是万维网联盟的标准
- SVG 与诸如 DOM 和 XSL 之类的 W3C 标准是一个整体

## 在Android中如何使用SVG？ ##

>Android 5.0（API级别21）是第一个正式支持矢量绘图的版本，可以通过VectorDrawable和AnimatedVectorDrawable来绘制矢量图形，你可以使用Android支持库支持旧版本，支持库提供了VectorDrawableCompat和 AnimatedVectorDrawableCompat类来向下兼容。但VectorDrawable只支持部分SVG的属性，相当于精简版本，不过也能绘制出所有你想要的图形。

## VectorDrawable ##
>VectorDrawable定义静态可绘制对象。与SVG格式类似，每个矢量图形被定义为树形层，由树Path和Group组成。每个的Path包含对象轮廓的几何形状，并由Group包含转换的详细信息。所有Path都会按XML文件中出现的顺序绘制。

![](https://developer.android.com/images/guide/topics/graphics/vectorpath.png)

>在Android Studio中提供了一个工具Vector asset studio来将一个矢量图形添加到项目中作为XML文件，这点可以自行百度了解一下。

这是一个VectorDrawable XML文件来渲染一个电池充电的图像示例：
```xml
<!-- res/drawable/battery_charging.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    <!-- intrinsic size of the drawable -->
    android:height="24dp"
    android:width="24dp"
    <!-- size of the virtual canvas -->
    android:viewportWidth="24.0"
    android:viewportHeight="24.0">
   <group
         android:name="rotationGroup"
         android:pivotX="10.0"
         android:pivotY="10.0"
         android:rotation="15.0" >
      <path
        android:name="vect"
        android:fillColor="#FF000000"
        android:pathData="M15.67,4H14V2h-4v2H8.33C7.6,4 7,4.6 7,5.33V9h4.93L13,7v2h4V5.33C17,4.6 16.4,4 15.67,4z"
        android:fillAlpha=".3"/>
      <path
        android:name="draw"
        android:fillColor="#FF000000"
        android:pathData="M13,12.5h2L11,20v-5.5H9L11.93,9H7v11.67C7,21.4 7.6,22 8.33,22h7.33c0.74,0 1.34,-0.6 1.34,-1.33V9h-4v3.5z"/>
   </group>
</vector>
```
![](https://developer.android.com/images/guide/topics/graphics/ic_battery_charging_80_black_24dp.png)
## 额外补充篇：语法知识 ##
> - `width`、`height`代表了图片的宽高。
- `viewportWidth`、`viewportHeight`意味着把上面的图片宽高`width`、`height`等分成多少份。（示例中把宽24dp高24dp的正方形分成24*24的网格，路径就是用这些网格坐标来描述位置、并连接成图形的。）
- `fillColor`是图形填充的颜色。
- `pathData`描述网格坐标的标签。

在SVG中，最重要的还是pathData，里面包含了整个矢量图形的坐标路径，它基本上就是字母和数字组成,数字之间可以用空格或者逗号隔开 (但其实逗号会被忽略掉，只是方便我们阅读)。它的命令如下：

> - **M(Move To)：** 移动虚拟画笔到对应的点,但不会绘制，默认的位置是在（0,0）。
- **L(Line To)：** 从当前位置画一条直线到对应的坐标点。
- **H(Horizontal Line To)：** 从当前位置画一条水平直线到对应的坐标点。
- **V(Vertical Lineto)：** 从当前位置画一条垂直直线到对应的坐标点。
- **A(Elliptical Arc)：** 从当前位置画一条椭圆弧线到对应的坐标点。
- **Q(Quadratic Belzier Curve)：** 从当前位置画一条二阶贝塞尔曲线到对应的坐标点。
- **T(Smooth Quadratic Belzier Curveto)：** 当你已经画完一条二阶贝塞尔曲线后，再次延长贝塞尔曲线到相应的坐标点，控制点会根据前一个二阶贝塞尔曲线来推断。
- **C(Curveto)：** 从当前位置画一条三阶贝塞尔曲线到对应的坐标点。
- **S(Smooth Curvet)：** 当你已经画完一条三阶贝塞尔曲线后，再次延长贝塞尔曲线到相应的坐标点，控制点会根据前一个三阶贝塞尔曲线来推断。
- **Z(Close Path)：** 从当前位置画一条线连接起始位置，闭合路径。
<br/>**注意：命令区分大小写，大写命令是基于原点（0,0）的坐标，即绝对坐标，小写的命令是基于当前虚拟画笔的位置，即相对坐标。**

**M：x,y（移动虚拟画笔）**
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="50dp"
    android:height="50dp"
    android:viewportWidth="50"
    android:viewportHeight="50">

    <group>
        <path android:strokeWidth="1"
            android:strokeColor="@color/colorPrimary"
            android:pathData="M10,10"/>
    </group>

</vector>
```
**L：x,y（线段）Z（闭合路径）**
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="50dp"
    android:height="50dp"
    android:viewportWidth="50"
    android:viewportHeight="50">

    <group>
        <path android:strokeWidth="1"
            android:strokeColor="@color/colorPrimary"
            android:pathData="M10,10 l30,0 l0,30 l-30,0 z"/>
    </group>

</vector>
```
![](http://wx2.sinaimg.cn/large/0065B4vHgy1fxgx6h3f7vj308007v0sh.jpg)
通过L命令给出的坐标让我们渲染出了一个正方形，最后使用Z直接闭合路径。由于这样的正方形是垂直和水平线组合成的，我们也可以使用V、H来简写这个路径数据，效果是一样的。
**V：y（垂直线） H：x（水平线）**
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="50dp"
    android:height="50dp"
    android:viewportWidth="50"
    android:viewportHeight="50">

    <group>
        <path android:strokeWidth="1"
            android:strokeColor="@color/colorPrimary"
            android:pathData="M10,10 h30 v30 h-30 z"/>
    </group>

</vector>
```
![](http://wx2.sinaimg.cn/large/0065B4vHgy1fxgx6h3f7vj308007v0sh.jpg)
**A：rx,ry x-axis-rotation large-arc-flag,sweepflag x,y（弧线）**

 >- **rx ry：**这个椭圆的x轴半径和y轴半径。
- **x-axis-rotation：** x轴旋转角度。
- **large-arc-flag：** 为0时表示取小弧度，1时取大弧度。
- **sweep-flag：** 0取逆时针方向，1取顺时针方向
- **x,y：**终点的坐标

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="50dp"
    android:height="50dp"
    android:viewportWidth="50"
    android:viewportHeight="50">

    <group>
        <path android:strokeWidth="1"
            android:strokeColor="@color/colorPrimary"
            android:pathData="M10,25 a15,15 0 0 1 30,0 z"/>
    </group>

</vector>
```
![](http://wx1.sinaimg.cn/large/0065B4vHgy1fxgx6kziyaj308d067dfo.jpg)
设置rx,ry半径，其他几个参数根据需要修改，再设置x,y坐标，一条弧线就被渲染出来了，下面是弧线总结图。
![](https://upload-images.jianshu.io/upload_images/1175492-c79b5e4bf39363c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/465/format/webp)
**Q：x1,y1 x,y（二阶贝塞尔曲线）T：x,y（平滑的二次二阶贝塞尔曲线）**
二阶贝塞尔曲线公式：
![](https://upload-images.jianshu.io/upload_images/1175492-e659f1bd3f9dd7c4?imageMogr2/auto-orient/strip%7CimageView2/2/w/422/format/webp)

![](https://upload-images.jianshu.io/upload_images/1175492-67b35c33f0aeda7f.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/360/format/webp)
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="50dp"
    android:height="50dp"
    android:viewportWidth="50"
    android:viewportHeight="50">

    <group>
        <path android:strokeWidth="1"
            android:strokeColor="@color/colorPrimary"
            android:pathData="M10,25 q7,-20 15,0 t15,0 z"/>
    </group>

</vector>
```
![](http://wx3.sinaimg.cn/large/0065B4vHgy1fxgx6puqykj307v05ta9x.jpg)
通过设置的x1,y1来确定P1的坐标，再设置x,y确定P2的坐标，结合虚拟画笔当前的坐标（P0）就会计算出t的数值，然后渲染出一条二阶贝塞尔曲线，而T指令在你渲染出一条贝塞尔曲线后，只需要指定终点坐标（P2），会再次延伸出一条贝塞尔曲线，P1的坐标是上一条条贝塞尔曲线的对称坐标，这样就会产生上图效果，你还能继续添加T命令来延长它。
**C：x1,y1 x2,y2 x,y（三阶贝塞尔曲线） S：x2,y2 x,y（平滑的二次三阶贝塞尔曲线）**

![](https://upload-images.jianshu.io/upload_images/1175492-d45534739c6a0b62?imageMogr2/auto-orient/strip%7CimageView2/2/w/558/format/webp)

![](https://upload-images.jianshu.io/upload_images/1175492-6b58a1da6d923644?imageMogr2/auto-orient/strip%7CimageView2/2/w/360/format/webp)
```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="50dp"
    android:height="50dp"
    android:viewportWidth="50"
    android:viewportHeight="50">

    <group>
        <path android:strokeWidth="1"
            android:strokeColor="@color/colorPrimary"
            android:pathData="M10,25 c0,-20 15,-20 15,0 s15,20 15,0"/>
    </group>

</vector>
```
![](http://wx4.sinaimg.cn/large/0065B4vHgy1fxgx6tcqicj307x07l3ye.jpg)
三阶贝塞尔曲线与二阶贝塞尔曲线类似，但多了一个控制点P2，通过虚拟画笔当前的坐标（P0），x1,y1的坐标（P1）、x2,y2（P2）、x,y计算出t的值，渲染出一条三阶贝塞尔曲线。S命令与T类似，P1的坐标仍然是上一条贝塞尔曲线的P1对称坐标，指定P2，P3的坐标即可渲染二次三阶贝塞尔曲线，你还能继续添加S命令来延长它。

## AnimatedVectorDrawable  ##
AnimatedVectorDrawable为矢量图形的属性添加了动画。您可以将一个动画向量图形定义为几个单独的资源文件，或者将其定义为一个单独的XML文件。就是说可以使用多个XML文件或者单个XML文件的方式来完成。
多个文件示例：

> - 一个VectorDrawable XML文件。
- 一个AnimatedVectorDrawable XML文件，它定义了目标VectorDrawable、要动画的目标路径和组、属性以及ObjectAnimator对象或AnimatorSet对象定义的动画。
- 一个Animator XML文件

这是一个VectorDrawable矢量图形文件：`vd.xml`
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
   android:height="64dp"
   android:width="64dp"
   android:viewportHeight="600"
   android:viewportWidth="600" >
   <group
      android:name="rotationGroup"
      android:pivotX="300.0"
      android:pivotY="300.0"
      android:rotation="45.0" >
      <path
         android:name="vectorPath"
         android:fillColor="#000000"
         android:pathData="M300,70 l 0,-70 70,70 0,0 -70,70z" />
   </group>
</vector>
```
这是一个AnimatedVectorDrawable矢量动画文件：`avd.xml`
```xml
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
   android:drawable="@drawable/vd" >
     <target
         android:name="rotationGroup"
         android:animation="@anim/rotation" />
     <target
         android:name="vectorPath"
         android:animation="@anim/path_morph" />
</animated-vector>
```
在AnimatedVectorDrawable的XML文件中使用的Animator XML文件:`rotation.xml`和`path_morph.xml`
```xml
<objectAnimator
   android:duration="6000"
   android:propertyName="rotation"
   android:valueFrom="0"
   android:valueTo="360" />
```
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
   <objectAnimator
      android:duration="3000"
      android:propertyName="pathData"
      android:valueFrom="M300,70 l 0,-70 70,70 0,0   -70,70z"
      android:valueTo="M300,70 l 0,-70 70,0  0,140 -70,0 z"
      android:valueType="pathType"/>
</set>
```

----------


 
单个文件示例：
```xml
<animated-vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:aapt="http://schemas.android.com/aapt">
    <aapt:attr name="android:drawable">
        <vector
            android:width="24dp"
            android:height="24dp"
            android:viewportWidth="24"
            android:viewportHeight="24">
            <path
                android:name="root"
                android:strokeWidth="2"
                android:strokeLineCap="square"
                android:strokeColor="?android:colorControlNormal"
                android:pathData="M4.8,13.4 L9,17.6 M10.4,16.2 L19.6,7" />
        </vector>
    </aapt:attr>
    <target android:name="root">
        <aapt:attr name="android:animation">
            <objectAnimator
                android:propertyName="pathData"
                android:valueFrom="M4.8,13.4 L9,17.6 M10.4,16.2 L19.6,7"
                android:valueTo="M6.4,6.4 L17.6,17.6 M6.4,17.6 L17.6,6.4"
                android:duration="300"
                android:interpolator="@android:interpolator/fast_out_slow_in"
                android:valueType="pathType" />
        </aapt:attr>
    </target>
</animated-vector>
```
>通过使用这种方法，您可以通过XML Bundle格式将相关的XML文件合并到单个XML文件中。在构建应用程序时，aapt标记会创建单独的资源并在矢量动画中引用它们。这种方法需要构建工具24或更高版本，并且输出是向上兼容的。

<b><font color="FF0000">文章内容均为本人手写；<br>禁止转载，请尊重原著；<br>本人QQ：794763733。</font></b>