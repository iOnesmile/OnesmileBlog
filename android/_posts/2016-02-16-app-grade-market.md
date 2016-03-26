---
layout: post
title: "跳转到应用市场评分"
modified: 2016-2-16 16:46:00
excerpt: "跳转到应用市场，对当前应用评分......"
tags: [Android]
comments: true
---
###一、简单的介绍###
通过一个意图跳转到手机市场，如果有多个应用市场，系统会显示一个对话框来选择进入哪一个应用  

	Intent intent = new Intent();
	intent.setAction("android.intent.action.VIEW");
	intent.setData(Uri.parse("market://details?id=" + getPackageName()));
	startActivity(intent);

我们可以通过如下代码，来查看哪些应用是支持这个意图的，也就是系统会弹出哪些供选择的应用  

	List<ResolveInfo> localList = context.getPackageManager().queryIntentActivities(paramIntent, PackageManager.GET_INTENT_FILTERS);

获取应用的信息，icon、label、packageName  

	ResolveInfo.loadIcon(pManager)	// Icon  
	ResolveInfo.loadLabel(getPackageManager()).toString()	// Label  
	ResolveInfo.activityInfo.packageName	// PackageName  

###二、代码具体实现###

	private String[] validPackageNames = new String[]{
			// "com.android.vending",	// GooglePlay
			"com.baidu.appsearch",		// 百度助手
			"com.qihoo.appstore",		// 360手机助手	
			"com.ijinshan.ShouJiKongService", 		// 金山手机助手
			"com.tencent.android.qqdownloader"};	// 应用宝 
	
	// 按钮的单击事件，点击后系统会弹出所有支持改意图的应用供选择
	public void gotoMarket(View v) {
	     Intent paramIntent = MarketUtil.getIntent(packageName);
	     List<ResolveInfo> resolveList = MarketUtil.getResolveInfos(getApplicationContext(), paramIntent, validPackageNames);
	     if (resolveList == null || resolveList.size() == 0) {
				// TODO open WebView
				Toast.makeText(this, "empty market app", Toast.LENGTH_SHORT).show();
			} else {
				startActivity(paramIntent);
			}
	}
	
	// 按钮的单击事件，点击后弹出对话框显示自定义的部分应用
	public void gotoMarketCustom(View v) {
		Intent paramIntent = MarketUtil.getIntent(packageName);
		List<ResolveInfo> resolveList = MarketUtil.getResolveInfos(getApplicationContext(), paramIntent, validPackageNames);
		if (resolveList == null || resolveList.size() == 0) {
			// TODO open WebView
			Toast.makeText(this, "empty market app", Toast.LENGTH_SHORT).show();
		} else {
			if (resolveList.size() == 1) {
				paramIntent.setPackage(resolveList.get(0).activityInfo.packageName);
				startActivity(paramIntent);
			} else {
				AlertDialog.Builder builder = new AlertDialog.Builder(this);
				builder.setTitle("请选择应用");
				ListView lvMarket = new ListView(this);
				builder.setView(lvMarket);
				List<MarketBean> marketBeans = parseMarkBean(resolveList);
				MarketAppAdapter marketAppAdapter = new MarketAppAdapter(this, marketBeans);
				lvMarket.setAdapter(marketAppAdapter);
				lvMarket.setOnItemClickListener(new OnItemClickListener() {
					
					@Override
					public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
						MarketBean bean = (MarketBean) parent.getItemAtPosition(position);
						Intent intent = new Intent();
						intent.setAction("android.intent.action.VIEW");
						intent.setData(Uri.parse("market://details?id=" + packageName));
						intent.setPackage(bean.getmPackageName());
						startActivity(intent);
					}
				});
				AlertDialog dialog = builder.create();
				dialog.setCanceledOnTouchOutside(true);	// 设置点击对话框以外部分取消对话框
				dialog.show();
			}
		}
	}

	private List<MarketBean> parseMarkBean(List<ResolveInfo> resolveList) {
		List<MarketBean> marketBeans = new ArrayList<MarketBean>();
	     PackageManager pManager = getPackageManager();
	     for (ResolveInfo item : resolveList) {
	    	 MarketBean bean = new MarketBean();
	    	 bean.setmIcon(item.loadIcon(pManager));
	    	 bean.setmLabel(item.loadLabel(pManager).toString());
	    	 bean.setmPackageName(item.activityInfo.packageName);
	    	 marketBeans.add(bean);
	     }
		return marketBeans;
	}

'MarketUtil' 工具类

	public static Intent getIntent(String packageName) {
		StringBuilder localStringBuilder = new StringBuilder().append("market://details?id=");
		localStringBuilder.append(packageName);
		Uri localUri = Uri.parse(localStringBuilder.toString());
		return new Intent("android.intent.action.VIEW", localUri);
	}

	/**
	 * queryIntentActivties by  paramIntent
	 * @param context
	 * @param paramIntent
	 * @param validPackageNames    is null return intent Activity, else return accord Activity
	 * @return
	 */
	public static List<ResolveInfo> getResolveInfos(Context context, Intent paramIntent, String[] validPackageNames) {
		List<ResolveInfo> localList = context.getPackageManager().queryIntentActivities(paramIntent, PackageManager.GET_INTENT_FILTERS);
		// when validPackageNames is null, return all intent activity.
		if (validPackageNames != null) {
			// else return accord Activity
			List<ResolveInfo> newList = new ArrayList<ResolveInfo>(validPackageNames.length);
			for(ResolveInfo item : localList){
				String itemName = item.activityInfo.packageName;
				Log.i("myTag", "packageName = " + itemName + "    Label = " + item.loadLabel(context.getPackageManager()).toString());
				if (itemName == null) continue;
				for (int i = 0; i < validPackageNames.length; i++) {
					if (itemName.equals(validPackageNames[i])) {
						newList.add(item);
						break;
					}
				}
			}
			return newList;
		}
		return localList;
	}


###三、相关介绍###
#####1，通过PackageManager获取手机端已安装的apk文件的信息#####

	PackageManager packageManager = this.getPackageManager();  
	List<PackageInfo> packageInfoList = packageManager.getInstalledPackages(0);  

	/** 
	 * 查询手机内非系统应用 
	 * @param context 
	 * @return 
	 */  
	public static List<PackageInfo> getAllApps(Context context) {  
	    List<PackageInfo> apps = new ArrayList<PackageInfo>();  
	    PackageManager pManager = context.getPackageManager();  
	    //获取手机内所有应用  
	    List<PackageInfo> paklist = pManager.getInstalledPackages(0);  
	    for (int i = 0; i < paklist.size(); i++) {  
	        PackageInfo pak = (PackageInfo) paklist.get(i);  
	        //判断是否为非系统预装的应用程序  
	        if ((pak.applicationInfo.flags & pak.applicationInfo.FLAG_SYSTEM) <= 0) {  
	            // customs applications  
	            apps.add(pak);  
	        }  
	    }  
	    return apps;  
	}  

获取应用icon、label、packageName等信息

	//set Icon  
	shareItem.setIcon(pManager.getApplicationIcon(pinfo.applicationInfo));  
    //set Application Name  
    shareItem.setLabel(pManager.getApplicationLabel(pinfo.applicationInfo).toString());  
    //set Package Name   
    shareItem.setPackageName(pinfo.applicationInfo.packageName);  


#####2，分享文本#####

	Intent sendIntent = new Intent();  
	sendIntent.setAction(Intent.ACTION_SEND);  
	sendIntent.setType("text/*");  
	sendIntent.putExtra(Intent.EXTRA_TEXT, contentEditText.getText().toString());  
	startActivity(sendIntent);  