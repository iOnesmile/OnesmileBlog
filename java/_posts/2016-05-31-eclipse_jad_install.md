---
layout: post
title: "Eclipse 反编译插件安装jad"
modified: 2016-5-31 17:58:00
excerpt: "Eclipse 反编译插件安装jad"
tags: [Java]
comments: true
---


**1，下载jadClipse**

 [http://sourceforge.net/projects/jadclipse/](http://sourceforge.net/projects/jadclipse/)

   

**2，将net.sf.jadclipse_3.3.0.jar 拷贝到eclipse的plugins目录下**


**3，设置jad的可执行文件路径和生成的临时文件路径**    

<img src="http://www.ionesmile.com/images/java/java_jad_plus_setting1.jpg" width="600"/>   


**4，如果你发现进入class并没有被反编译，那么修改文件关联**    

<img src="http://www.ionesmile.com/images/java/java_jad_plus_setting2.jpg" width="600"/>     

注意，这里有两个.class的关联，可以直接修改第二个就是没有源代码的情况，Associated editors下添加一个编辑器,并且设置为默认的，如下图:  
<img src="http://www.ionesmile.com/images/java/java_jad_plus_setting3.jpg" width="600"/>   



**5，我之前下的一些eclipse并没有.class without source项，这时候就在.class 下添加jad的编辑器并且设置为默认**    

　  

文件下载地址：[http://www.ionesmile.com/file/Plugins/jad.zip](http://www.ionesmile.com/file/Plugins/jad.zip)  

原文地址: [http://tangmingjie2009.iteye.com/blog/1916992](http://tangmingjie2009.iteye.com/blog/1916992)

  




