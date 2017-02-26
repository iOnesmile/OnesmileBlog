---
layout: post
title: "Android 框架设计总结"
modified: 2017-2-25 7:10:00
excerpt: "Android 框架设计，从 MVC 到 MVP..."
tags: [Android]
comments: true
---

#### 一、MVC/MVP/MVVP 框架的理解   

- **MVC** 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。  
	<img src="http://www.ionesmile.com/images/android/mvc_structure_chart.png"/>  
	Android 中的 MVC：
	- **视图层(View)**：一般采用 XML 文件进行界面的描述，这些 XML 可以理解为 Android 应用的 View。 
	- **控制层(Controller)**：Android的控制层的重任通常落在了众多的 Activity/Fragment 的肩上。   
	- **模型层(Model)**：针对业务模型，建立的数据结构和相关的类，就可以理解为 Android 应用的Model，Model 是与 View 无关，而与业务相关。对*数据库*的操作、对*网络*和对*业务计算*等操作都应该在该层处理。    
	**MVC 模式的缺点**：    
	在Android开发中，Activity并不是一个标准的MVC模式中的Controller，同时它还负责加载布局、初始化界面和接受并处理用户的操作请求。随着界面及其逻辑的复杂度不断提升以致 Activity 变得庞大臃肿。    

- **MVP** 从更早的MVC框架演变过来，与MVC有一定的相似性：Controller/Presenter负责逻辑的处理，Model提供数据，View负责显示。在MVP模式里通常包含3个要素（加上View interface是4个）：  
 
	<img src="http://www.ionesmile.com/images/android/mvp_structure_chart.png" width=300/>  
	
	- **View：**负责绘制 UI 元素、与用户进行交互(在 Android 中体现为 Activity)   
	- **Model：**负责存储、检索、操纵数据(有时也实现一个 Model interface 用来降低耦合)   
	- **Presenter：**作为 View 与 Model 交互的中间纽带，处理与用户交互的负责逻辑   
	- **View interface：**需要 View 实现的接口，View 通过 View interface 与 Presenter 进行交互，降低耦合，方便进行单元测试   

- **MVVM** 模式将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。唯一的区别是，它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。这就使得视图和控制层之间的耦合程度进一步降低，关注点分离更为彻底，同时减轻了Activity的压力。   

	<img src="http://www.ionesmile.com/images/android/mvvm_structure_chart.png" width=200/>  

#### 二、如何构建一个 MVP 的框架   

先看我写的一个 MVP 的框架，如下图：  

<img src="http://www.ionesmile.com/images/android/mvp_demo_treelist.png"/>  

- UI 采用 DataBinding 的方式，把 View 和 Data 绑定在一起   
- 网络采用 Retrofit + RxJava 的方式实现   
- 包名用 ui、presenter 和 data 隔离  

MainActivity 代码如下：

	public class MainActivity extends BaseActivity<MainActivityBinding> implements ISearchView {
	
	    private SearchPresenter searchPresenter;
	
	    @Override
	    protected int getLayoutId() {
	        return R.layout.activity_main;
	    }
	
	    @Override
	    protected void initBase() {
	        searchPresenter = new SearchPresenter();
	        searchPresenter.onCreate();
	        searchPresenter.attachView(this);
	    }
	
	    @Override
	    protected void initUI() {
	
	    }
	
	    @Override
	    protected void initData() {
	        rootBinding.setKeyword("com.chipsguide.demo.cloudmusic");
	    }
	
	    @Override
	    protected void initListener() {
	        rootBinding.setClickListener(this);
	    }
	
	    @Override
	    protected void onDestroy() {
	        super.onDestroy();
	        searchPresenter.onStop();
	    }
	
	    @Override
	    public void onClick(View view) {
	        switch (view.getId()){
	            case R.id.btn_search:
	                searchPresenter.startSearch();
	                break;
	        }
	    }
	
	    @Override
	    public String getSearchKey() {
	        return rootBinding.getKeyword();
	    }
	
	    @Override
	    public void onSearchSuccess(String result) {
	        rootBinding.setResult(result);
	    }
	
	    @Override
	    public void onSearchFailure(String message) {
	        rootBinding.setResult(message);
	    }
	}

