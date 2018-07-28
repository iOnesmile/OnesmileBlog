---
layout: post
title: "Android 播放器通知栏样式适配"
modified: 2017-7-31 19:53:00
excerpt: "根据系统主题，适配不同手机的通知栏..."
tags: [Notification]
comments: true
---

### 一、获取通知栏主题颜色

由于调用系统的属性，获取颜色在某些手机上是不兼容的。因此采用先创建一个系统通知栏对象，然后迭代其中的 View 获取对应的颜色。代码如下：

```java
import android.app.Notification;
import android.content.Context;
import android.support.v4.app.NotificationCompat;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by iOnesmile on 06/07/2017.
 */
public class NotificationColor {

    private static final String NOTIFICATION_TITLE = "notification_title";
    public static final int INVALID_COLOR = -1; // 无效颜色
    private static int notificationTitleColor = INVALID_COLOR; // 获取到的颜色缓存

    /**
     * 获取系统通知栏主标题颜色，根据Activity继承自AppCompatActivity或FragmentActivity采取不同策略。
     *
     * @param context 上下文环境
     * @return 系统主标题颜色
     */
    public static int getNotificationColor(Context context) {
        try {
            if (notificationTitleColor == INVALID_COLOR) {
                if (context instanceof AppCompatActivity) {
                    notificationTitleColor = getNotificationColorCompat(context);
                } else {
                    notificationTitleColor = getNotificationColorInternal(context);
                }
            }
        } catch (Exception ignored) {
        }
        return notificationTitleColor;
    }

    /**
     * 通过一个空的Notification拿到Notification.contentView，通过{@link RemoteViews#apply(Context, ViewGroup)}方法返回通知栏消息根布局实例。
     *
     * @param context 上下文
     * @return 系统主标题颜色
     */
    private static int getNotificationColorInternal(Context context) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context);
        builder.setContentTitle(NOTIFICATION_TITLE);
        Notification notification = builder.build();
        try {
            ViewGroup root = (ViewGroup) notification.contentView.apply(context, new FrameLayout(context));
            TextView titleView = (TextView) root.findViewById(android.R.id.title);
            if (null == titleView) {
                iteratorView(root, new Filter() {
                    @Override
                    public void filter(View view) {
                        if (view instanceof TextView) {
                            TextView textView = (TextView) view;
                            if (NOTIFICATION_TITLE.equals(textView.getText().toString())) {
                                notificationTitleColor = textView.getCurrentTextColor();
                            }
                        }
                    }
                });
                return notificationTitleColor;
            } else {
                return titleView.getCurrentTextColor();
            }
        } catch (Exception e) {
            Log.e("NotificationColor", "", e);
            return getNotificationColorCompat(context);
        }
    }

    /**
     * 使用getNotificationColorInternal()方法，Activity不能继承自AppCompatActivity（实测5.0以下机型可以，5.0及以上机型不行），
     * 大致的原因是默认通知布局文件中的ImageView（largeIcon和smallIcon）被替换成了AppCompatImageView，
     * 而在5.0及以上系统中，AppCompatImageView的setBackgroundResource(int)未被标记为RemotableViewMethod，导致apply时抛异常。
     *
     * @param context 上下文
     * @return 系统主标题颜色
     */
    private static int getNotificationColorCompat(Context context) {
        try {

            NotificationCompat.Builder builder = new NotificationCompat.Builder(context);
            Notification notification = builder.build();
            int layoutId = notification.contentView.getLayoutId();
            ViewGroup root = (ViewGroup) LayoutInflater.from(context).inflate(layoutId, null);
            TextView titleView = (TextView) root.findViewById(android.R.id.title);
            if (null == titleView) {
                return getTitleColorIteratorCompat(root);
            } else {
                return titleView.getCurrentTextColor();
            }
        } catch (Exception e) {
        }
        return INVALID_COLOR;
    }

    private static void iteratorView(View view, Filter filter) {
        if (view == null || filter == null) {
            return;
        }
        filter.filter(view);
        if (view instanceof ViewGroup) {
            ViewGroup viewGroup = (ViewGroup) view;
            for (int i = 0; i < viewGroup.getChildCount(); i++) {
                View child = viewGroup.getChildAt(i);
                iteratorView(child, filter);
            }
        }
    }

    private static int getTitleColorIteratorCompat(View view) {
        if (view == null) {
            return INVALID_COLOR;
        }
        List<TextView> textViews = getAllTextViews(view);
        int maxTextSizeIndex = findMaxTextSizeIndex(textViews);
        if (maxTextSizeIndex != Integer.MIN_VALUE) {
            return textViews.get(maxTextSizeIndex).getCurrentTextColor();
        }
        return INVALID_COLOR;
    }

    private static int findMaxTextSizeIndex(List<TextView> textViews) {
        float max = Integer.MIN_VALUE;
        int maxIndex = Integer.MIN_VALUE;
        int index = 0;
        for (TextView textView : textViews) {
            if (max < textView.getTextSize()) {
                // 找到字号最大的字体，默认把它设置为主标题字号大小
                max = textView.getTextSize();
                maxIndex = index;
            }
            index++;
        }
        return maxIndex;
    }

    /**
     * 实现遍历View树中的TextView，返回包含TextView的集合。
     *
     * @param root 根节点
     * @return 包含TextView的集合
     */
    private static List<TextView> getAllTextViews(View root) {
        final List<TextView> textViews = new ArrayList<>();
        iteratorView(root, new Filter() {
            @Override
            public void filter(View view) {
                if (view instanceof TextView) {
                    textViews.add((TextView) view);
                }
            }
        });
        return textViews;
    }

    private interface Filter {
        void filter(View view);
    }
}
```

