---
layout: post
title: "自定义滚动按钮调节器"
modified: 2016-7-25 20:36:00
excerpt: "滚轴效果的进度条"
tags: [Android, View, progressbar]
comments: true
---


**一、效果图如下：**  

<img src="http://www.ionesmile.com/images/android/android_view_srcollball.png" width="300"/>

一个滚动条，通过上下滚动来调节进度。这里的难点是滚动时的动画效果，下面一一说来。

　  
**二、实现思路**  

1，绘制一组动画图片，从第一个小横杠开始到第一个小横杠结束滚动的这一过程我们用20帧来表示   

1. 绘制中间的滚动轴
1. 用得到的滚动轴与一个比背景圆略小的圆合成，插入滚轴Y值的偏移量，用背景圆减去滚轴多余的部分
1. 把滚轴和图片背景相结合生成最终的图片
1. 不断改变滚轴与背景圆的在Y轴的偏移量，生成一组动画图片

2，重写View，显示滚动效果    
View的onTouch()方法，监听MOVE事件，在滑动改变时改变进度，并根据进度来得到此时应该绘制图片的Index帧

	1，初始化View大小，动画帧等
            在构造函数中初始化View大小，动画帧等。我在这里只重写了ScrollBallView(Context context, AttributeSet attrs)构造函数，由于在布局文件中是重写调用该方法来创建对象。当然如果有需要最好把View的三个构造函数都重写了吧。

    2，重写onTouchEvent()方法，监听DOWN、MOVE、UP事件
        在DOWN事件中，判断是否可以滑动View，以及初始化相关参数，并返回true来继续接受之后的事件
        在MOVE事件中，获取每次移动的距离，通过该值的变化来改变进度和确定显示哪一帧图片，最后触发监听事件
        在UP事件中，再次初始化并释放相关资源

    3，MOVE事件分析
        在MOVE事件中，我们调用了@setScroll(int dy)方法处理滑动的值，
            传入的值dy为本次MOVE时在屏幕上的触摸点与上次在屏幕上触摸点的差值，数字的大小与手指在屏幕上移动的速度有关，不同方向得到正负值
            然后把滑动值记录下来，当滚动值大于最大值或小于0时，限制值，并不做更新，只有在合法范围内才去做响应的更新
            正常情况下会进入@getScrollIndex(int dy)方法，把值移动的距离累加，之后模以移动每一个刻度显示的帧数，就是当前要显示哪一帧了

3，给自定义View添加监听    
在MotionEvent.ACTION_MOVE事件中回调监听，获取一个百分比的进度。

注：滑动总数为Interger的一半，是为了解决Interger为负数时滚动条的滚动方向与手势方向相反的BUG。   

	mScrollIndexSum = Integer.MAX_VALUE / 2


**三、Demo：[https://github.com/iOnesmile/ScrollBallDemo](https://github.com/iOnesmile/ScrollBallDemo "https://github.com/iOnesmile/ScrollBallDemo")**  
