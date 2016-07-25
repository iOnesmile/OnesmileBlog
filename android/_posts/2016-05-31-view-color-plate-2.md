---
layout: post
title: "自定义取色盘（下）"
modified: 2016-5-31 16:36:00
excerpt: "一个取色盘，通过拖动中间的按钮来获取按钮上标示所指示的颜色值"
tags: [Android, View, ColorPlate]
comments: true
---

介绍完上一章，下面我们来分解其中的一些知识点，来一一认识下吧。  

**一、HSV模型了解**  

<img src="http://www.ionesmile.com/images/android/android_hsv_model.png" width="600"/>

它是一个倒锥子模型，这个模型就是按色彩、深浅、明暗来描述的。  

- H是色彩，范围0° ~ 360°，红（0°）、绿（120°）、蓝（240°）；
- S是深浅， S = 0时，只有灰度，越往圆心的位置越偏白；
- V是明暗，表示色彩的明亮程度，越往锥子的顶点越偏黑（v = max(r, g, b)）。

由RGB到HSV的转换：  
<img src="http://www.ionesmile.com/images/android/android_hsv_model_formula.png" width="600"/>     
参考网址：[http://blog.csdn.net/viewcode/article/details/8203728](http://blog.csdn.net/viewcode/article/details/8203728)  

　  
**二、图像渲染（Shader）**  

在Android中，提供了Shader类专门用来渲染图像以及一些几何图形。    
Shader类包括了5个直接子类，分别为：BitmapShader、ComposeShader、LinearGradient、RadialGradient以及SweepGradient。其中，BitmapShader用于图像渲染；ComposeShader用于混合渲染；LinearGradient用于线性渲染；RadialGradient用于环形渲染；而SweepGradient则用于梯度渲染。显示效果如下图：  
<img src="http://www.ionesmile.com/images/android/android_shader_effect.png" width="700"/>    
参考网址：[http://www.cnblogs.com/menlsh/archive/2012/12/09/2810372.html](http://www.cnblogs.com/menlsh/archive/2012/12/09/2810372.html)

　  
**三、色环角度的等比压缩，360°-270°的加压和解压**  
 
这里我再唠叨一下说说从360°转到270°色盘是怎样实现的，来让渲染的色盘与实际的值尽可能的接近。  
回到之前所说的SweepGradient，它的构造函数的第四个参数位置比例我传递的是new float[]{0.125f, 0.1875f, 0.25f, 0.3125f, 0.375f, 0.4375f, 0.5f, 0.5648148f, 0.6296296f, 0.6944444f, 0.7592593f, 0.8240741f, 0.8888889f}，第三个参数颜色我传递的是new int[]{360, 330, 300, 270, 240, 210, 180, 150, 120, 90, 60, 30, 0}HSV角度对应的颜色。   
位置渲染是从0.125f开始到0.8888889f结尾，这个值怎么得来，我们先转换成360°的角度是45°和320°，如下图：  
<img src="http://www.ionesmile.com/images/android/android_color_plate_analyze.png" width="700"/>    

现在说说为什么是320°，而不是315°。是因为如果是45-315°，可以发现这两个点其实都代表的是红色，这样一个颜色对应的就有两个位置，色值与角度达不到一一对应关系，所以在此这样实现。修改后色盘渲染的是45-320°，而View所能显示出来的是45-315°，这样315-320°这个范围的色值被舍弃并统一归为45°的红色。  

色盘角度和位置的生成代码如下：    

	private static void printPlateAngleAndOffsetPlus() {
        int partNum = 13;    // 生成12 + 1个代表点
        int compressPoint = 180;    // 以中心点作为压缩中心
        // 生成角度原（[360, 330, 300, 270, 240, 210, 180, 150, 120, 90, 60, 30, 0]）
        int[] sourceAngle = new int[partNum];
        for (int i = 0; i < sourceAngle.length; i++) {
            sourceAngle[i] = 360 - i*(360/(partNum-1));
        }
        System.out.println(Arrays.toString(sourceAngle));

        float percentLeft = 135/180f;
        float percentRight = 140/180f;
        int[] targetAngle = new int[partNum];
        for (int i = 0; i < sourceAngle.length; i++) {
            if (sourceAngle[i] >= 180) {
                targetAngle[i] = (int) ((sourceAngle[i]-compressPoint)*percentLeft + compressPoint);
            } else {
                targetAngle[i] = (int) ((sourceAngle[i]-compressPoint)*percentRight + compressPoint);
            }
        }
        System.out.println(Arrays.toString(targetAngle));


        float[] percentAngle = new float[partNum];
        for (int i = 0; i < percentAngle.length; i++) {
            if (sourceAngle[i] >= 180) {
                percentAngle[i] = 1 - (((sourceAngle[i]-compressPoint)*percentLeft + compressPoint)/360);
            } else {
                percentAngle[i] = 1 - (((sourceAngle[i]-compressPoint)*percentRight + compressPoint)/360);
            }
        }
        System.out.println(toStringF(percentAngle));
    }

    public static String toStringF(float[] a) {
        if (a == null)
            return "null";

        int iMax = a.length - 1;
        if (iMax == -1)
            return "{}";

        StringBuilder b = new StringBuilder();
        b.append('{');
        for (int i = 0; ; i++) {
            b.append(a[i]).append("f");
            if (i == iMax)
                return b.append('}').toString();
            b.append(", ");
        }
    }


　  
案例Demo：[https://github.com/iOnesmile/MyAndroidDemo/blob/master/src/com/ionesmile/variousdemo/BrightViewActivity.java](https://github.com/iOnesmile/MyAndroidDemo/blob/master/src/com/ionesmile/variousdemo/BrightViewActivity.java "Demo的Activity")