SearchView 代码如下：   

	public interface ISearchView extends IBaseView {
	
	    String getSearchKey();
	
	    void onSearchSuccess(String result);
	
	    void onSearchFailure(String message);
	}

SearchPresenter 代码如下：   

	public class SearchPresenter implements Presenter {
	
	    private ISearchView searchView;
	    private CompositeSubscription mCompositeSubscription;
	    private DataManager dataManager;
	    private String mResult;
	
	    @Override
	    public void onCreate() {
	        mCompositeSubscription = new CompositeSubscription();
	        dataManager = new DataManager();
	    }
	
	    @Override
	    public void onStop() {
	        if (mCompositeSubscription.hasSubscriptions()){
	            mCompositeSubscription.unsubscribe();
	        }
	    }
	
	    @Override
	    public void attachView(IBaseView baseView) {
	        this.searchView = (ISearchView) baseView;
	    }
	
	    public void startSearch(){
	        final String searchKey = searchView.getSearchKey();
	        if (TextUtils.isEmpty(searchKey)) {
	            searchView.showToast("搜索关键字不能为空！");
	            return;
	        }
	        mCompositeSubscription.add(dataManager.getSearchResult(searchKey)
	                .subscribeOn(Schedulers.io())
	                .observeOn(AndroidSchedulers.mainThread())
	                .subscribe(new Observer<ResponseBody>() {
	                    @Override
	                    public void onCompleted() {
	                        if (mResult != null) {
	                            searchView.onSearchSuccess(mResult);
	                        }
	                    }
	
	                    @Override
	                    public void onError(Throwable e) {
	                        e.printStackTrace();
	                        searchView.onSearchFailure("请求失败！！！");
	                    }
	
	                    @Override
	                    public void onNext(ResponseBody responseBody) {
	                        if (responseBody != null) {
	                            try {
	                                mResult = responseBody.source().readUtf8();
	                            } catch (IOException e) {
	                                e.printStackTrace();
	                            }
	                        }
	                    }
	                })
	        );
	    }
	}

#### 三、一些好的框架设计流程   

1. AOP 在 Android 框架中的应用   

AOP(Aspect-Oriented Programming, 面向切面编程)，诞生于上个世纪90年代，是对OOP(Object-Oriented Programming, 面向对象编程)的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当我们需要为分散的对象引入公共行为的时候，OOP则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。对于其他类型的代码，如安全性、异常处理和透明的持续性也是如此。这种散布在各处的无关的代码被称为横切（Cross-Cutting）代码，在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。而AOP技术则恰恰相反，它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。   

AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处都基本相似。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

示例：[Android AOP 框架（SAF-AOP）](https://github.com/fengzhizi715/SAF-AOP)   

#### 四、Google 推荐的框架结构蓝图   

在 [GoogleSamples](https://github.com/googlesamples/android-architecture) 的中有推荐了一些好的架构图，如下：   

- [todo‑mvp](https://github.com/googlesamples/android-architecture/tree/todo-mvp/)
- [todo‑mvp‑loaders](https://github.com/googlesamples/android-architecture/tree/todo-mvp-loaders/)
- [todo‑databinding](https://github.com/googlesamples/android-architecture/tree/todo-databinding/)
- [todo‑mvp‑clean](https://github.com/googlesamples/android-architecture/tree/todo-mvp-clean/)
- [todo‑mvp‑dagger](https://github.com/googlesamples/android-architecture/tree/todo-mvp-dagger/)
- [todo‑mvp‑contentproviders](https://github.com/googlesamples/android-architecture/tree/todo-mvp-contentproviders/)
- [todo‑mvp‑rxjava](https://github.com/googlesamples/android-architecture/tree/todo-mvp-rxjava/)   



> Demo 下载地址：<https://github.com/iOnesmile/ModularizationDemo>      
> 
> 参考网站：   
> - [Android App的设计架构：MVC,MVP,MVVM与架构经验谈](https://www.tianmaying.com/tutorial/AndroidMVC)   

