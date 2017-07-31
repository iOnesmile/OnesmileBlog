---
layout: post
title: "Android 第三方推送整合文档"
modified: 2017-7-31 19:09:00
excerpt: "集成第三方推送笔记，应用未启动时点击通知栏执行的一连串打开页面操作..."
tags: [Push]
comments: true
---

### 一、集成开发文档

官方文档已经非常详细，而且更新及时，这里直接给出地址了。集成过程中可能会遇到一点小坑，在网上找找还是有的。

#### 1，小米推送

配置文档地址：<https://dev.mi.com/doc/?p=544>

> 注：开发中遇到推送多条消息只显示最后一条的问题。原因是服务器在构建消息时 `notifyId` 使用了默认值，当  `notifyId` 相同时会覆盖之前的消息。详见[《小米推送服务 Server 端 SDK 》](https://dev.mi.com/doc/?p=533#d5e351)。

#### 2，激光推送

配置文档地址：<http://docs.jiguang.cn/jpush/client/Android/android_sdk/>

#### 3，阿里云推送

配置文档地址：<https://help.aliyun.com/document_detail/51056.html?spm=5176.doc30066.6.641.pXaihr>

控制台地址：<https://push.console.aliyun.com/>

> 注：阿里云在创建应用时只需要有一个名字，待审核通过后会出现“**配置**”按钮，此时一定要进入**配置对应的包名**应用才能正常。


### 二、点击推送消息后的处理（参考）

点击推送消息后，如果需要打开一连串的 Fragment 页面，还得考虑应用是否启动，逻辑就变得繁琐起来。

首先有这么几条信息先整理一下：

- 将 Activity 的启动模式设置为 **singleTask** 后，调用开启，此时如果 Activity 已经启动会执行生命周期的 `onNewIntent(Intent intent)` 方法，如果未启动才会执行 `onCreate(Bundle savedInstanceState)` 方法。

- 在 Fragment 基础上嵌套再开启另外一个 Fragment 时，只有在第一个 Fragment 执行了 `onResume()` 方法后才可以正常的开启第二个 Fragment，否者会替换失败。（测试得出）


有如下一条逻辑链：点击通知条 ---》 开启 Activity ---》 加载 FragmentA ---》 FragmentA onResume() ---》 加载 FragmentB ---》 FragmentB onResume() ---》 加载 FragmentC，可以参照如下方式实现：

在广播接收者中，接收用户点击通知条的事件：

	// 小米推送的实现方式
	public class MiPushMusicReceiver extends PushMessageReceiver {
	
	    @Override
	    public void onNotificationMessageClicked(Context context, MiPushMessage miPushMessage) {
	        String pushContent = miPushMessage.getContent();
	        if (!TextUtils.isEmpty(pushContent)) {
	            PushMusic pushMusic = GsonHelper.fromJson(pushContent, PushMusic.class);
	            if (pushMusic != null) {
	                startMusic(context, pushMusic);
	            }
	        }
	    }
	
	    public static final String EXTRA_PUSH_MUSIC = "extraPushMusic";
	
	    private void startMusic(Context context, PushMusic pushMusic) {
	        if (isAppAlive(context, context.getPackageName())) {
	            Intent intent = new Intent(context, MainActivity.class);
	            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	            intent.putExtra(EXTRA_PUSH_MUSIC, pushMusic);
	            context.startActivity(intent);
	        } else {
	            Intent launchIntent = context.getPackageManager().
	                    getLaunchIntentForPackage("com.ionesmile.pushdemo");
	            launchIntent.setFlags(
	                    Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED);
	            launchIntent.putExtra(EXTRA_PUSH_MUSIC, pushMusic);
	            context.startActivity(launchIntent);
	        }
	    }
	
	    /**
	     * 判断应用是否已经启动
	     *
	     * @param context     一个context
	     * @param packageName 要判断应用的包名
	     * @return boolean
	     */
	    public static boolean isAppAlive(Context context, String packageName) {
	        Log.v("NotificationLaunch", "isAppAlive() packageName = " + packageName);
	        ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
	        List<ActivityManager.RunningAppProcessInfo> processInfos = activityManager.getRunningAppProcesses();
	        for (int i = 0; i < processInfos.size(); i++) {
	            if (processInfos.get(i).processName.equals(packageName)) {
	                Log.i("NotificationLaunch", String.format("the %s is running, isAppAlive return true", packageName));
	                return true;
	            }
	        }
	        Log.i("NotificationLaunch", String.format("the %s is not running, isAppAlive return false", packageName));
	        return false;
	    }
	}

在 Activity 中，处理如下，并给 Activity 添加 **SingleTask** 启动模式：

	public class MainActivity extends Activity implements ContinuousEventCallback {
	
	    private Map<Class, Runnable> classRunnableMap = new HashMap<>();
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        ...
	        startPushMusic(getIntent(), false);
	    }
	
	    @Override
	    protected void onNewIntent(Intent intent) {
	        super.onNewIntent(intent);
	        startPushMusic(intent, true);
	    }
	
	    @Override
	    public void onWindowResumeReady(Object window) {
	        Runnable runnable = classRunnableMap.remove(fragment.getClass());
	        if (runnable != null) {
	            runnable.run();
	        }
	    }
	
	    // 执行一连串的事情，当窗体在 onResume() 时做什么操作
	    private void startPushMusic(Intent intent, boolean isNewIntent) {
	        if (intent != null && intent.getExtras() != null) {
	            final MiPushMusicReceiver.PushMusic pushMusic = (MiPushMusicReceiver.PushMusic) intent.getExtras().getSerializable(MiPushMusicReceiver.EXTRA_PUSH_MUSIC);
	            if (pushMusic != null) {
	                classRunnableMap.put(FragmentA.class, new Runnable() {
	                    @Override
	                    public void run() {
	                        // TODO FragmentA something 
	                    }
	                });
	                classRunnableMap.put(FragmentB.class, new Runnable() {
	                    @Override
	                    public void run() {
	                        // TODO FragmentB something 
	                    }
	                });
	                ...
	                // 如果当前 Activity 已经存在，说明部分子 Fragment 已经执行了 onResume()，可以直接执行子操作
	                if (isNewIntent) {
	                    classRunnableMap.remove(FragmentA.class).run();
	                    classRunnableMap.remove(FragmentB.class).run();
	                }
	            }
	        }
	    }
	}

	public interface ContinuousEventCallback {
	
	    void onWindowResumeReady(Object window);
	}

另外，在 Fragment（BaseFragment 或是需要监听的 Fragment） 的 onResume() 方法中调用添加如下代码：

    @Override
    public void onResume() {
        super.onResume();
        if (getActivity() instanceof ContinuousEventCallback) {
            ((ContinuousEventCallback) getActivity()).onWindowResumeReady(this);
        }
    }
