---
layout: post
title: "Google 支付笔记"
modified: 2017-10-30 10:18:00
excerpt: "接入流程、测试环境、常见错误..."
tags: [Pay]
comments: true
---


### 一、接入流程

其实没啥好说的，直接看 [快速上手文档](https://developer.android.com/training/in-app-billing/preparing-iab-app.html#AddToDevConsole) 和 [SDK API 的使用说明](https://developer.android.com/google/play/billing/billing_overview.html#samples)。 

...     
...    
...   

接入中一些小小的总结：

1. 下载[官网支付 Demo](https://github.com/googlesamples/android-play-billing)，熟悉调用流程

2. 按照上面 Training 文档，将 Demo 中的支付相关代码移植到自己的项目中
	- 添加 `IInAppBillingService.aidl` 文件
	- 设置 `com.android.vending.BILLING` 权限
	- 调用 IabHelper 对象，完成初始化操作（其中 base64EncodedPublicKey 需要到 [控制台](http://play.google.com/apps/publish) 拿取）

3. 打包当前版本，并上传到控制台
	- 进入控制台： <http://play.google.com/apps/publish>
	- 在当前应用下【版本管理】 -> 【应用版本】，在 【Aplha 版】 -> 【管理 Alpha 版】 -> 【修改版本】 中上传当前项目打包的安装包
	- 注意接下来测试要和已上传安装包的签名文件、版本号保持一致
	- 在 【设置】 -> 【开发者账号】 -> 【账号详情】 -> 【许可测试】 下添加测试人员账号

4. 在当前应用下 【商品发布】 -> 【应用内商品】 中添加商品

5. 查询商品详情 [mHelper.queryInventoryAsync](https://developer.android.com/training/in-app-billing/list-iab-products.html#QueryDetails)

6. 购买商品 [mHelper.launchPurchaseFlow](https://developer.android.com/training/in-app-billing/purchase-iab-products.html)

7. 消耗商品 [mHelper.consumeAsync](https://developer.android.com/training/in-app-billing/purchase-iab-products.html)
	
	
### 二、测试环境

1. 支持 Google Play 的手机（比如 Nexus，国内很多手机不支持需要 root）
2. Google 开发者账号和测试账号
3. 支持双币的信用卡（测试账号付款的时候要用，虽然实际不扣费）
4. 梯子

### 三、常见错误

1. 无法购买您要买的商品

	- 当前Google Play帐号不是测试帐号
	- 当前商品未在后台配置

2. 此版本的应用为配置为通过Google Play结算。有关详情，请访问帮助中心。

	- 检查下打包所用的签名与上传Google Play后台的签名是否一致
	- 检查版本号与上传的版本号是否一致
