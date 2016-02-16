---
layout: post
title: "应用内存优化"
modified: 2016-2-16 17:42:00
excerpt: "优化应用内存，减少OOM的产生......"
tags: [Android, Optimize]
comments: true
---
主要是说一下我对OOM的一些优化处理的小点，有很多不足的地方，欢迎指正。  

首先大概描述一下我优化的App，在构建上它是由一个MainActivity嵌套着N多的Fragment，所有的界面跳转其实只是Fragment的变化，Activity一直没有改变，主体风格是半透明和毛玻璃效果，以及有大量的网络图片加载。  
应用一般在使用的时间过长后，就很容易出现闪退的现象，在用AS调试后，发现应用的内存占用一直只增不减，当内存超过一定范围后就会闪退了。  
好了，上面部分都是瞎扯的，可以忽略不计。

##从以下几个方面，对应用进行优化
####1，手动调用Bitmap的recycle()方法
由于Bitmap 的内存自动回收机制不是那么理想，往往要自己处理。如果是临时变量，在没有Bitmap对象时要即时调用recycle()释放，如果是Activity中的Bitmap对象，需要在onDestroy()中明确释放。

    if (mBitmap != null && !mBitmap.isRecycled()){
        mBitmap.recycle();
        mBitmap = null;
    }

####2，单例对象需要Context时，不要用Activity，而是ApplicationContext
如果用Activity的引用，即便是在Activity销毁后，由于又静态变量引用着，它是不会销毁的，从而导致内存泄露。

####3，在Activity销毁时，设置一个空的布局给界面，以便更快的清理。setContentView(R.layout.layout_null);
此处是在讨论群上问的大神，大神给的答复。感觉用上去后会有一些作用，暂时不懂原理，等以后知道了再来补上。

####4，Fragment销毁时，可以在onDestoryView()方法中，设置rootView的removeAllViews()方法，移除所有View。或设置一个新的View。
我暂时的做法是在onDestoryView()时，移除所有的View来回收内存的。暂时这么写的，还不知道副作用，切用且珍惜

        if (rootView instanceof ViewGroup){
            try{
                ((ViewGroup)rootView).removeAllViews();
            } catch (Exception e){
                if (getActivity() != null){
                    rootView = new View(getActivity());
                } else {
                    rootView = null;
                }
            }
        }

####5，在清单文件的Appliction节点上设置 android:largeHeap="true"
Android设备出厂以后，java虚拟机对单个应用的最大内存分配就确定下来了，超出这个值就会OOM。这个属性值是定义在/system/build.prop文件中的  
**dalvik.vm.heapstartsize=8m**　　　　它表示堆分配的初始大小，它会影响到整个系统对RAM的使用程度，和第一次使用应用时的流畅程度。

**dalvik.vm.heapgrowthlimit=64m**　　　　// 单个应用可用最大内存  
主要对应的是这个值,它表示单个进程内存被限定在64m,即程序运行过程中实际只能使用64m内存，超出就会报OOM。（仅仅针对dalvik堆，不包括native堆）  

**dalvik.vm.heapsize=384m**　　　　//heapsize参数表示单个进程可用的最大内存，但如果存在heapgrowthlimit参数，则以heapgrowthlimit为准.   
heapsize表示不受控情况下的极限堆，表示单个虚拟机或单个进程可用的最大内存。而android上的应用是带有独立虚拟机的，也就是每开一个应用就会打开一个独立的虚拟机（这样设计就会在单个程序崩溃的情况下不会导致整个系统的崩溃）。  
**注意**：在设置了heapgrowthlimit的情况下，单个进程可用最大内存为heapgrowthlimit值。在android开发中，如果要使用大堆，需要在manifest中指定android:largeHeap为true，这样dvm heap最大可达heapsize。   

