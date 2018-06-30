---
layout: post
title: "Fragment 小结和使用技巧"
modified: 2018-6-30 09:30:00
excerpt: "实现侧滑退出的 SwipeBackFragment，优化过渡绘制..."
tags: [Fragment]
comments: true
---


主体功能基本完成后开始优化项目。打开【开发者选项】中的【调试 GPU 过渡绘制】后，惊奇的发现自己的应用全部是红色的警告。简单的调试后找到了如下几个原因：

1. BaseFragment 中设置了背景色，几乎所有的页面都继承了它
2. ImageView 同时设置了 background 和 src 属性
3. ListView 的 ItemView 默认情况下有背景色
4. 打开新的页面时，只使用了 Fragment add()，而没有调用 hide() 隐藏之前的
5. 给文字下的背景图加的遮罩直接用的一个透明灰色 View，而不是对图片处理
6. 没有去除 Activity 的默认背景：`getWindow().setBackgroundDrawable(null)`

处理完这些后页面的警告全部消失。但是处理完第 4 条 hide() 之前的 Fragment 后，又导致如下两个问题：

1. 开启新的页面，即打开 Fragment 时，当前页面先隐藏，显示空白
2. 侧滑 Fragment 退出时，上一个 Fragment 还处于隐藏状态，显示空白

#### 这里进入正题，带着疑问研究 Fragment

##### 一、问：调用 add() 方法打开新的 Fragment，为什么绘制层会叠加

答：Fragment 可以简单的理解为 **有生命周期的 View**，add() 方法是在 ViewGroup 的方法添加了一层 View，这里的重叠就导致了过渡绘制。

处理办法就是调用 hide() 来隐藏旧的 Fragment，下次展示时在调用 show() 方法显示。在源码 `FragmentManager` 类中可以看到使用 `f.mView.setVisibility(View.GONE)` 来隐藏 Fragment。

##### 二、问：FragmentTransaction 是什么，设置的动画怎么执行的

答：实现类是 BackStackRecord，管理记录一组操作，有 ArrayList\<Op\> 属性。`beginTransaction()` 方法创建并返回该实例。


#### 回到解决 hide() 导致的问题

1. 开启新页面时，给当前页面设置 300ms 的延时隐藏动画
2. 在侧滑触发时，设置上个页面的 Fragment 显示



示例代码如下：

SwipeFragment

```java
public class SwipeBackFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View rootView = inflater.inflate(getLayoutId(), null);
        ...
        return attachSwipe(rootView);
    }

    protected View attachSwipe(View rootView) {
        SwipeBackLayout swipeBackLayout = new SwipeBackLayout(getActivity());
        swipeBackLayout.addView(rootView);
        swipeBackLayout.setContentView(rootView);
        swipeBackLayout.addSwipeListener(new SwipeBackLayout.SwipeListenerEx() {

            private boolean hasSwipeBack = false;
            @Override
            public void onContentViewSwipedBack() {
                if (getActivity() != null) {
                    if (hasSwipeBack) {
                        return;
                    }
                    hasSwipeBack = true;
                    getActivity().getSupportFragmentManager().popBackStack();
                }
            }

            @Override
            public void onScrollStateChange(int state, float scrollPercent) {
            }

            @Override
            public void onEdgeTouch(int edgeFlag) {
                if (getActivity() instanceof XXXActivity) {
                    (((XXXActivity) getActivity()).getFmxosActivityHelper()).showLastFragment();
                }
            }

            @Override
            public void onScrollOverThreshold() {
            }
        });
        return swipeBackLayout;
    }
}
```

父 Activity

```java
public class FmxosActivity extends FragmentActivity {

    private Stack<Fragment> fragmentStack = new Stack<>();

    @Override
    public void onBackPressed() {
        if (getFragmentManager().getBackStackEntryCount() > 0) {
            getFragmentManager().popBackStack();
            fragmentStack.pop();
            return;
        }
        super.onBackPressed();
    }

    @Override
    public void startFragment(Fragment fragment){
        final int inID = R.anim.fmxos_slide_in_from_right;
        final int outID = R.anim.fmxos_slide_out_to_right;
        FragmentTransaction transaction = getFragmentManager().beginTransaction();
        transaction.setCustomAnimations(inID, R.anim.fmxos_open_fragment_cache, 0, outID)
                .add(R.id.layout_fragment_music_root, fragment)
                .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                .addToBackStack("fmxosMusic");
        transaction.hide(lastFragment);
        transaction.commitAllowingStateLoss();
        fragmentStack.add(fragment);
    }
    
    @Override
    public void showLastFragment() {
        Fragment fragment = fragmentStack.get(fragmentStack.size() - 2);
        if (fragment.isHidden()) {
            getFragmentManager().beginTransaction().show(fragment).commitAllowingStateLoss();
        }
    }
}
```

动画布局：

```xml
anim/fmxos_slide_in_from_right
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:fromXDelta="100.0%p"
    android:interpolator="@android:anim/decelerate_interpolator"
    android:toXDelta="0.0" />

anim/fmxos_slide_out_to_right
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:fromXDelta="0.0"
    android:interpolator="@android:anim/decelerate_interpolator"
    android:toXDelta="100.0%p" />

anim/fmxos_open_fragment_cache
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:fromAlpha="100"
    android:interpolator="@android:anim/decelerate_interpolator"
    android:toAlpha="90" />
```

注：`SwipeBackLayout` 来源 <https://github.com/ikew0ng/SwipeBackLayout>


#### 补充

> 在【调试 GPU 过渡绘制】模式下查看微信，底层的主页面并没有出现红色的过渡渲染。可以猜测背景使用的是上个页面的截图。
> 
> <img src="http://www.ionesmile.com/images/android/screenshot_20180625_wechat_fragment_render.jpg"/>


