---
layout: post
title: "自定义View简介"
modified: 2016-3-26 13:36:00
excerpt: "Android自定义View的基础介绍......"
tags: [Android, View]
comments: true
---
###一、 View的生命周期

1. onFinishInflate() 当View中所有的子控件均被映射成xml后触发   
2. onMeasure( int ,  int ) 确定所有子元素的大小 
3. onLayout( boolean ,  int ,  int ,  int ,  int ) 当View分配所有的子元素的大小和位置时触发     
4. onSizeChanged( int ,  int ,  int ,  int ) 当view的大小发生变化时触发  
5. onDraw(Canvas) view渲染内容的细节  
6. onKeyDown( int , KeyEvent) 有按键按下后触发  
7. onKeyUp( int , KeyEvent) 有按键按下后弹起时触发  
8. onTrackballEvent(MotionEvent) 轨迹球事件  
9. onTouchEvent(MotionEvent) 触屏事件  
10. onFocusChanged( boolean ,  int , Rect) 当View获取或失去焦点时触发   
11. onWindowFocusChanged( boolean ) 当窗口包含的view获取或失去焦点时触发  
12. onAttachedToWindow() 当view被附着到一个窗口时触发  
13. onDetachedFromWindow() 当view离开附着的窗口时触发，Android123提示该方法和  onAttachedToWindow() 是相反的。  
14. onWindowVisibilityChanged( int ) 当窗口中包含的可见的view发生变化时触发 

其中，在自定义View中常用的方法有如下几个：

1. onMeasure( int ,  int ) 确定所有子元素的大小 
2. onSizeChanged( int ,  int ,  int ,  int ) 当view的大小发生变化时触发  
3. onDraw(Canvas) view渲染内容的细节  
4. onTouchEvent(MotionEvent) 触屏事件  

还有两个方法，在ViewGroup中应用的要比较多：

1. onFinishInflate() 当View中所有的子控件均被映射成xml后触发   
2. onLayout( boolean ,  int ,  int ,  int ,  int ) 当View分配所有的子元素的大小和位置时触发     

###二、 Canvas简介

#### Canvas, 它相当于一个画布，你可以在里面画很多东西；  
我们可以把这个Canvas理解成系统提供给我们的一块内存区域(但实际上它只是一套画图的API，真正的内存是下面的Bitmap)，而且它还提供了一整套对这个内存区域进行操作的方法，所有的这些操作都是画图API。也就是说在这种方式下我们已经能一笔一划或者使用Graphic来画我们所需要的东西了，要画什么要显示什么都由我们自己控制。  
这种方式根据环境还分为两种：一种就是使用普通View的canvas画图，还有一种就是使用专门的SurfaceView的canvas来画图。两种的主要是区别就是可以在SurfaceView中定义一个专门的线程来完成画图工作，应用程序不需要等待View的刷图，提高性能。前面一种适合处理量比较小，帧率比较小的动画，比如说象棋游戏之类的；而后一种主要用在游戏，高品质动画方面的画图。

下面是Canvas类常用的方法：  

1. drawRect(RectF rect, Paint paint) //绘制区域，参数一为RectF一个区域  
1. drawPath(Path path, Paint paint) //绘制一个路径，参数一为Path路径对象  
1. drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)  //贴图，参数一就是我们常规的Bitmap对象，参数二是源区域(这里是bitmap)，参数三是目标区域(应该在canvas的位置和大小)，参数四是Paint画刷对象，因为用到了缩放和拉伸的可能，当原始Rect不等于目标Rect时性能将会有大幅损失。  
1. drawLine(float startX, float startY, float stopX, float stopY, Paintpaint) //画线，参数一起始点的x轴位置，参数二起始点的y轴位置，参数三终点的x轴水平位置，参数四y轴垂直位置，最后一个参数为Paint 画刷对象。  
1. drawPoint(float x, float y, Paint paint) //画点，参数一水平x轴，参数二垂直y轴，第三个参数为Paint对象。  
1. drawText(String text, float x, floaty, Paint paint)  //渲染文本，Canvas类除了上面的还可以描绘文字，参数一是String类型的文本，参数二x轴，参数三y轴，参数四是Paint对象。  
1. drawOval(RectF oval, Paint paint)//画椭圆，参数一是扫描区域，参数二为paint对象；  
1. drawCircle(float cx, float cy, float radius,Paint paint)// 绘制圆，参数一是中心点的x轴，参数二是中心点的y轴，参数三是半径，参数四是paint对象；  
1. drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)//画弧，参数一是RectF对象，一个矩形区域椭圆形的界限用于定义在形状、大小、电弧，参数二是起始角(度)在电弧的开始，参数三扫描角(度)开始顺时针测量的，参数四是如果这是真的话,包括椭圆中心的电弧,并关闭它,如果它是假这将是一个弧线,参数五是Paint对象；


####还要理解一个paint类：
Class Overview  
The Paint class holds the style and color information about how to draw geometries, text and bitmaps.