此处参考：[http://blog.csdn.net/vshuang/article/details/39647167](http://blog.csdn.net/vshuang/article/details/39647167)

####6，Fragment切换时的优化
1，在Fragment切换时，用hide()、add()和show()方法代替replace()方法。参考[http://blog.csdn.net/jys1115/article/details/41725611?utm_source=tuicool&utm_medium=referral](http://blog.csdn.net/jys1115/article/details/41725611?utm_source=tuicool&utm_medium=referral "CSDN对Fragment切换的优化")  

之前代码  

	/** 
	 * 切换页面，这里采用回调 
	 *  
	 * @param f 
	 */  
	public void switchFragment(Fragment f) {  
	    if (f == null)  
	        return;  
	    FragmentTransaction transaction = getSupportFragmentManager()  
	            .beginTransaction();  
	    transaction.replace(R.id.fl_main, f);  
	    // transaction.addToBackStack(descString);  
	    transaction.commit();  
	
	    // 让menu回去  
	    menu.toggle();  
	}  
优化后的代码  

	/** 
	 * 切换页面的重载，优化了fragment的切换 
	 *  
	 * @param f 
	 * @param descString 
	 */  
	public void switchFragment(Fragment from, Fragment to) {  
	    if (from == null || to == null)  
	        return;  
	    FragmentTransaction transaction = getSupportFragmentManager()  
	            .beginTransaction().setCustomAnimations(R.anim.tran_pre_in,  
	                    R.anim.tran_pre_out);  
	    if (!to.isAdded()) {  
	            // 隐藏当前的fragment，add下一个到Activity中  
	            transaction.hide(from).add(R.id.fl_main, to).commit();  
	        } else {  
	            // 隐藏当前的fragment，显示下一个  
	            transaction.hide(from).show(to).commit();  
	        }  
	    // 让menu回去  
	    menu.toggle();  
	}  


2，fragment+viewpager的优化
fragment在滑动时，会不断销毁和重建fragment，这样会比较影响性能。加入rootView,缓存加载后的view，如果有就不重新加载数据。加入判断是否已经加载数据完成的标志变量，如果已经加载了数据，就不重新加载数据。  
参考：[http://blog.csdn.net/u013173289/article/details/44002371](http://blog.csdn.net/u013173289/article/details/44002371 "关于fragment+viewpager的优化")  

	/** 
	 * 提供了fragment的封装后基类，提供context给子类使用 
	 * 
	 * @author Graypn 
	 */  
	public abstract class BaseFragment extends Fragment {  
	  
	    //根部view  
	    private View rootView;  
	    protected Context context;  
	    private Boolean hasInitData = false;  
	  
	    @Override  
	    public void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        context = getActivity();  
	    }  
	  
	    @Override  
	    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
	                             Bundle savedInstanceState) {  
	        if (rootView == null) {  
	            rootView = initView(inflater);  
	        }  
	        return rootView;  
	    }  
	  
	    @Override  
	    public void onActivityCreated(Bundle savedInstanceState) {  
	        super.onActivityCreated(savedInstanceState);  
	        if (!hasInitData) {  
	            initData();  
	            hasInitData = true;  
	        }  
	    }  
	  
	    @Override  
	    public void onDestroyView() {  
	        super.onDestroyView();  
	        ((ViewGroup) rootView.getParent()).removeView(rootView);  
	    }  
	  
	    /** 
	     * 子类实现初始化View操作 
	     */  
	    protected abstract View initView(LayoutInflater inflater);  
	  
	    /** 
	     * 子类实现初始化数据操作(子类自己调用) 
	     */  
	    public abstract void initData();  
	  
	    /** 
	     * 封装从网络下载数据 
	     */  
	    protected void loadData(HttpRequest.HttpMethod method, String url,  
	                            RequestParams params, RequestCallBack<String> callback) {  
	        if (0 == NetUtils.isNetworkAvailable(getActivity())) {  
	            new CustomToast(getActivity(), "无网络，请检查网络连接！", 0).show();  
	        } else {  
	            NetUtils.loadData(method, url, params, callback);  
	        }  
	    }  
	  
	}

