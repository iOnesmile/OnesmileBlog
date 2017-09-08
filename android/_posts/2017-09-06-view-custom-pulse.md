---
layout: post
title: "Android 自定义带呼吸效果的 View"
modified: 2017-9-6 18:30:00
excerpt: "围绕圆绘制文字，自定义插值器实现类似于呼吸的波形..."
tags: [View]
comments: true
---

啥也别说，先上效果图。

<img src="http://www.ionesmile.com/images/android/speakview.gif"/>


> 分析效果图，主要涉及两部分：   
> 1，围绕圆圈绘制的文字   
> 2，动画（背景的呼吸动效，文字的旋转动效）


### 一、围绕圆圈绘制的文字

在 Canvas 中有一个绘制方法，

```java
canvas.drawTextOnPath(String text, Path path, float hOffset,
        float vOffset, Paint paint)
```

另外要考虑的因素是如何旋转文字：

- `hOffset` 属性可以水平方向移动文字
- `vOffset` 属性垂直方向移动文字

让文字环绕圆绘制，定义一个 Path，并添加 Circle 形路径

```java
Path path = new Path();
path.addCircle(float x, float y, float radius, Direction dir);
```

最后一个属性 `Direction`，有两种类型：`CW`、`CCW`，效果如下：

<img src="http://www.ionesmile.com/images/android/circle_direction_text.png"/>

- 设置为 `Path.Direction.CW` 时，文字沿顺时针绘制；   
- 设置为 `Path.Direction.CCW` 时，文字沿逆时针绘制。

默认文字是从 0 角度开始绘制，那么 **如何让文字的起始角度偏移** ？有两种方式：

1，设置 `hOffset`，`偏移的距离 = angle /  360f *  圆形周长`。
但是有一个问题：当偏移角度 + 文字长度 >  360° 时，**文字显示不全**，最后放弃了这种做法。

2，设置 `Path` 的 `Matrix` 属性，通过旋转来控制偏移

```java
matrix.setRotate(float degrees, float px, float py);
```

1. 如果不设置 px、py，则使用默认 （0，0）作为中心点旋转

2. 设置时有 `pre`、`set`、`post` 三种形式。原因是矩阵乘法不满足乘法交换律，因此左乘还是右乘最终的效果都不一样。我们可以把Matrix变换想象成一个队列，队列里面包含了若干个变换操作，队列中每个操作按照先后顺序操作变换目标完成变换，`pre` 相当于 **向队首增加** 一个操作，`post` 相当于 **向队尾增加** 一个操作，`set` 相当于清空当前队列 **重新设置**。

这里已经把围绕圆圈绘制文字的部分说完了。接下来来分析下动画的部分。

### 二、动画

1. ##### 使用系统的 `ValueAnimator` 来执行动画刷新操作

2. ##### 在动画更新时，达到动画效果

	- 修改文字的开始绘制角度
	- 修改背景圆的半径

