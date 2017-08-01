---
layout: post
title: "打包和依赖 AAR 文件"
modified: 2017-8-01 12:30:00
excerpt: "AAR 文件的单模块打包和多模块打包，以及引入到项目中的两种方式，以及常见遇到的问题..."
tags: [AAR]
comments: true
---

### 一、打包 `aar`

**1、单个模块打包**

1. 打开 `Gradle` 工具窗口,找到 `Android Library` 模块. 在 `build` 任务中双击 `assemble`.

2. 执行成功后，在 `mylibrary/build/outputs/aar` 目录下找到 `aar` 包.

默认 `Debug` 和 `Release` 的 `AAR` 包都会打出来,当然你也可以选择只打 `Debug` 的包,双击 `assembleDebug` 任务就可以了. 只打 `Release` 的包同理.

**2、多个模块打包**

当要打包的模块又依赖了其它几个模块时，常常需要把它们打包成一个 `aar`。多模块打包使用 [**fat-aar**](https://github.com/adwiv/android-fat-aar)，打包关键步骤如下：

1. 将下载好的 `fat-aar.gradle` 文件添加到对应的模块目录中，并在 `build.gradle` 中引入 `apply from: 'fat-aar.gradle'`。或直接引用 `apply from: 'https://raw.githubusercontent.com/adwiv/android-fat-aar/master/fat-aar.gradle'`

2. 添加要打包的工程，使用 `embedded` 关键字。示例代码如下：

	```gradle
	apply from: 'fat-aar.gradle'
	dependencies {
		...
	   embedded project(':DynamicPageLibrary')
	   embedded project(':VideoPlayerLib')
	   embedded project(':AudioPlayLibrary')
	   embedded project(':BaseCloudMusicResource')
	}
	```

3. 步骤同上《单个模块打包》一致。

### 二、引入 `aar`


**方法一、通过 libs 引入到 app 中**

1. 把 `aar` 文件放在 `libs` 目录下

2. 在 `app` 的 `build.gradle` 中添加如下内容
	
	```gradle
	repositories {
	    flatDir {
	        dirs 'libs' 
	    }
	}
	```

3. 之后通过如下方式引入
	
	```gradle
	dependencies {
	    compile(name:'test', ext:'aar')
	}
	```

4. `Rebuild project`

5. 如果发现引入后无法使用，重启 Android studio


**方法二、把 `aar` 作为一个库工程的方式引入**

当项目中库工程较多且依赖关系比较复杂时，最好采用这一种方式。如：某一个库工程也要引入这个 `aar` 时。

`菜单栏` -> `File` -> `New` -> `New Module`
	
 -> `Import .Jar/.AAR Package` 
 
 -> **`Next`** 
 
 -> 选择 `File name` 的文件 -> `Subproject name` 命名工程 
 
 -> **`Finish`**
	
创建完成后是一个工程，工程中包括 `aar` 文件和 `build.gradle` 文件。`build.gradle` 文件内容如下：
	
```gradle
configurations.create("default")
artifacts.add("default", file('musiclibrary_20170622.aar'))
```

<img src="http://www.ionesmile.com/images/android/android_studio_import_aar.png"/>

### 三，遇到的问题

1. `Non-constant Fields in Case Labels`

	**原因**：在 Android Library 中不能使用 `switch case`
	
	**解决**：改成用 `else if`，如下图：

	<img src="http://www.ionesmile.com/images/android/android_library_switch_to_elseif.png"/>

2. `java.lang.IllegalArgumentException: No view found for id 0x7f0d013d () for fragment TestFragment`

	**描述**：在项目中引用了库里的 `fragment`，在运行后抛出了找不到 `view` 的异常。但是在 Demo 项目中运行是没有问题的。

	**原因**：库里 `fragment` 的 `layoutID` 与项目中另外一个 `Fragment` 的 `layoutID` **名字相同，导致项目中的布局会覆盖库中的布局。**。
	
	**解决**：修改为不同的名称。在库中要注意资源名称可能与项目同名的问题，比如在库中的资源文件都添加前缀或后缀，或较长不容易重复的名字。**同名的资源文件只会存在一个，根据库的嵌套关系，外层会覆盖内层的资源文件。**