### 二、渲染播放图标

播放器图标的渲染，采用 `v4` 包中的 `tint` 方法，代码如下：

```java
public static Bitmap getBitmapByIdAndRender(Context context, int drawableResId, int renderColor) {
    Drawable drawable = getDrawable(context, drawableResId);
    drawable = tintDrawable(drawable, ColorStateList.valueOf(renderColor));
    return drawableToBitmap(drawable);
}

public static Drawable getDrawable(Context context, int imageRes) {
    Drawable drawable = context.getResources().getDrawable(imageRes);
    drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
    return drawable;
}

public static Drawable tintDrawable(Drawable drawable, ColorStateList colors) {
    final Drawable wrappedDrawable = DrawableCompat.wrap(drawable);
    DrawableCompat.setTintList(wrappedDrawable, colors);
    return wrappedDrawable;
}
```

在设置副标题的颜色时，我采用了给主标题设置一个透明度的方式来达到，通过 HSV 模型把颜色和透明度合成一个新的色值：

```java
/**
 * 把RGB + Alpha合成为一个新的RGB
 * @param alpha
 * @param color
 * @return
 */
public static int getCompoundColor(int alpha, int color) {
	float[] hsv = new float[]{0, 0, 1};
	Color.colorToHSV(color, hsv);
	hsv[2] = (alpha + 0.0f) / 0xFF;
	color = Color.HSVToColor(hsv);
	return color;
}
```

#### === 补充 `将 ARGB 转成 RGB 色值`

上面的转换方式使用时发现有 BUG，变写了一个简单的转换函数。具体是添加一个背景色，新的色值为 `背景色 * （1-透明度） + 主色 * 透明度`，代码如下：

```java
/**
 * 计算 ARGB + 背景色 的混合 RGB 色
 * @param color     原始色
 * @param bgColor   背景色
 * @param alpha     透明度
 * @return
 */
public static int convertColorAlpha(int color, int bgColor, int alpha) {
    int[] colorArr = resolveColor(color);
    int[] bgColorArr = resolveColor(bgColor);

    float alphaPercent = alpha / 255f;
    int[] colors = new int[4];
    colors[0] = 0xFF;
    for (int i = 1; i < 4; i++) {
        colors[i] = (int) (bgColorArr[i]*(1-alphaPercent) + colorArr[i]*alphaPercent);
    }
    return argb(colors[0], colors[1], colors[2], colors[3]);
}

private static int[] resolveColor(int color) {
    int[] colors = new int[4];
    colors[0] = color >>> 24;
    colors[1] = (color >> 16) & 0xFF;
    colors[2] = (color >> 8) & 0xFF;
    colors[3] = color & 0xFF;
    return colors;
}

public static int argb(int alpha, int red, int green, int blue) {
    return (alpha << 24) | (red << 16) | (green << 8) | blue;
}
```



### 三、参考链接

Android自定义通知样式适配   <http://www.jianshu.com/p/426d85f34561>


Android通知栏介绍与适配总结  <http://iluhcm.com/2017/03/12/experience-of-adapting-to-android-notifications/>

