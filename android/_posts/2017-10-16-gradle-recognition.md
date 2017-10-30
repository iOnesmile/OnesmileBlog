---
layout: post
title: "重新认识 Gradle 打包"
modified: 2017-10-16 10:18:00
excerpt: "Gradle 是什么，Android 打包插件，AAR 文件，依赖冲突解决办法，AS 如何快速打包..."
tags: [Gradle]
comments: true
---


前一段时间因为打 AAR 包折腾了一整天，不得不怀疑我对 Gradle 的认识。虽然在此之前确实能解决一些 Gradle 打包依赖的冲突或错误，但并没有系统的去学习。


### 一、Gradle 是什么

[Gradle](http://blog.csdn.net/qinxiandiqi/article/details/37757065) 是 **依赖管理** + **构建工具**。它继承了 [Ant](http://www.cnblogs.com/super-d2/archive/2012/12/31/2840989.html) 的灵活和 [Maven](http://maven.apache.org/guides/getting-started/index.html) 的生命周期管理，它最后被 google 作为了 Android 御用管理工具。它最大的区别是不用 XML 作为配置文件格式，采用了DSL格式，使得脚本更加简洁。

- **Ant** 是最早的构建工具，基于 idea，好象是2000年有的，当时是最流行 java 构建工具，不过它的 XML 脚本编写格式让 XML 文件特别大。对工程构建过程中的过程控制特别好。

- **Maven** 它是用来给 Ant 补坑的，Maven 第一次支持了从网络上下载的功能，仍然采用 xml 作为配置文件格式，它的问题是不能很好的相同库文件的版本冲突。Maven 专注的是依赖管理，构建神马的并不擅长。

- **构建工具** 是什么

	单个源码文件，你可以很轻松地 javac、gcc。然而项目结构复杂的时候，从源代码到实际产出的生成物之间需要经过一些列的转换操作，比如说编译、打包。而这一整个完整的过剩叫做“构建”。

- [Maven 的主要功能主要分为5点](http://www.cnblogs.com/renhui/p/6855934.html)，分别是依赖管理系统、多模块构建、一致的项目结构、一致的构建模型和插件机制。


### 二、Android 是如何打包的

将一堆源码生成一个 APK 的过程就是打包，Gradle 作为一个构建平台已经有了很好的基础，到具体的打包应用步骤就由 [Android Gradle Plugin](https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle) 完成。即我们在项目下 build.gradle 的配置：

```gradle
buildscript {
  ...
  dependencies {
    classpath 'com.android.tools.build:gradle:2.3.3'
  }
}
```

另外在项目子工程中，app、XXXlibrary 内的 build.gradle 文件使用 `apply plugin` 来指定具体使用的插件，如：

```gradle
apply plugin: 'com.android.application'
apply plugin: 'com.android.library'
```

关于该插件打包时更多的配置，请参见（如：buildTypes、productFlavors、signingConfigs、ProGuard）：
<https://developer.android.com/studio/build/index.html>



### 三、什么是 AAR 文件

[AAR](https://developer.android.com/studio/projects/android-library.html#aar-contents) 文件本身是一个 zip 文件，在您构建相关应用模块时，库模块将先编译到 AAR 文件中，然后再添加到应用模块中。为了避免常用资源 ID 的资源冲突，请使用在模块（或在所有项目模块）中具有唯一性的前缀或其他一致的命名方案。解压后可以看到如下目录：

- aapt
- aidl
- AndroidManifest.xml
- assets
- **classes.jar**
- jni
- libs
- **R.txt**
- res



### 四、依赖冲突的解决办法

在集成多个库工程时，出现了如下异常：

```
java.lang.NoSuchMethodError: android.support.v4.app.ActivityCompat.startActivity
```

很明显是 support 包冲突的问题，查看 `gradle.build` 文件，在 `dependencies` 中有如下警告信息：

```
All com.android.support libraries must use the exact same version specification (mixing versions can lead to runtime crashes).
Found versions 25.2.0, 24.0.0. Examples include com.android.support:animated-vector-drawable:25.2.0 and com.android.support:mediarouter-v7:24.0.0
```

提示存在多个版本，可能会导致运行时错误，需要使用同一个版本。解决办法：

- 通过 `./gradlew dependencies` 或 `gradle dependencies` 查看依赖树

- 可以使用 `exclude` 关键字来排除单个的库工程，如：

	```
	compile('com.yanzhenjie:recyclerview-swipe:1.0.3') {
  		exclude group: 'com.android.support', module: 'recyclerview-v7'
	}
	```
	
- 可以使用全局的方式替换，在根目录的 build.gradle 文件中，代码如下：

	```
	subprojects {
	    project.configurations.all {
	        resolutionStrategy.eachDependency { details ->
	            if (details.requested.group == 'com.android.support'
	                    && details.requested.name.contains('appcompat-v7') ) {
	                details.useVersion "24.2.0"
	            }
	        }
	    }
	}
	```
	并且在项目中可见的地方，修改为相同的版本号，并 `sync project gradle`
	
参考：   
[the-exact-same-version-specification](https://stackoverflow.com/questions/42374151/all-com-android-support-libraries-must-use-the-exact-same-version-specification)    
[the-exact-same-version/42582204#42582204](https://stackoverflow.com/questions/42581963/all-com-android-support-libraries-must-use-the-exact-same-version/42582204#42582204)


### 五、Android Studio 如何快速运行程序

运行速度严重影响了开发效率，虽然换 MBP 后比以前的运行速度提高了三倍，但还是不够满意。提高速度主要有如下几个方法：

1. 提高编译内存，在工程目录下的 gradle.properties 文件中新增如下代码：

	```
	org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
	org.gradle.parallel=true
	org.gradle.daemon=true
	```
	
2. 在 app 子工程目录下的 build.gradle 中配置下改成增量编译和调整 minSdkVersion

	```
	dexOptions {
        incremental true
    }

    productFlavors {
        dev {
            // dev utilizes minSDKVersion = 21
            // to allow the Android gradle plugin
            // to pre-dex each module and produce an APK that can be tested on
            // Android Lollipop without time consuming dex merging processes.
            minSdkVersion 21
        }
        prod {
            // The actual minSdkVersion for the application.
            minSdkVersion 15
        }
    }
    ```
	
3. 使用插件 [Freeline](https://github.com/alibaba/freeline)


### 参考文档

上面是我再查找资料中做的一些总结，内容不全面和一些不连贯地方。如要更详细的了解请见如下链接：

- Android Gradle Plugin 指南： <https://developer.android.com/studio/build/index.html>

- Android Gradle Plugin 指南（1-6）： <http://blog.csdn.net/qinxiandiqi/article/category/2394347>

- 项目自动构建工具对比(Maven、Gradle、Ant)： <http://www.cnblogs.com/renhui/p/6855934.html>

- Android Gradle Plugin 版本： <https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle>

- AAR 详解： <https://developer.android.com/studio/projects/android-library.html#aar-contents>

- Maven 官网： <http://maven.apache.org/guides/getting-started/index.html>
