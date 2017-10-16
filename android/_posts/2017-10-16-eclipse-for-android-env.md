---
layout: post
title: "Eclipse for Android 开发环境搭建及各种坑"
modified: 2017-10-16 10:12:00
excerpt: "Eclipse 下载、配置和其中遇到的异常..."
tags: [Eclipse]
comments: true
---


由于公司还有一些老项目，不得不再重装一下 Eclipse，中间遇到了不少坑，把流程记录一下。

### 一、下载安装

1. **下载 Eclipse**    

	路径： <https://www.eclipse.org/downloads/packages/eclipse-android-developers/neonm6>
	
	
2. **下载 SDK**

	最开始我下载的是最新的 SDK 版本，但是发现 [Android SDK Manager 无法打开](http://www.cnblogs.com/yunxiblog/p/5820436.html)。最后尝试使用一个老一点的版本，如：[android-sdk_r24.4.1-macosx](http://tools.android-studio.org/index.php/sdk/)
	
	
3. **配置 SDK**

	在 Eclipse -> Preference -> Android -> `SDK Location` 中选择下载的版本，并点击 `Apply` 按钮。
	
	打开 Android SDK Manager，主要需要下载的有如下几个：
		
	- Tools
		- Android SDK tools
		- Android SDK Build-tools
	- Android X.X.X (API XX)
		- SDK Platform

		
### 二、遇到的异常

1. **运行时 Unable to build**，错误信息：
	
	```
	Failed to load C:\Program Files (x86)\Android\android-sdk\build-tools\26.0.0-preview\lib\dx.jar   
	Unable to build: the file dx.jar was not loaded from the SDK folder
	```
		
	**原因**： 
		build-tools 工具的版本太高出现的问题    
		  
	**解决**：
		换一个低版本的 build-tools 工具，如 24.0.0。最简单的方式是在目录下只留下一个，删除其它。
		

2. **打签名包时**，弹出对话框
	
	```
	Conversion to Dalvik format failed with error 1
	```
	
	**解决**：
	
	1. 关闭自动 BuildProject -> Build Automatically 
	2. clean 项目 Project -> Clean 
	3. 手动 Build，右键点击项目，然后 Build Project 
	4. 然后再 Export 项目，报错消失了！
	
	参考：<http://blog.csdn.net/myzlhh/article/details/52279443>
	

3. **打签名包时**，弹出对话框

	```
	No DEX file found.
	```
	
	**原因**：
		eclipse打包的时候，bin路径下的dex文件没有生成，导致无法打包。
		
	**解决**：
	
	- 方案一：在打包之前 Clean 并 run 一下项目
	- 方案二：eclipse -> preferences -> android -> build 下，Skip packaging and dexing until export to launch 选项去掉勾选。

	参考：<http://blog.csdn.net/henrychow_2015/article/details/60954473>


