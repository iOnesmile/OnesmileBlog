---
layout: post
title: "Kotlin 实践项目（密码本）"
modified: 2017-9-03 20:16:00
excerpt: "开源的项目，专治记不住密码..."
tags: [Kotlin, 项目]
comments: true
---

Google 推出 `Kotlin` 作为 `Android` 的官方语言已经有一段时间，最近用工作上一些闲暇时间做了个项目，切身体验下。


#### 一、需求描述

一直以来对各个网站的密码管理都比较头疼，因为担心“撞库”，所有网站密码都不相同。注册网站时都会随便写一个密码，却没有一个好的密码管理工具，下次登录时基本都需要找回密码，结果又忘记如此反复。对于普通的网站重新找回一次并不算复杂，但是对于像 QQ、微信、支付宝 这样有比较高安全验证的网站找回起来并不容易，处理起来很繁琐特别头疼（承认我记忆力不好，突然想到自己好几张银行卡密码也忘记了 (￣▽￣)~~~ ，不过也没存款。。。。。。）

之前找过管理密码的软件，但不是太放心。软件又没开源，也不确定有没有后门或漏洞，自己动手要踏实得多。

我的《密码本》正是基于这一需求产生的，不但让自己的密码相对有一个保障，同时练练手学习新的技术。最后该项目作为开源项目，希望也能帮助你解决同样的烦恼。


#### 二、项目截图

GitHub： <https://github.com/iOnesmile/PasswordNotebook>

安装包：  [百度云下载](http://pan.baidu.com/s/1jIDYHds)

<img src="http://www.ionesmile.com/images/android/cipher_box_v1_00_preview.jpg"/> 


#### 三、待完善

- 提升加密文件安全度，研究其它算法并检验安全性
- 应用内安全验证，如数据存储、锁屏、页面超时、导出权限等
- 优化交互体验，简化操作流程，和指纹解锁等验证机制
- 其它平台开发（iOS、Windows、MacOS），信息同步
- 语言国际化
- 其它......

> 如果有什么好的想法和建议，或在使用中遇到什么问题，欢迎反馈，我们一起完善吧！！！

#### 四、使用 Kotlin 的坑或技术总结

1. 在设置监听时，提示错误 `Expected a value of type Boolean`   
	**原因**：该监听有一个返回值，类型是 `Boolean`   
	例如：   
	
	```kotlin
	textView.onLongClick {
	    // TODO
	    return@onLongClick true
	}
	```
	
2. EditText 设置值时提示 `Type mismatch.  Required: Editable!  Found: String`    
	原因：要给 `EditText` 设置 `String` 类型的值时，需要使用 `setText()` 方法   
	例如：
	
	```kotlin
	editText.setText("XXX")
	```	

3. [函数式编程](http://ohmerhe.com/2016/07/05/kotlin_function_three_common_methods/)   
	- `map`   
		映射函数也是一个高阶函数，将一个集合经过一个传入的变换函数映射成另外一种集合
	
	- `filter`      
		筛选函数将用户给定的布尔逻辑作用于集合，返回由原集合中符合条件的元素组合的一个子集
		
	- `reduce`     
		归纳函数将一个数据集合的所有元素通过传入的操作函数实现数据集合的积累叠加效果


#### 五、使用技术/库

- [Kotlin](http://kotlinlang.org/) 语言（目前小部分是 Java）   
- [Realm](https://blog.realm.io/realm-for-android/) 数据库
- DES 加密，BASE64 编码
- Toolbar
- Preference	设置
- [recyclerview-swipe](https://github.com/yanzhenjie/SwipeRecyclerView)	侧滑
- [exfilepicker](https://github.com/bartwell/ExFilePicker)   文件选择
- [lockpattern](https://github.com/sym900728/LockPattern) 九宫格锁屏
- [simpleRatingbar](https://github.com/FlyingPumba/SimpleRatingBar)   星级评分条
- Gson		解析库
- 应用内图标来源		<https://github.com/google/material-design-icons>
- 应用图标来源			<http://www.iconfont.cn>
