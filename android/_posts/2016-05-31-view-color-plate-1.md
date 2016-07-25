---
layout: post
title: "自定义取色盘（上）"
modified: 2016-5-31 16:28:00
excerpt: "一个取色盘，通过拖动中间的按钮来获取按钮上标示所指示的颜色值"
tags: [Android, View, ColorPlate]
comments: true
---

前言，最近开发中遇到一个环形色盘取色器，效果图如下：  
<img src="http://www.ionesmile.com/images/android/android_color_plate_design.png" width="300"/>  
说一下它分为外侧的色环和内面的转盘两部分，通过旋转内面的圆盘来获取外环色盘上指针所指方向的颜色，内面的转盘很好解决，网上可以找到很多Demo，我们拿过来稍作修改就直接使用啦，而外环色盘就是一个大问题了。   
外环色盘这里涉及到两点，一个是如何绘制一个这样270度的色环，另一个更为复杂的问题是给这个圆盘设置一个颜色值后，让圆盘上的指针指对应的位置。


**一、绘制一个270°的色环**  


1. 首先绘制一个用SweepGradient渲染出一个带有颜色的圆；
2. 创建一个新的Bitmap，将第一步绘制的图像旋转90°加入进来，由于渲染出来的圆红色部分在直角坐标系的0°位置，而效果图在270°位置，又因为0°与360°两端的颜色无法混合，导致一个很明显的分割线，非常影响效果，故在此逆时针选择90°；
3. 裁剪图片，得到一个270°的环形

关键代码如下：  

	private Bitmap getArcBitmap(int width, int height){
        // 绘制一个渲染的圆
        Bitmap bmp = Bitmap.createBitmap(width, height, Config.ARGB_8888);   
        Canvas canvas = new Canvas(bmp); 
        canvas.drawArc(arcRect, 0, 360, false, progressBgPaint);
        // 创建一个结果Bitmap
        Bitmap tagBitmap = Bitmap.createBitmap(width, height, Config.ARGB_8888);
        Canvas tagCanvas = new Canvas(tagBitmap);
        Paint paint = new Paint();   
        paint.setAntiAlias(true);
        paint.setColor(Color.WHITE);
        Matrix matrix = new Matrix();
        // 将Bitmap旋转90°，处理0°角颜色无法融为一体的样式问题
        matrix.setRotate(90, centerX, centerY);
        tagCanvas.drawBitmap(bmp, matrix, paint);
        // 绘制颜色环的外形，从135°开始，绘制270°
        Bitmap arcBitmap = Bitmap.createBitmap(width, height, Config.ARGB_8888);
        Canvas arcCanvas = new Canvas(arcBitmap);
        paint.setStrokeWidth(progressWidth);
        paint.setStyle(Style.STROKE);
        paint.setStrokeCap(Cap.ROUND);
        arcCanvas.drawArc(arcRect, START_ANGLE, MAX_SWEEP_ANGLE, false, paint);
        // 设置合成模式，以bmp为底，arcBitmap为覆盖，生成一个arcBitmap形状的bmp图形
        paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));  
        tagCanvas.drawBitmap(arcBitmap, new Matrix(), paint);
        return tagBitmap;
    }

    // 对笔进行环形渲染
    int[] colorAngle = new int[]{360, 330, 300, 270, 240, 210, 180, 150, 120, 90, 60, 30, 0};
    // 每个角度对应的位置，位置生成方式待续...
    float[] positions = new float[]{0.125f, 0.1875f, 0.25f, 0.3125f, 0.375f, 0.4375f, 0.5f, 0.5648148f, 0.6296296f, 0.6944444f, 0.7592593f, 0.8240741f, 0.8888889f};
    int[] colors = new int[colorAngle.length];
    float hsv[] = new float[] { 0f, 1f, 1f };
    for (int i = 0; i < colors.length; i++) {
        // 根据HSV色盘，旋转的角度取色
        hsv[0] = colorAngle[i];
        colors[i] = Color.HSVToColor(hsv);
    }
    SweepGradient sweepGradient = new SweepGradient(getWidth() / 2, getHeight() / 2, colors, positions);
    progressBgPaint.setShader(sweepGradient);


　  
**二、色值->角度的对应**  
 
调用Android的Color.colorToHSV(color, hsv)函数，可以将颜色转换为一个HSV模型。  
hsv的数据类型是float[3]，第0位表示H（0-360），第1位表示S，第2位表示V。  
通过这一步我们就可以获取到该色值所对应的角度，再把该值等比压缩到一个270°环形上对应角度就OK了。  

关键代码如下：

	// 将颜色转换为角度
    private float getColorAngle(int color) {
        // 第一步，转为HSV模型
        float[] hsv = new float[3];
        Color.colorToHSV(color, hsv);
        // 获取角度H（0-360）
        float angle = hsv[0];
        /*
         * 修改最左边的颜色值与最右边颜色值一致，导致底部颜色（红色）没有一个固定角度的问题
         * 现在处理为损失（0-(180/140f * 5)）这个角度的颜色，左半部分不变压缩为135°，右半部分压缩为140°（色盘上135°-140°颜色无法获取到，对应HSV的（0-6.428））
         */
        if (angle <= 5) {
            angle = 360;
        }
        // 以180°为中心点，乘以两边压缩比，色盘角度45-335的角度
        if (angle >= 180) {
            angle = (angle-180)*(135/180f) + 180;
        } else {
            angle = (angle-180)*(140/180f) + 180;
        }
        // 转成0-270°
        angle -= 45;
        // 反转，与显示的图片对应
        angle = 270 - angle;
        return angle;
    }

