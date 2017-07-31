---
layout: post
title: "Android 应用保活笔记"
modified: 2017-7-31 17:10:00
excerpt: "提高进程优先级降低被杀死，通过系统机制拉活应用..."
tags: [Android]
comments: true
---

#### 一、进程拉活包括两个层面

- **提高进程优先级，降低进程被杀死的概率**   
	1. 服务绑定通知栏
	2. 一个像素点在桌面
	
- **在进程被杀死后，进行拉活**   
	1. 利用第三方应用广播拉活
	2. 利用系统Service机制拉活（START_STICKY）
	3. 利用Native进程拉活（Linux 中的 fork 机制创建 Native 进程，Android5.0以下可行）
	4. 利用 JobScheduler 机制拉活
	5. 利用账号同步机制拉活
	6. 其它唤醒应用方式（各种推送：小米、极光、阿里、华为、腾讯信鸽）


#### 二、补充其它方式

1. 双进程Service：让2个进程互相保护，其中一个Service被清理后，另外没被清理的进程可以立即重启进程 
  
2. 在已经root的设备下，修改相应的权限文件，将App伪装成系统级的应用（Android4.0系列的一个漏洞，已经确认可行）


#### 三、系统进程等级

- Foreground Process(前台进程)

- Visible Process(可见进程)

- Service Process(服务进程)

- Background Process(后台进程)

- Empty Process(空进程)

Service 的进程处于第三个位置. 系统的回收会 **从低到高** 依次回收, 所以我们必须提高 Service 的等级。


#### 四、参考

Android 进程保活招式大全： <https://segmentfault.com/a/1190000006251859>     

知乎问答：<https://www.zhihu.com/question/29826231/answer/71207109>   

一个像素点保活：<http://www.jianshu.com/p/a61ecc016aa5>