---
layout: post
title: "Android 网络请求框架对比分析"
modified: 2017-2-22 8:12:00
excerpt: "Android 网络请求框架对比分析..."
tags: [Android]
comments: true
---

## Android 网络请求框架对比分析   

> 这两天都在调研 Android 平台上的网络请求框架，由于框架太多只抽取了网上公认比较好的几个框架进行了对比。主要分析框架结构，请求速度和适用性等方面。

#### 一、网络框架的基本结构   

在分析其它框架之前，先做一个简易的网络请求框架。这里的结构和 Volley 一致，主要分为 4 部分，如图：   

<img src="http://www.ionesmile.com/images/android/simple_net_framework.jpeg"/>  

- 第一部分：各种**请求**类型，如 JsonRequest、StringRequest 等。   
- 第二部分：**消息队列**，消息队列维护了提交给网络框架的请求列表，并且根据相应的规则进行排序。   
- 第三部分：**Executor**，也就是网络的执行者。该 Executor 继承自 Thread，在 run 方法中循环访问第二部分的请求队列，请求完成之后将结果投递给UI线程。      
- 第四部分：**Response**投递类，在第三部分的 Executor 中执行网络请求，Executor 是 Thread，但是我们并不能在主线程中更新UI，因此我们使用 ResponseDelivery 来封装 Response 的投递，保证Response执行在UI线程。   


#### 二、几个高 star 框架特征   

##### 1，[okhttp](https://github.com/square/okhttp)   

- OkHttp 是一个高效的 HTTP 库，它实现了几乎和java.net.HttpURLConnection一样的API：

	* Http2.0 的支持，共享同一个Socket来处理同一个服务器的所有请求   
	* 如果 Http2.0 不可用，通过连接池来减少请求延迟   
	* 支持传输 GZIP 来减少数据流量     
	* 缓存 Response，避免相同的请求   

- 框架图：   

	<img src="http://www.ionesmile.com/images/android/okhttp_construct.png"/>   

- 主要是通过 **Diapatcher** 不断从RequestQueue中取出请求（Call），根据是否已缓存调用 **Cache** 或 **Network** 这两类数据获取接口之一，从内存缓存或是服务器取得请求的数据。该引擎有同步和异步请求，同步请求通过 **Call.execute()** 直接返 回当前的 **Response** ，而异步请求会把当前的请求Call.enqueue添加（AsyncCall）到请求队列中，并通过回调（Callback） 的方式来获取最后结果。


##### 2，[retrofit](https://github.com/square/retrofit)   

Retrofit 是一个类型安全的 REST 客户端，在 Retrofit2 依赖了 OkHttp。在使用上拥有如下优点：   

1. 可以利用接口，方法和注解参数（parameter annotations）来声明式定义一个请求应该如何被创建。   
2. 支持序列化机制（JSON/XML 协议）
3. 在一个类型中的同步和异步请求   

Retrofit2 发布会链接：<https://realm.io/cn/news/droidcon-jake-wharton-simple-http-retrofit-2>

##### 3，[volley](https://github.com/mcxiaoke/android-volley)   

Volley 是 Google 在 2013 年推出的 Android 异步网络请求框架和图片加载框架。   

- Volley 的主要特点：    
	* 扩展性强。Volley 中大多是基于接口的设计，可配置性强。  
	* 一定程度符合 Http 规范，包括返回 ResponseCode(2xx、3xx、4xx、5xx）的处理，请求头的处理，缓存机制的支持等。并支持重试及优先级定义。   
	* 默认 Android2.3 及以上基于 HttpURLConnection，2.3 以下基于 HttpClient 实现，这两者的区别及优劣在4.2.1 Volley中具体介绍。   
	* 提供简便的图片加载工具。

- 总体设计图    

	<img src="http://www.ionesmile.com/images/android/volley_construction.png"/>

* 通过两种Dispatch Thread不断从RequestQueue中取出请求，根据是否已缓存调用Cache或Network这两类数据获取接口之一，从内存缓存或是服务器取得请求的数据，然后交由ResponseDelivery去做结果分发及回调处理。

#### 三、Retrofit 的基本使用   

官方文档：<http://square.github.io/retrofit/>   

从 GitHub 的 star 排名来看，Retrofit 数量是最多的，其次是 Okhttp，并且 Retrofit 的网络请求框架也是依赖 OKhttp 的，在网络性能上相当不错；又因为 Retrofit 通过接口注解的方式定义 Http Api，可以很方便的管理和调用；另外 GitHub 上还有一些基于 Okhttp 框架拓展的新框架（如 OkGo），使该框架也具有较强的拓展性。最终决定采用 Retrofit 作为现阶段我们应用网络请求的主流框架。   

##### 1，基本配置   

请求的 Http 接口需要定义在 Java 接口中，如下：

	public interface GitHubService {
		@GET("users/{user}/repos")
		Call<List<Repo>> listRepos(@Path("user") String user);
	}

用 Retrofit 创建一个实现了 GitHubService 接口的对象：   

	Retrofit retrofit = new Retrofit.Builder()
	    .baseUrl("https://api.github.com/")
	    .build();
	
	GitHubService service = retrofit.create(GitHubService.class);

最后调用接口中的方法来请求服务器，并返回一个 Call 对象：   

	Call<List<Repo>> repos = service.listRepos("octocat");


##### 2，Get 请求

**@Path**   
在替换 Http 接口中的路径时，用 @path 属性。如下：

	@GET("group/{id}/users")
	Call<List<User>> groupList(@Path("id") int groupId);

**@Query**   
在替换 Http 接口中的参数时，用 @Query 属性。如下：  

	@GET("group/123456/users")
	Call<List<User>> groupList(@Query("sort") String sort);

等同于： group/123456/users?sort=name    

**@QueryMap**   
在请求参数过多时，可以通过键值对的形式，如下：   

	@GET("group/123456/users")
	Call<List<User>> groupList(@QueryMap Map<String, String> options);


##### 3，Post 请求

**@Body**   
可以把一个对象作为请求内容传递，用法如下：

	@POST("users/new")
	Call<User> createUser(@Body User user);

**@Field**   
在方法的参数中添加 @Field 注解，并在方法上加上 @FormUrlEncoded 注解，代码如下：

	@FormUrlEncoded
	@POST("user/edit")
	Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);

**@Part**   
多部分请求在方法上添加 @Multipart，并在参数前加入 @Part 属性，代码如下：

	@Multipart
	@PUT("user/photo")
	Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);

##### 4，请求头修改   

**@Headers**   
请求头信息通过 @Headers 注释设置，代码如下：

	@Headers("Cache-Control: max-age=640000")
	@GET("widget/list")
	Call<List<Widget>> widgetList();

	// 多条参数
	@Headers({
	    "Accept: application/vnd.github.v3.full+json",
	    "User-Agent: Retrofit-Sample-App"
	})
	@GET("users/{username}")
	Call<User> getUser(@Path("username") String username);

另外，可以通过注解动态的使用，如下：

	@GET("user")
	Call<User> getUser(@Header("Authorization") String authorization)



- 参考网站：    
	* Okhttp 源码分析：   
	　　<http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0326/2643.html>       
	　　
	* Volley 源码解析：    
	　　<http://a.codekk.com/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90>   

	* 教你写Android网络框架之基本架构   
	　　<http://blog.csdn.net/bboyfeiyu/article/details/42740443/>

