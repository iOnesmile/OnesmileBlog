---
layout: post
title: "CentOS 上搭建 Jenkins"
modified: 2018-6-19 12:54:00
excerpt: "踩坑笔记，打包 Android 项目......"
tags: [Jenkins]
comments: true
---  

#### 开发环境：  

> 服务器系统：CentOS release 6.5 (Final)   
> 服务器内核版本：2.6.32-431.el6.x86_64   
> 我的电脑：macOS    


- 登录远程服务器

	```shell
	# ssh -p 端口 用户名@IP
	ssh -p 22 root@192.168.11.129
	```
	
- 安装 JDK，默认环境有问题
	
	```
	/usr/lib/jvm/jdk1.8.0_161
	```
	
	- 查看服务器 JDK	`rpm -qa | grep java`
	- 卸载已有 JDK		`yum remove *openjdk*`
	- 官网下载 JDK 包 [jdk-8u161-linux-x64.tar.gz](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
	- 将下载的安装包上传到服务器
		
		```shell
		scp /Users/ionesmile/Downloads/jdk-8u161-linux-x64.tar.gz root@192.168.11.129:/usr/lib/jenkins-env
		```
		
		- 使用这条命令时需要处于登出状态 `logout`
	
	- 解压安装包到 `/usr/lib/jvm` 目录
		
		```
		tar -zxvf jdk-8u161-linux-x64.tar.gz -C /usr/lib/jvm
		```
	- 添加环境变量

		- 打开配置 `vim /etc/profile`
		- 在最前面添加如下代码：

			```
			export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_131  
			export JRE_HOME=${JAVA_HOME}/jre  
			export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
			export  PATH=${JAVA_HOME}/bin:$PATH
			```
		- 执行更新配置文件 `source /etc/profile`

- 下载 Jenkins 的 AAR 文件，并启动

	```
	/usr/lib/jenkins
	// 不挂断的执行程序
	nohup java -jar jenkins.war --httpPort=8085 &
	```

- 开启防火墙端口号（以 8085 端口为例）

	```
    /sbin/iptables -I INPUT -p tcp --dport 8085 -j ACCEPT   写入修改
 
    /etc/init.d/iptables save   保存修改
 
    service iptables restart    重启防火墙，修改生效
	```

- Git 证书配置
	
	- 在服务器上生成 SSH key，包括公钥和私钥

		```
		// 查看是否有公钥
		cat ~/.ssh/id_rsa.pub
		// 如果未查询到公钥，通过此命令生成
		ssh-keygen -t rsa -C "wangrui@snaillove.com"
		```
		
	- 在 GitLab 中设置 SSH 公钥（http://xxx/gitlab/profile/keys）
	- 在 Jenkins 的证书管理中添加 SSH 私钥

- Android SDK 下载，并安装 SDK Platforms 和 Tools

	- 在官网下载 Linux SDK 命令行工具（页面最底部 [sdk-tools-linux-3859397.zip](https://developer.android.com/studio/index.html)）
	- 将下载的包上传到服务器

		```
		scp /Users/ionesmile/Downloads/sdk-tools-linux-3859397.zip root@192.168.11.129:/usr/lib/jenkins-env/android-sdk
		```
	- 解压 zip 压缩包到当前目录

		```
		unzip sdk-tools-linux-3859397.zip 
		```
		
	- 进入到 `/tools/bin/` 目录，使用 [`sdkmanager`](https://developer.android.com/studio/command-line/sdkmanager.html#options)

		- 查看已安装和有效的 SDK 包

			```
			sdkmanager --list
			```
		- 安装包（命令后面可以跟一个或多个包的路径 Path）

			```
			sdkmanager "build-tools;26.0.0" "platforms;android-26"
			```

		- 我的已安装包如下
		
			```
			Installed packages:
			  Path                        | Version | Description                    | Location                    
			  -------                     | ------- | -------                        | -------                     
			  build-tools;24.0.2          | 24.0.2  | Android SDK Build-Tools 24.0.2 | build-tools/24.0.2/         
			  build-tools;25.0.3          | 25.0.3  | Android SDK Build-Tools 25.0.3 | build-tools/25.0.3/         
			  build-tools;26.0.0          | 26.0.0  | Android SDK Build-Tools 26     | build-tools/26.0.0/         
			  extras;android;m2repository | 47.0.0  | Android Support Repository     | extras/android/m2repository/
			  extras;google;m2repository  | 58      | Google Repository              | extras/google/m2repository/ 
			  patcher;v4                  | 1       | SDK Patch Applier v4           | patcher/v4/                 
			  platforms;android-24        | 2       | Android SDK Platform 24        | platforms/android-24/       
			  platforms;android-25        | 3       | Android SDK Platform 25        | platforms/android-25/       
			  platforms;android-26        | 2       | Android SDK Platform 26        | platforms/android-26/       
			  tools                       | 26.0.1  | Android SDK Tools 26.0.1       | tools/                      
			```

- 项目基础配置（构建化参数，构建命令等）

	随便找一个文档吧，比如：<https://www.jianshu.com/p/38b2e17ced73>

- 下载打包好的 Apk
	- 开启 Tomcat 服务器

		- 下载 Tomcat，不过发现目录已存在 `/usr/local/apache-tomcat-7.0.85`
		- 开启 Tomcat `catalina.sh start`
		- 访问 Tomcat，将文件放到 `webapps` 目录下访问
	
	- Gradle 配置，设置安装包输出路径

		```gradle
		android.applicationVariants.all { variant ->
	        variant.outputs.each { output ->
	            if ("true" == IS_JENKINS){
	                String fileName = "android_test_v${defaultConfig.versionName}_${new Date().format("yyyyMMdd")}_${variant.buildType.name}.apk"
	                output.outputFile = new File(JENKINS_BUILD_PATH, fileName)
	            }
	        }
	    }
		```
	
	- 生成输出路径二维码，实现扫码下载
		- 通过 Python 生成二维码
		- 在 “构建后操作” -> “Set build description” 设置，如果不存在检查是否有安装 “description setter” 插件
	
	
- 在提交代码时实现自动 Build



- 遇到的问题

	- 缺少 SDK 包导致

		```
		* What went wrong:
		A problem occurred configuring project ':BaseCloudMusicResource'.
		> Could not resolve all dependencies for configuration ':BaseCloudMusicResource:_debugCompile'.
		   > Cannot resolve external dependency com.android.support:appcompat-v7:24.2.0 because no repositories are defined.
		     Required by:
		         iLight:BaseCloudMusicResource:unspecified
		```	
		
		或如下错误：
		
		```
		* What went wrong:
		A problem occurred configuring project ':DeviceMusicLibrary'.
		> A problem occurred configuring project ':cloudmusiclibrary:cmreally'.
		   > failed to find target with hash string 'android-25' in: /usr/lib/jenkins-env/android-sdk
		```

		解决：使用前面提到的 `sdkmanager` 工具安装提示所缺少的包
		
	
	- 库 `glibc` 版本过低

		```
		AAPT err(Facade for 1129807373): xxx/aapt: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by xxx/lib64/libc++.so)

		Exception in thread "png-cruncher_20" java.lang.RuntimeException: Timed out while waiting for slave aapt process, make sure the aapt execute at /usr/lib/jenkins-env/android-sdk/build-tools/24.0.2/aapt can run successfully (some anti-virus may block it) or try setting environment variable SLAVE_AAPT_TIMEOUT to a value bigger than 5 seconds
		```	
		
		原因：CentOS 6.5 最高的 `glibc` 版本为 2.12，而 aapt 提示需要 2.14 的包
		
		解决：安装 2.14 的包（存在风险）<https://www.cnblogs.com/gavin56/p/6655025.html>
		
		- 下载安装包 <http://pan.baidu.com/s/1o83vPxS> 或 <http://ftp.redsleeve.org/pub/steam/>
		- 安装

			```
			rpm -Uvh glibc-2.15-60.el6.x86_64.rpm glibc-common-2.15-60.el6.x86_64.rpm glibc-devel-2.15-60.el6.x86_64.rpm glibc-headers-2.15-60.el6.x86_64.rpm --nodeps --force
			```