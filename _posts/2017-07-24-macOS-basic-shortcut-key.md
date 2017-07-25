---
layout: post
title: "Mac OS 使用笔记"
modified: 2017-7-24 19:38:00
excerpt: "基础快捷键和常用操作......"
tags: [Notes]
comments: true
---

----------

### 一、窗体操作快捷键

- 关闭窗体

	`Command + W`				

- 退出程序

	`Command  +  Q`				
	
- 窗体间切换

	`Command + Tab`		
	
- 全屏模式

	`Control + Command + F`		
	
- 隐藏当前窗体（只有当前一个窗体时不可用）

	`Command  +  H`				
	
- 隐藏其它窗体（只有当前一个窗体时不可用）

	`Command  + Option + H`		
	
- 最小化当前应用窗体   

	`Command +  M`		
			
- 最小化当前应用所有窗体

	`Command + Option + M`		


### 二、搜索工具 Spotlight

在 **“系统设置” -> "Spotlight" -> "快捷键"** 中可以设置打开搜索的快捷键。我设置的快捷键如下：

1. 应用程序搜索

	`Option + Space`
	
	输入应用名回车即可打开，如打开 `Terminal`。和在 Windows 下用 `Win + R` 输入 `cmd` 打开 `DOS` 的感觉差不多。 
	
2. 在 Finder 中搜索

	`Option + Command  +  Space`
	
	全局搜索文件，速度相比 Windows 要快得多。


### 三、文件管理器 Finder

1. 显示或隐藏文件的快捷键

	`Command  +  Shift  +  .`		
	
2. 查看系统根目录

	菜单栏 **“GO” -> "Go to Folder"** 中输入 `/` 符号，回车即可查看整个文件目录。
	
3. 打开多个窗体（通用）

	`Command + N`
	
4. 设置文件默认打开方式

	**选择文件 -> 右键 -> 显示简介 -> 选择“打开方式” -> 全部更改**
	
	
### 四、其它

1. 按键符号对应关系

	- ⌘（command）

	- ⌥（option）

	- ⇧（shift）

	- ⌃（control）


2. 在未配置环境变量的情况下，访问 Gradle 命令

	-  进入 bin 目录，如：

		`cd /Users/iOnesmile/Library/gradle-4.0/bin`
		
	-  输入命令（注意前面的 `./`）

		`./gradle -version`