1. paint类拥有风格和颜色信息如何绘制几何学,文本和位图。
1. Paint 代表了Canvas上的画笔、画刷、颜料等等；
1. Paint类常用方法:
1. setARGB(int a, int r, int g, int b) // 设置 Paint对象颜色，参数一为alpha透明值
1. setAlpha(int a) // 设置alpha不透明度，范围为0~255
1. setAntiAlias(boolean aa) // 是否抗锯齿
1. setColor(int color)  // 设置颜色，这里Android内部定义的有Color类包含了一些常见颜色定义
1. setTextScaleX(float scaleX)  // 设置文本缩放倍数，1.0f为原始
1. setTextSize(float textSize)  // 设置字体大小
1. setUnderlineText(booleanunderlineText)  // 设置下划线


###三、 Attr属性，在xml布局中引用以及自定义属性

**1，自定义View的属性，首先在res/values/  下建立一个attrs.xml ， 在里面定义我们的属性和声明我们的整个样式。**

	<?xml version="1.0" encoding="utf-8"?>  
	<resources>  
  
	    <attr name="titleText" format="string" />  
	    <attr name="titleTextColor" format="color" />  
	    <attr name="titleTextSize" format="dimension" />  
		<attr name="titleTextType">
	        <enum name="colorful" value="0" />
	      	<enum name="bold" value="1" />
     	</attr>
	  
	    <declare-styleable name="CustomTitleView">  
	        <attr name="titleText" />  
	        <attr name="titleTextColor" />  
	        <attr name="titleTextSize" />  
	        <attr name="titleTextType" />  
	    </declare-styleable>  
  
	</resources> 


我们定义了字体，字体颜色，字体大小和字体样式四个属性，format是值该属性的取值类型，其中字体样式我们用枚举表示。  
一共有：**string,color,demension,integer,enum,reference,float,boolean,fraction,flag.**

**2, 在布局文件中引入**

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
	    xmlns:tools="http://schemas.android.com/tools"  
	    xmlns:custom="http://schemas.android.com/apk/res/com.example.customview"  
	    android:layout_width="match_parent"  
	    android:layout_height="match_parent" >  
	  
	    <com.example.customview.view.CustomTitleView  
	        android:layout_width="200dp"  
	        android:layout_height="100dp"  
	        custom:titleText="3712"  
	        custom:titleTextColor="#ff0000"  
	        custom:titleTextSize="40sp"
	        custom:titleTextType="bold"   />  
	  
	</RelativeLayout>

在xml布局应用中要加入xmlns:custom="http://schemas.android.com/apk/res/com.example.customview"，我们的命名空间，后面的包路径指的是项目的package。  
或则是加入xmlns:custom="http://schemas.android.com/apk/res-auto"

**3，在View的构造函数中加载自定义的属性**

	public CustomTitleView(Context context, AttributeSet attrs) {
		super(context, attrs);
		TypedArray mTypedArray = context.obtainStyledAttributes(attrs, R.styleable.CustomTitleView);
		String titleText = mTypedArray.getString(R.styleable.CustomTitleView_titleText);
		int textColor = mTypedArray.getColor(R.styleable.CustomTitleView_titleTextColor, Color.GRAY);
		int textSize = mTypedArray.getDimension(R.styleable.CustomTitleView_titleTextSize, 16);
		int textType = mTypedArray.getInt(R.styleable.CustomTitleView_titleTextType, 0);
		mTypedArray.recycle();
	}

在xml布局文件中的View创建时会调用View的第二个构造函数，枚举类型我们通过getInt()来获取。


###四、 一些常用的小技巧

**1，让文字居中显示**

	Rect mBound = new Rect();  
    mPaint.getTextBounds(titleText, 0, titleText.length(), mBound);		// 测量，得到一个包裹着Text的Rect矩阵对象
	canvas.drawText(titleText, getWidth() / 2 - mBound.width() / 2, getHeight() / 2 + mBound.height() / 2, mPaint);


**2，加载Drawable图片**

	Drawable mThumb = getResources().getDrawable(R.drawable.btn_fun_model_brightness);
    int thumbHalfheight = mThumb.getIntrinsicHeight() / 2;  
    int thumbHalfWidth = mThumb.getIntrinsicWidth() / 2;  
    mThumb.setBounds(-thumbHalfWidth, -thumbHalfheight, thumbHalfWidth, thumbHalfheight);   
	
	// 在onDraw()方法中绘制Drawable
	canvas.save();  
	canvas.translate(x, y);  
	mThumb.draw(canvas);  
	canvas.restore();



####参考资料：   

1. View的生命周期： [http://www.cnblogs.com/manbu/p/3583985.html?utm_source=tuicool&utm_medium=referral](http://www.cnblogs.com/manbu/p/3583985.html?utm_source=tuicool&utm_medium=referral)
1. Android利用canvas画各种图形： [http://luoshui.blog.51cto.com/3683334/1575316](http://luoshui.blog.51cto.com/3683334/1575316)
1. Android自定义View： [http://blog.csdn.net/lmj623565791/article/details/24252901](http://blog.csdn.net/lmj623565791/article/details/24252901)
