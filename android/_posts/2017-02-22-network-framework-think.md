---
layout: post
title: "Android 网络请求框架解惑"
modified: 2017-2-22 18:32:00
excerpt: "Android 网络请求框架学习总结..."
tags: [Android]
comments: true
---

## Android 网络请求框架解惑   

> 上一篇博客主要是对几个网络框架的结构和特点做了一个简单的列举，虽然找了很多资料东拼西凑的弄了很多时间才完成，但发现还有一些重点没有说清楚。本篇通过自问自答的方式进行再一次的总结，以及提出一些自己的想法。   

#### 1，OkHttp 到底快在哪里？

- 支持 [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) 和 [SPDY](https://en.wikipedia.org/wiki/SPDY) 协议，共享同一个Socket来处理同一个服务器的所有请求   
- 当服务器 Http/2 或 SPDY 协议不支持时，则通过连接池来减少请求延时    
- 使用透明的 GZIP 压缩来减少数据流量   
- 利用响应缓存来避免重复的网络请求   
- 网络出现问题时，可以自动从常见的连接问题当中恢复    
- 当服务器某一个 IP 地址连接失败时，会尝试其它地址   

SPDY 一种开放的网络传输协议，由Google开发，其成果最终演变为 HTTP/2 的关键功能。其目的在于降低网页的加载时间。通过优先级和**多路复用**，SPDY 使得只需要创建一个 TCP 连接即可传送网页内容及图片等资源。

#### 2，Android 的几种网络连接方式

- **Apache HttpClient** 早就被不推荐使用，并在 6.0 上被删除。Google 推荐在 Android 上使用 HttpUrlConnection。DefaultHttpClient和它的兄弟AndroidHttpClient都是HttpClient具体的实现类，它们都拥有众多的API，而且实现比较稳定，bug数量也很少。但同时也由于HttpClient的API数量过多，使得我们很难在不破坏兼容性的情况下对它进行升级和扩展。虽然高效稳定，但是维护成本高昂，故android 开发团队不愿意在维护该库而是转投更为轻便的 HttpUrlConnection。     

- **HttpURLConnection** 是一种多用途、轻量极的 HTTP 客户端，使用它来进行 HTTP 操作可以适用于大多数的应用程序。虽然 HttpURLConnection 的 API 提供的比较简单，但是同时这也使得我们可以更加容易地去使用和扩展它。在 Android 2.2 版本之前，HttpURLConnection 一直存在着一些令人厌烦的 BUG，因此 Volley 框架在 2.2 之前使用的是 HttpClient。    

- **OKHttp** 是一款高效的 HTTP 客户端，支持 SPDY(或HTTP)、连接池、GZIP 和 HTTP 缓存，使用 Okio 来大大简化数据的访问与存储。同时处理了很多网络疑难杂症，会从很多常用的连接问题中自动恢复，处理了代理服务器问题和SSL握手失败问题。Android4.4 开始 HttpURLConnection 的底层实现采用的是 OkHttp。   

#### 3，如何让自己应用与网络请求框架隔离，方便以后更换框架？   

一个好的隔离方法是不让上次应用不直接调用网络请求的接口，可以通过一个对象把网络请求的部分的代码包裹起来，对上层应用只提供要请求的接口和返回的回调。

如下对 Retrofit 做了一个简单的使用封装（参照 AsyncTask 回调的方式）：

	public abstract class ContentTask implements Callback<ResponseBody> {
	
	    private static final String TAG = ContentTask.class.getSimpleName();
	    private Context mContext;
	    private boolean isShowDialog = false;
	    private Dialog mProgressDialog;
	    private Call<ResponseBody> callTask;
	
	    public ContentTask(Context context) {
	        this.mContext = context;
	    }
	
	    /**
	     * @param context       上下文
	     * @param isShowDialog  是否显示对话框
	     */
	    public ContentTask(Context context, boolean isShowDialog) {
	        this.mContext = context;
	        this.isShowDialog = isShowDialog;
	    }
	
	    // 开始执行网络请求
	    public void exec() {
	        callTask = getCall(NetManager.getInstance().getNetHelper());
	        if (callTask != null) {
	            callTask.enqueue(this);
	
	            onHttpStart();
	        }
	    }
	
	    protected void onHttpStart() {
	        if (isShowDialog) {
	            mProgressDialog = new TaskProgressDialog(mContext);
	            mProgressDialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
	                @Override
	                public void onDismiss(DialogInterface dialogInterface) {
	                    if (callTask != null && !callTask.isCanceled()) {
	                        callTask.cancel();
	                    }
	                }
	            });
	            mProgressDialog.show();
	        }
	    }
	
	    @Override
	    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
	        cancelProgressDialog();
	        try {
	            ResponseBody body = response.body();
	            if (body != null){
	                String result = body.source().readUtf8();
	                Log.v(TAG, "onResponse() " + result);
	                onSuccessResult(result, response);
	            } else {
	                onFailure(call, new NullPointerException("onResponse().body() is Null"));
	            }
	        } catch (Exception e) {
	            onFailure(call, e);
	        }
	    }
	
	    @Override
	    public void onFailure(Call<ResponseBody> call, Throwable t) {
	        Log.w(TAG, "onFailure()", t);
	        cancelProgressDialog();
	    }
	
	    protected void cancelProgressDialog() {
	        if (mProgressDialog != null) {
	            mProgressDialog.dismiss();
	            mProgressDialog = null;
	        }
	    }
	
	    public abstract Call<ResponseBody> getCall(CloudMusicHandle handle);
	
	    protected abstract void onSuccessResult(String result, Response<ResponseBody> response);
	}

调用方式如下：

	new ContentTask(this, true){

		@Override
		public Call<ResponseBody> getCall(CloudMusicHandle handle) {
			return handle.getBannerVideo(videoID);
		}

		@Override
		protected void onSuccessResult(String result, Response<ResponseBody> response) {
			// do something...
		}
	}.exec();


接口中的代码：

	public interface CloudMusicHandle {
	
	    @GET("api/getvideo")
	    Call<ResponseBody> getBannerVideo(@Query("id") String id);
	}

Retrofit 管理类：

	public class NetManager {
	
	    private CloudMusicHandle netHelper;
	
	    public CloudMusicHandle getNetHelper(){
	        return netHelper;
	    }
	
	    public static NetManager getInstance(){
	        if (mInstance == null){
	            mInstance = new NetManager();
	        }
	        return mInstance;
	    }
	
	    private static NetManager mInstance;
	
	    private NetManager(){
	        Retrofit retrofit = new Retrofit.Builder()
	                .baseUrl("http://xxx.xxxx")
	                .build();
	        netHelper = retrofit.create(CloudMusicHandle.class);
	    }
	}