3. ##### 研究效果图一个动画周期的波形
	
	1. **文字旋转波形**，研究发现大概是这样的

		![文字旋转波形](http://www.ionesmile.com/images/android/circle_around_text_wave.png)
		
		从 0 到一半周期为线性递增到最大值，后一半再从最大值线性递减到 0。反正实现起来很容易，略过。

	2. **背景的动画**，分为内圆和外圆
	
		通过对 GIF 图片的帧分析，发现内圆和外圆的波形并不一致，最终波形研究如下。

		![背景的动画波形](http://www.ionesmile.com/images/android/circle_background_wave.png)

		如上图所示，`红线` 代表 **内圆** 半径变化的波形，`蓝线` 代表 **外圆** 半径变化的波形。

		对于这样的波形变化，可以用控制波形的插值器（`Interpolator`），不过 Android 中自带的几种插值器波形与我们的并不相符。根据如上图的波形变化情况，我们自定义一个可定制的插值器。

		根据波形的变化，我们定义如下几种类型：
		
		1. 从 A 点递减到 B 点（结合效果图观察，这里定义线性变化就可以了），用 `Decline` 表示
		2. 从 B 点保持到 C 点，因为此时并不需要显示，我们用 `Lose` 表示
		3. 从 C 点递增到 D 点，用 `Rise` 表示
		4. 从 D 点递减到 E 点，用 Decline
		5. 从 E 点不显示到 F 点，用 Lose
		6. 从 F 点递增到 G 点，用 Rise
		7. 从 G 点保持到 H 点，用 `Keep` 表示

		如此，可以定义一个 Wave  对象，包含如上几种变化的过程，通过例如下面的方式设置波形的变化规则，最后调用 `float getInterpolation(float input)` 的方式来获取当前值。模拟代码如下：

		```java
		new Wave(A).declineTo(B).loseTo(C).riseTo(D).decline(E).loseTo(F).riseTo(G).keepTo(H);
		```

		通过如上方式自定义，让背景圆按照自定义的波形收缩。
		
		
### 三、代码如下

```java
import android.animation.ValueAnimator;
import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Matrix;
import android.graphics.Paint;
import android.graphics.Path;
import android.support.annotation.Nullable;
import android.util.AttributeSet;
import android.view.View;
import android.view.animation.Interpolator;
import android.view.animation.LinearInterpolator;

import com.ionesmile.test.common.utils.WaveInterpolator;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by ionesmile on 05/09/2017.
 */

public class SpeakView extends View {

    private static final String TAG = SpeakView.class.getSimpleName();
    private SpeakModel speakModel;
    private SpeakDraw speakDraw;
    private SpeakCalc speakCalc;
    private ViewAnimation viewAnimation;

    public SpeakView(Context context) {
        super(context);
        initBase(context);
    }

    public SpeakView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initBase(context);
    }

    public SpeakView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initBase(context);
    }

    private void initBase(Context context) {
        speakModel = new SpeakModel(context);
        speakDraw = new SpeakDraw(speakModel);
        speakCalc = new SpeakCalc(speakDraw, speakModel);
        ViewAnimation.SimpleAnimationListener animationListener = new ViewAnimation.SimpleAnimationListener() {
            @Override
            public void clearAnimation() {
                SpeakView.this.clearAnimation();
            }

            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                float value = (float) valueAnimator.getAnimatedValue();

                // set text start draw angle
                float textAngle = Math.abs(0.5f - value) * 2;
                textAngle = speakCalc.getTextInterpolator(textAngle) * 60;
                for (TextCircle textCircle : speakModel.textCircleList) {
                    textCircle.drawTextCurrentAngle = textCircle.drawTextStartAngle + textAngle;
                }

                // set background circle anim radius
                for (BackgroundCircle backgroundCircle : speakModel.backgroundCircleList) {
                    float realValue = backgroundCircle.interpolator.getInterpolation(value);
                    if (realValue < 0){
                        backgroundCircle.currentRadius = 0;
                    } else {
                        backgroundCircle.currentRadius = (backgroundCircle.maxRadius - backgroundCircle.minRadius) * realValue + backgroundCircle.minRadius;
                    }
                }

                invalidate();
            }
        };
        viewAnimation = new ViewAnimation(animationListener);
    }

    @Override
    protected void onDraw(Canvas canvas) {

        drawBackgroundCircle(canvas);

        drawTextCircle(canvas);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        speakCalc.onSizeChange(w, h);
    }


    private void drawBackgroundCircle(Canvas canvas) {
        for (BackgroundCircle backgroundCircle : speakDraw.speakModel.backgroundCircleList) {
            // draw background circle
            if (backgroundCircle.currentRadius > 0)
                canvas.drawCircle(speakCalc.centerX, speakCalc.centerY, backgroundCircle.currentRadius, speakDraw.getBackgroundPaint(backgroundCircle));
        }
    }

    private void drawTextCircle(Canvas canvas) {
        boolean isLockWise = true;
        for (TextCircle textCircle : speakDraw.speakModel.textCircleList) {

            // draw circle ring
            if (textCircle.needDrawCircle) {
                canvas.drawCircle(speakCalc.centerX, speakCalc.centerY, textCircle.circle.radius, speakDraw.getCirclePaint(textCircle.circle));
            }

            if (textCircle.textList == null || textCircle.textList.isEmpty()){
                continue;
            }

            float hOffset = 0;

            // draw circle text
            for (Text text : textCircle.textList) {

                Paint textPaint = speakDraw.getTextPaint(text);

                canvas.drawTextOnPath(text.text, speakDraw.getPath(textCircle, isLockWise), hOffset, textCircle.offsetVertical, textPaint);

                hOffset += (textPaint.measureText(text.text) + speakDraw.getTextSpace());
            }

            isLockWise = !isLockWise;
        }
    }

    public void startAnim(){
        viewAnimation.startViewAnim(0, 1, 3600);
    }

    public void stopAnim(){
        viewAnimation.stopAnim();
    }

    class SpeakCalc {

        private final SpeakDraw speakDraw;
        private final SpeakModel speakModel;

        private Interpolator textInterpolator;

        int centerX, centerY;

        SpeakCalc(SpeakDraw speakDraw, SpeakModel speakModel){
            this.speakDraw = speakDraw;
            this.speakModel = speakModel;

            textInterpolator = new LinearInterpolator();
        }

        void onSizeChange(int viewWidth, int viewHeight) {
            centerX = viewWidth / 2;
            centerY = viewHeight / 2;
        }

        float getTextInterpolator(float value){
            return textInterpolator.getInterpolation(value);
        }
    }

    class SpeakDraw {

        private final SpeakModel speakModel;
        Paint circlePaint;
        Paint textPaint;
        Paint backgroundPaint;

        Path path = new Path();
        Matrix matrix = new Matrix();
        private int textSpace;

        SpeakDraw(SpeakModel speakModel){
            this.speakModel = speakModel;

            circlePaint = new Paint();
            circlePaint.setStyle(Paint.Style.STROKE);

            textPaint = new Paint();
            textSpace = dp2px(30, getContext());

            backgroundPaint = new Paint();
            backgroundPaint.setStyle(Paint.Style.FILL);
        }

        public Paint getCirclePaint(Circle circle){
            circlePaint.setColor(circle.lineColor);
            circlePaint.setStrokeWidth(circle.lineWidth);
            return circlePaint;
        }

        public Paint getTextPaint(Text text){
            textPaint.setTextSize(dp2px(TextUtil.getTextSize(text.ratio), getContext()));
            textPaint.setColor(TextUtil.getTextColor(text.ratio));
            return textPaint;
        }

        public Paint getBackgroundPaint(BackgroundCircle backgroundCircle){
            backgroundPaint.setColor(backgroundCircle.color);
            return backgroundPaint;
        }

        public Path getPath(TextCircle textCircle, boolean isLockWise){
            path.reset();
            path.addCircle(speakCalc.centerX, speakCalc.centerY, textCircle.circle.radius, textCircle.direction);
            matrix.reset();
            matrix.setRotate(textCircle.drawTextCurrentAngle * (isLockWise ? 1 : -1), speakCalc.centerX, speakCalc.centerY);
            path.transform(matrix);
            return path;
        }

        public float getTextSpace() {
            return textSpace;
        }
    }

    class SpeakModel {
        List<TextCircle> textCircleList;
        List<BackgroundCircle> backgroundCircleList;

        SpeakModel(Context context){
            textCircleList = new ArrayList<>(5);
            textCircleList.add(buildTextCircle1(context));
            textCircleList.add(buildTextCircle2(context));
            textCircleList.add(buildTextCircle3(context));
            textCircleList.add(buildTextCircle4(context));
            textCircleList.add(buildTextCircle5(context));

            backgroundCircleList = new ArrayList<>(2);
            backgroundCircleList.add(buildBackgroundCircle1(context));
            backgroundCircleList.add(buildBackgroundCircle2(context));
        }

        private BackgroundCircle buildBackgroundCircle1(Context context) {
            BackgroundCircle backgroundCircle = new BackgroundCircle();
            backgroundCircle.maxRadius = dp2px(160, context);
            backgroundCircle.minRadius = dp2px(20, context);
            backgroundCircle.color = backgroundCircle.color & 0x33FFFFFF;
            backgroundCircle.interpolator = new WaveInterpolator(1)
                    .declineTo(9).loseTo(13).riseTo(21).declineTo(29)
                    .loseTo(32).riseTo(42).keepTo(50);
            return backgroundCircle;
        }

        private BackgroundCircle buildBackgroundCircle2(Context context) {
            BackgroundCircle backgroundCircle = new BackgroundCircle();
            backgroundCircle.maxRadius = dp2px(120, context);
            backgroundCircle.minRadius = dp2px(10, context);
            backgroundCircle.color = backgroundCircle.color & 0x88FFFFFF;
            backgroundCircle.interpolator = new WaveInterpolator(1)
                    .declineTo(6).loseTo(16).riseTo(21).declineTo(26)
                    .loseTo(34).riseTo(42).keepTo(50);
            return backgroundCircle;
        }

        private TextCircle buildTextCircle1(Context context) {
            TextCircle textCircle = new TextCircle();

            textCircle.circle = new Circle();
            textCircle.circle.radius = dp2px(60, context);
            textCircle.circle.lineWidth = dp2px(textCircle.circle.lineWidth, context);
            textCircle.circle.lineColor = textCircle.circle.lineColor & 0xFFFFFFFF;

            textCircle.setDrawTextStartAngle(-100);
            textCircle.offsetVertical = -dp2px(1, context);

            textCircle.textList = new ArrayList<>();
            textCircle.textList.add(new Text("亮一点", 1f));
            textCircle.textList.add(new Text("灯光律动", 2f));
            textCircle.textList.add(new Text("开灯", 1f));
            textCircle.textList.add(new Text("夜灯", 1f));

            return textCircle;
        }

        private TextCircle buildTextCircle2(Context context) {
            TextCircle textCircle = new TextCircle();

            textCircle.circle = new Circle();
            textCircle.circle.radius = dp2px(100, context);
            textCircle.circle.lineWidth = dp2px(textCircle.circle.lineWidth, context);
            textCircle.circle.lineColor = textCircle.circle.lineColor & 0x99FFFFFF;

            textCircle.setDrawTextStartAngle(60);
            textCircle.offsetVertical = -dp2px(1, context);

            textCircle.textList = new ArrayList<>();
            textCircle.textList.add(new Text("灯光律动", 1f));
            textCircle.textList.add(new Text("暗一点", 1.2f));
            textCircle.textList.add(new Text("关灯", 2.4f));
            textCircle.textList.add(new Text("夜灯", 1.2f));

            return textCircle;
        }

        private TextCircle buildTextCircle3(Context context) {
            TextCircle textCircle = new TextCircle();

            textCircle.circle = new Circle();
            textCircle.circle.radius = dp2px(140, context);
            textCircle.circle.lineWidth = dp2px(textCircle.circle.lineWidth, context);
            textCircle.circle.lineColor = textCircle.circle.lineColor & 0x33FFFFFF;

            textCircle.setDrawTextStartAngle(-135);
            textCircle.offsetVertical = -dp2px(1, context);

            textCircle.textList = new ArrayList<>();
            textCircle.textList.add(new Text("亮一点", 1.2f));
            textCircle.textList.add(new Text("灯光律动", 1.2f));
            textCircle.textList.add(new Text("开灯", 2.5f));
            textCircle.textList.add(new Text("夜灯", 1.2f));

            return textCircle;
        }

        private TextCircle buildTextCircle4(Context context) {
            TextCircle textCircle = new TextCircle();

            textCircle.circle = new Circle();
            textCircle.circle.radius = dp2px(140, context);
            textCircle.circle.lineWidth = dp2px(textCircle.circle.lineWidth, context);
            textCircle.circle.lineColor = textCircle.circle.lineColor & 0x33FFFFFF;

            textCircle.setDrawTextStartAngle(150);
            textCircle.direction = Path.Direction.CCW;
            textCircle.offsetVertical = dp2px(textCircle.circle.lineWidth, context);
            textCircle.needDrawCircle = false;

            textCircle.textList = new ArrayList<>();
            textCircle.textList.add(new Text("开灯", 2.5f));
            textCircle.textList.add(new Text("亮一点", 2f));
            textCircle.textList.add(new Text("灯光律动", 2f));

            return textCircle;
        }

        private TextCircle buildTextCircle5(Context context) {
            TextCircle textCircle = new TextCircle();

            textCircle.circle = new Circle();
            textCircle.circle.radius = dp2px(180, context);
            textCircle.circle.lineWidth = dp2px(textCircle.circle.lineWidth, context);
            textCircle.circle.lineColor = textCircle.circle.lineColor & 0x11FFFFFF;

            textCircle.textList = new ArrayList<>();

            return textCircle;
        }
    }

    class BackgroundCircle {
        int maxRadius = 100;
        int minRadius = 10;
        float currentRadius = 0;
        int color = 0xFF57FFFF;
        Interpolator interpolator;
    }

    class TextCircle {
        Circle circle;
        List<Text> textList;
        int drawTextStartAngle;
        float drawTextCurrentAngle;
        Path.Direction direction = Path.Direction.CW;
        int offsetVertical = 0;
        boolean needDrawCircle = true;

        public void setDrawTextStartAngle(int drawTextStartAngle) {
            this.drawTextStartAngle = drawTextStartAngle;
            this.drawTextCurrentAngle = drawTextStartAngle;
        }
    }

    class Text {
        String text;
        float ratio = 1;

        public Text(String text, float ratio) {
            this.text = text;
            this.ratio = ratio;
        }
    }

    class Circle {
        int radius;
        int lineWidth = 2;
        int lineColor = 0xFF04A9E0;
    }

    static class TextUtil {

        static final int TEXT_SIZE = 12;
        static final int TEXT_COLOR = 0xFF0097D9;

        public static int getTextSize(float ratio){
            return (int) (TEXT_SIZE * ratio);
        }

        public static int getTextColor(float ratio){
            return TEXT_COLOR;
        }
    }

    public static int dp2px(float value, Context context) {
        final float scale = context.getResources().getDisplayMetrics().densityDpi;
        return (int) (value * (scale / 160) + 0.5f);
    }
}
```

动画帮助类：

```java
import android.animation.Animator;
import android.animation.AnimatorListenerAdapter;
import android.animation.ValueAnimator;
import android.view.animation.Interpolator;
import android.view.animation.LinearInterpolator;

/**
 * Created by ionesmile on 05/09/2017.
 */

public class ViewAnimation {

    private AnimationListener animationListener;

    public ValueAnimator valueAnimator;

    public ViewAnimation(AnimationListener animationListener) {
        this.animationListener = animationListener;
    }

    public void startAnim() {
        stopAnim();
        startViewAnim(0f, 1f, 1000);
    }

    public void startAnim(int time) {
        stopAnim();
        startViewAnim(0f, 1f, time);
    }


    public void stopAnim() {
        if (valueAnimator != null) {
            animationListener.clearAnimation();

            valueAnimator.setRepeatCount(0);
            valueAnimator.cancel();
            valueAnimator.end();
            if (animationListener.onStopAnim() == 0) {
                valueAnimator.setRepeatCount(0);
                valueAnimator.cancel();
                valueAnimator.end();
            }
        }
    }

    public ValueAnimator startViewAnim(float startF, final float endF, long time) {
        valueAnimator = ValueAnimator.ofFloat(startF, endF);
        valueAnimator.setDuration(time);
        valueAnimator.setInterpolator(animationListener.getInterpolator());

        valueAnimator.setRepeatCount(animationListener.setAnimRepeatCount());

        if (ValueAnimator.RESTART == animationListener.setAnimRepeatMode()) {
            valueAnimator.setRepeatMode(ValueAnimator.RESTART);

        } else if (ValueAnimator.REVERSE == animationListener.setAnimRepeatMode()) {
            valueAnimator.setRepeatMode(ValueAnimator.REVERSE);

        }

        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                animationListener.onAnimationUpdate(valueAnimator);
            }
        });
        valueAnimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
            }

            @Override
            public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
                super.onAnimationRepeat(animation);
                animationListener.onAnimationRepeat(animation);
            }
        });
        if (!valueAnimator.isRunning()) {
            animationListener.animIsRunning();
            valueAnimator.start();

        }

        return valueAnimator;
    }


    public interface AnimationListener {

        void clearAnimation();

        void onAnimationUpdate(ValueAnimator valueAnimator);

        void onAnimationRepeat(Animator animation);

        int onStopAnim();

        int setAnimRepeatMode();

        int setAnimRepeatCount();

        void animIsRunning();

        Interpolator getInterpolator();
    }

    public abstract static class SimpleAnimationListener implements AnimationListener {

        @Override
        public void onAnimationRepeat(Animator animation) {

        }

        @Override
        public int onStopAnim() {
            return 0;
        }

        @Override
        public int setAnimRepeatMode() {
            return ValueAnimator.RESTART;
        }

        @Override
        public int setAnimRepeatCount() {
            return ValueAnimator.INFINITE;
        }

        @Override
        public void animIsRunning() {

        }

        @Override
        public Interpolator getInterpolator() {
            return new LinearInterpolator();
        }
    }
}
```

自定义的波形插值器：

```java
import android.view.animation.Interpolator;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by ionesmile on 06/09/2017.
 */

public class WaveInterpolator implements Interpolator {

    private List<Pulse> pulseList = new ArrayList<>();

    public WaveInterpolator(float startValue) {
        pulseList.add(new Pulse(Action.KEEP, startValue, 0));
    }

    public WaveInterpolator declineTo(float progress) {
        pulseList.add(new Pulse(Action.DECLINE, 0, progress));
        return this;
    }

    public WaveInterpolator riseTo(float progress) {
        pulseList.add(new Pulse(Action.RISE, 1, progress));
        return this;
    }

    public WaveInterpolator keepTo(float progress) {
        pulseList.add(new Pulse(Action.KEEP, pulseList.get(pulseList.size() - 1).value, progress));
        return this;
    }

    public WaveInterpolator loseTo(float progress) {
        pulseList.add(new Pulse(Action.LOSE, -1, progress));
        return this;
    }

    /**
     * 将输入的进度装换成对应的值
     *
     * @param input 0 ~ 1.0f
     * @return
     */
    public float getInterpolation(float input) {
        float durationProgress = pulseList.get(pulseList.size() - 1).progress - pulseList.get(0).progress;
        float progress = durationProgress * input;
        int pulseIndex = getPulseIndex(progress);
        Pulse pulse = pulseList.get(pulseIndex);
        switch (pulse.action) {
            case KEEP:
                return pulse.value;
            case LOSE:
                return -1;
            case DECLINE:
                return getDeclineValue(pulseList.get(pulseIndex - 1), pulse, progress);
            case RISE:
                return getRiseValue(pulseList.get(pulseIndex - 1), pulse, progress);
        }
        return input;
    }

    private float getDeclineValue(Pulse lastPulse, Pulse pulse, float progress) {
        progress = progress - lastPulse.progress;
        return 1 - progress / (pulse.progress - lastPulse.progress);
    }

    private float getRiseValue(Pulse lastPulse, Pulse pulse, float progress) {
        progress = progress - lastPulse.progress;
        return progress / (pulse.progress - lastPulse.progress);
    }

    private int getPulseIndex(float progress) {
        int length = pulseList.size();
        int index = 1;
        while (index < length) {
            float itemProgress = pulseList.get(index).progress;
            if (progress <= itemProgress) {
                return index;
            }
            index++;
        }
        return 0;
    }

    enum Action {
        DECLINE, RISE, KEEP, LOSE
    }

    class Pulse {
        Action action;
        float value;
        float progress;

        public Pulse(Action action, float value, float progress) {
            this.action = action;
            this.value = value;
            this.progress = progress;
        }
    }
}
```

